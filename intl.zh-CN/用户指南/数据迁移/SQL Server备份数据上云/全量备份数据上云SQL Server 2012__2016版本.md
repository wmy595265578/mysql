# 全量备份数据上云SQL Server 2012/2016版本 {#concept_msg_hpw_ydb .concept}

本文档介绍如何把用户OSS上的全量备份文件迁移到阿里云RDS for SQL Server。

本文档适用于如下以下版本的实例：

-   RDS for SQL Server 2012/2016 Web版、企业版的基础系列（即单机版）
-   RDS for SQL Server 2012/2016标准版、企业版的高可用系列（即双机版）

关于RDS for SQL Server 2008 R2企业版的高可用系列的数据上云方法，请参考[全量备份数据上云SQL Server 2008 R2版](intl.zh-CN/用户指南/数据迁移/SQL Server备份数据上云/全量备份数据上云SQL Server 2008 R2版.md#)。

## 限制条件 {#section_pzt_zsj_zdb .section}

**备份文件版本**

不支持由高版本的备份文件往低版本做迁移，比如：从SQL Server 2016迁移到SQL Server 2012等。

**备份文件类型限制**

不支持差异备份文件或日志备份文件。

**备份文件后缀名限制**

备份文件名仅支持bak、diff、trn或者log为后缀名。如果没有使用本文中的脚本生成备份文件，请使用如下后缀名：

-   bak：表示全量备份文件
-   diff：表示差异备份文件
-   trn或者log：表示事务日志备份

**备份文件命名限制**

全量备份文件名不能包含@或者|等特殊字符，否则会导致数据库上云失败。

## 注意事项 {#section_ybw_1tj_zdb .section}

**AliyunRDSImportRole的角色**

授予RDS服务账号访问OSS的权限以后，系统会在访问控制RAM的角色管理中创建名为AliyunRDSImportRole的角色，请勿修改或删除这个角色，否则会导致上云任务无法下载备份文件而失败。如果修改或删除了这个角色，您需要通过数据上云向导重新授权。

**备份文件命名**

全量备份文件名，不能包含中划线\(|\)、@等特殊字符。

**删除OSS上备份文件**

在OSS备份数据恢复上云任务没有完成之前，请不要删除OSS上的备份文件，否则会导致上云任务失败。

## 前提条件 {#section_ibz_btj_zdb .section}

**实例空间要求**

请确保阿里云RDS for SQL Server实例拥有足够的存储空间，如果空间不足，请提前升级实例空间，以免因为空间不足而导致迁入失败。

**目标实例中不能存在同名的目标数据库**

您无需先创建目标数据库。这一点和[全量备份数据上云SQL Server 2008 R2版](intl.zh-CN/用户指南/数据迁移/SQL Server备份数据上云/全量备份数据上云SQL Server 2008 R2版.md#)的要求相反。

如果同名的数据库已经存在，请先备份该数据库，删除该数据库，再创建迁移任务。

**在目标实例上创建高权限账号**

建议先通过 RDS 控制台创建目标实例的高权限账号，如果已经存在高权限账号，请跳过本步骤。如果目标实例中不存在高权限账号，OSS备份数据上云任务也会成功，但是您无法访问该数据库，需要参照本文最后章节“常见的错误信息”才能解决。

高权限账号的创建方法，请参考[创建数据库和账号SQL Server 2012/2016版](../../../../intl.zh-CN/快速入门SQL Server版/初始化配置/创建数据库和账号/创建数据库和账号SQL Server 2012__2016版.md#) 中的第1步至第7步。

**准备OSS Bucket**

您需要创建与目标实例同地域的OSS Bucket。如果Bucket已经存在，请跳过本步骤。创建方法如下：

1.  登录阿里云[OSS控制台](https://oss.console.aliyun.com/)。
2.  单击存储空间后面的加号**+**。
3.  设置Bucket名称、地域、存储类型和读写权限，单击**确定**。（请确保与RDS for SQL Server实例位于相同地域，否则会导致后面的步骤中无法选中备份文件。）如下图所示。

**运行DBCC CHECKDB**

请在本地环境对需要上云的数据库做DBCC CHECKDB\(‘xxx’\)检查，执行完毕后，确保没有任何的allocation errors和consistency errors。正常的结果如下：

```
...
CHECKDB found 0 allocation errors and 0 consistency errors in database 'xxx'.
DBCC execution completed. If DBCC printed error messages, contact your system administrator.
```

如果发现DBCC CHECKDB有任何错误，请先在本地环境修复数据库，否则会导致上云失败。

## 文本介绍 {#section_f25_nfk_zdb .section}

只需下面简单三步就可以轻松将本地数据迁移到云数据库RDS for SQL Server 2012/2016：

1.  备份本地数据库
2.  上传备份文件到OSS
3.  创建数据上云任务

## 备份本地数据库 {#section_idn_mfk_zdb .section}

在对本地数据库做全量备份之前，请确保已停止写入数据。备份过程中新写入的数据将不会被备份。

您可以按已知的方式执行全量备份，或者使用如下方法进行全量备份：

1.  下载[备份脚本](https://rdshelpattachments.oss-cn-beijing.aliyuncs.com/Migration/RDSBackupSpecifiedDatabasesToLocal.sql)，用SSMS打开备份脚本。
2.  根据实际情况，修改如下4个参数：

    |配置项|说明|
    |---|--|
    |@backup\_databases\_list|需要备份的数据库，多个数据库以分号或者逗号分隔。|
    |@backup\_type|备份类型。参数值如下：    -   FULL：全量备份；
    -   DIFF：差异备份；
    -   LOG：日志备份。
|
    |@backup\_folder|备份文件所在的本地目录。如不存在，会自动创建。|
    |@is\_run|是否执行备份。参数值如下：    -   1：执行备份；
    -   0：只做检查，不执行备份。
|

3.  执行备份脚本。

## 上传备份文件到OSS {#section_g33_vfk_zdb .section}

本地数据库备份完成后，需要将备份文件上传到用户自己的OSS Bucket中。

**方法一：使用ossbrowser工具上传**

推荐使用ossbrowser工具上传备份文件到OSS，具体请参考 [ossbrowser](https://www.alibabacloud.com/help/doc-detail/61872.html)。

**方法二：使用OSS控制台上传**

如果备份文件小于5GB，可以直接使用OSS控制台上传。具体请参考[使用OSS控制台上传](https://www.alibabacloud.com/help/doc-detail/31886.html)。

**方法三：使用OSS API上传**

如果您有全自动无人干预的上云需求，请使用OSS OpenAPI，通过断点续传的方式上传备份文件到OSS Bucket，具体请参考[断点续传](https://www.alibabacloud.com/help/doc-detail/31850.html)。

## 创建数据上云任务 {#section_tmd_xfk_zdb .section}

1.  登录[RDS控制台](https://rdsnew.console.aliyun.com/)。
2.  选择目标实例所在地域。
3.  单击目标实例的ID，进入基本信息页面。
4.  在左侧菜单栏中选择**备份恢复**。
5.  单击右上角**OSS备份数据恢复上云**。
6.  如果您是第一次使用OSS备份数据恢复上云功能，需要给RDS官方服务账号授予访问OSS的权限：
    1.  单击数据导入向导第三项数据导入页面中的授权地址，如下图所示：
    2.  跳转到RAM授权页面，请单击**同意授权**，完成授权。
7.  授权完毕后，在数据导入向导第三步数据导入页面设置如下参数，单击**确认**生成OSS备份数据上云任务。

    |配置项|说明|
    |---|--|
    |数据库名|目标实例上的目标数据库名。|
    |OSS Bucket|选择备份文件所在的OSS Bucket。|
    |OSS子文件夹名|备份文件所在的子文件夹名字。|
    |OSS文件列表|单击右侧放大镜按钮，可以按照备份文件名前缀模糊查找，会展示文件名、文件大小和更新时间。请选择需要上云的备份文件。|
    |上云方案|     -   打开数据库（只有一个全量备份文件）：全量上云，指用户仅有一个完全备份文件上云RDS for SQL Server的场景。本操作选择**打开数据库**。

**说明：** 选择此项将调用API [CreateMigrateTask](../../../../intl.zh-CN/API参考/SQL Server备份文件上云/CreateMigrateTask.md#)，且参数BackupMode=FULL，IsOnlineDB=True。

    -   不打开数据库（还有差异备份或日志文件）：增量上云，用户有全量备份文件和差异或者日志备份文件，增量上云RDS for SQL Server的场景。默认选择此项。

**说明：** 选择此项将调用API [CreateMigrateTask](../../../../intl.zh-CN/API参考/SQL Server备份文件上云/CreateMigrateTask.md#)，且参数BackupMode=UPDF，IsOnlineDB=False。

 |
    |一致性检查方式|     -   异步执行DBCC：在打开数据库的时候系统不做DBCC CheckDB，会在打开数据库任务结束以后，异步执行DBCC CheckDB操作，以此来节约打开数据库操作的时间开销（数据库比较大，DBCC CheckDB非常耗时），减少用户的业务停机时间。如果，您对业务停机时间要求非常敏感，且不关心DBCC CheckDB结果，建议使用异步执行DBCC。

**说明：** 选择此项将调用API [CreateMigrateTask](../../../../intl.zh-CN/API参考/SQL Server备份文件上云/CreateMigrateTask.md#)，且参数CheckDBMode=SyncExecuteDBCheck。

    -   同步执行DBCC：相对于异步执行DBCC，有的用户非常关心DBCC CheckDB的结果，以此来找出用户线下数据库数据一致性错误。此时，建议您选择同步执行DBCC，影响是会拉长打开数据库的时间。默认选择此项。

**说明：** 选择此项将调用API [CreateMigrateTask](../../../../intl.zh-CN/API参考/SQL Server备份文件上云/CreateMigrateTask.md#)，且参数CheckDBMode=AsyncExecuteDBCheck。

 |

    ![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/7998/15439173096230_zh-CN.png)

    您可以不断单击刷新按钮，来查看数据上云任务最新状态。如果上云失败，请根据任务描述提示排查错误，可参考本文的常见错误部分。


## 查看备份上云记录 {#section_j5n_bgk_zdb .section}

您也可以查看一段时间内的备份上云记录，具体操作如下：

进入备份恢复页面，选择**备份上云恢复记录**，默认会展示最近一周的记录。当然，您同样可以修改时间范围来查看特定时间段内的上云恢复记录。

## 常见错误 {#section_cgb_fgk_zdb .section}

每一条备份上云恢复记录中，都会有任务描述信息，可以通过这些描述信息提示来发现任务失败或报错的原因，常见的错误信息如下：

**同名数据库已经存在**

-   错误信息：The database \(xxx\) is already exist on RDS, please backup and drop it, then try again.

-   错误原因：为了保证用户RDS for SQL Server上数据的安全性，我们不予许RDS for SQL Server上已经存在同名数据库的上云操作。

-   解决方法：如果用户确实需要对现有数据库的数据进行覆盖，请自行先备份已经存在的数据，然后删除数据库，最后再重新数据上云任务。


**差异备份文件**

-   错误信息：Backup set \(xxx.bak\) is a Database Differential backup, we only accept a FULL Backup.

-   错误原因：用户提供的备份文件是差异备份，不是全量备份文件，一次性全量迁入上云仅支持全量备份文件，不支持差异备份。


**事务日志备份文件**

-   错误信息：Backup set \(xxx.trn\) is a Transaction Log backup, we only accept a FULL Backup.

-   错误原因：用户提供的备份文件是日志备份，不是全量备份文件，一次性全量迁入上云仅支持全量备份文件，不支持日志备份。


**备份文件校验失败**

-   错误信息：Failed to verify xxx.bak, backup file was corrupted or newer edition than RDS.

-   错误原因：备份文件损坏或者备份文件所在的本地环境SQL Server实例版本比RDS for SQL Server版本更高，导致校验失败。比如：用户想将一个来自于SQL Server 2016的备份还原到RDS for SQL Server 2012版本，就会报告这个错误。

-   解决方法：如果是备份文件损坏，请在本地环境重新做一个全量备份，重新生成迁移上云任务；如果是版本过高，请使用与本地环境版本一致或者更高的RDS for SQL Server，比如：将用户本地环境的SQL Server 2012备份上云到RDS for SQL Server 2016上。


**DBCC CHECKDB失败**

-   错误信息：DBCC checkdb failed

-   错误原因：用户备份文件还原到RDS for SQL Server上，DBCC CheckDB检查操作报错，说明用户数据库在本地环境中已经有错误发生。

-   解决方法：

    1.  使用如下命令修复本地环境数据库错误（注意：使用该命令修复错误的过程，可能会导致用户数据丢失。）：

        ```
        DBCC CHECKDB (DBName, REPAIR_ALLOW_DATA_LOSS) WITH NO_INFOMSGS, ALL_ERRORMSGS
        ```

    2.  重新对数据库做一个全量备份。
    3.  将新的全量备份文件上传到OSS。
    4.  在RDS控制台重新执行OSS上云步骤。

**OSS下载链接过期**

OSS下载链接过期错误仅针对RDS for SQL Server 2008 R2高可用版本。

-   错误信息：Failed to download backup file since OSS URL was expired.

-   错误原因：OSS下载链接地址过期导致备份文件下载失败。用户在共享OSS上备份文件下载链接地址时，设置的有效期过短，导致文件还未下载完毕，链接地址过期。

-   解决方法：

    -   方法一：将备份文件OSS共享链接地址的有效期设置为更大的值或者最大值18个小时，方法如下截图所示：

    -   方法二：将OSS上的数据库备份文件直接修改为**公共读**，方法如下图所示。

        **说明：** 修改为公共读的数据库备份文件，是永久可以下载的，所以存在安全风险，请用户在完成备份文件上云后，将该文件还原为**私有**属性。


**空间不足1**

-   错误信息：Not Enough Disk Space for restoring, space left \(xxx MB\) < needed \(xxx MB\)

-   错误原因：用户实例剩余空间不满足备份文件上云所需要的最小空间要求。

-   解决方法：用户升级实例空间。


**空间不足2**

-   错误信息：Not Enough Disk Space, space left xxx MB < bak file xxx MB

-   错误原因：用户实例剩余空间比备份文件本身还要小，不满足最小空间要求。

-   解决方法：用户升级实例空间。


**没有高权限账号**

-   错误信息：Your RDS doesn’t have any init account yet, please create one and grant permissions on RDS console to this migrated database \(XXX\).

-   错误原因：RDS目标实例中，不存在高权限账号，OSS备份数据上云任务不知道需要为哪个用户授权。但是，备份文件已经成功还原到目标实例上，所以任务状态是成功的。

-   解决方法：

    1.  创建高权限账号，具体操作请参考[创建数据库和账号SQL Server 2012/2016版](../../../../intl.zh-CN/快速入门SQL Server版/初始化配置/创建数据库和账号/创建数据库和账号SQL Server 2012__2016版.md#)中的第1步至第7步。
    2.  重置高权限账号密码，具体操作请参考[重置密码](intl.zh-CN/用户指南/账号管理/重置密码.md#)。
    3.  使用高权限账号访问上云的数据库，也可以执行为其他用户授权等操作。


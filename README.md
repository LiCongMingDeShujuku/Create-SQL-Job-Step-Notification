![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 创建SQL作业步骤通知
#### Create SQL Job Step Notification
**发布-日期: 2016年07月01日 (评论)**

![#](images/Create-SQL-Job-Step-Notification-a.png?raw=true "#")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
如果你打开了这篇文章，这意味着你需要一个发送有关特定作业及其历史记录的电子邮件通知，并可能在此过程中添加作业步骤持续时间的解决方案。 下面是一些快速的SQL逻辑（logic），可以帮你节省时间。
好，我估计你会问：”这个逻辑（logic）究竟能干什么呢？”
首先它会检查你是否配置了数据库邮件。如果没有配置，它会自动为你配置。这对你意味着什么呢？这意味着你可以在任何你想用的服务器上创建此步骤，只要你在SQL逻辑（logic）中提供了正确的SMTP服务器名称
（查找MySMTPServer.MyDomain.com），那么大部分情况下它都可以正常工作。它会选择一系列默认值，但如果你的环境中某些默认配置不同，则可能无法正常工作。你需要进行相应的编辑，但它现在设计为按原样工作。
然后，如果你在Management Studio中查看作业历史记录，它会创建一个与你所看到的内容相同的临时表，但会将其格式化为HTML表格，并将它们嵌入到电子邮件中，方便你随时轻松地查看参考。因为它被配置为显示为电子表格视图，
所以眼睛看起来也容易些。这是通过在运行时创建的电子邮件格式中加入一些CSS来实现的。
我已经在SQL 2008,2012和2014的几个服务器上部署了，运行良好。
你需要如何操作呢？
好的，你可以使用以下脚本并更改要获取通知的SMTP服务器名称，收到通知的作业名称。 最后只需一个步骤并将整个脚本粘贴到那里就可以了。
下面是SQL逻辑（logic）：

## English
If you’re here… This means you want a solution to sending an email notification about a particular Job and it’s history and maybe add the Job step durations along the way. Here’s some quick SQL logic to save you the time.
Ok; so I hear you say.. “What does the logic do exactly”
It first checks to see if you have database mail configured. If not; it will automatically configure it for you. What does this mean for you? It means you can create this step on any server you want; and as long as you supply the correct SMTP server name (look for MySMTPServer.MyDomain.com) in the SQL logic then it should work in most cases. It chooses a series of defaults, but if your environment has some of those defaults configured differently then it may not work. All you need to do is edit it accordingly, but it’s designed right now to work as-is.
Then it creates a temporary table which looks on par with what you see in if you were looking at the job history from within Management studio, but this time it formats it into HTML Tables, and imbeds them into an email so you can always refer to it without any fuss. It’s also a lot easier on the eyes cause it’s configured to appear as a spreadsheet view. This is done by providing some CSS in the email formatting which is created at run-time.
I’ve deployed this across several of my servers SQL 2008, 2012, and 2014 and it works great.
How does this work?
Ok so you take the script and change the SMTP server name, Job Name you want to get the notification on. Then simply add a step and paste the entire script in there and you’re done.
Here’s the SQL Logic:


---
## Logic
```SQL
----------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------
-- 	to use this process; simply replace My_Job_Name_Goes_here with the Job name you want to notify you and simply add this as the final step in the job. 
-- 	使用此程序，只需将My_Job_Name_Goes_here替换为你想要收到通知的作业名称并将它添加在作业的最后一步即可。
set nocount on
set ansi_nulls on
set quoted_identifier on
 
----------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------
-- 	Configure SQL Database Mail if it's not already configured.
-- 	配置SQL 数据库邮件（Database Mail）如果它尚未配置的话
if (select top 1 name from msdb..sysmail_profile) is null
    begin
        ----------------------------------------------------------------------------------------
        ----------------------------------------------------------------------------------------
-- 	Enable SQL Database Mail
-- 	启用SQL数据库邮件
        exec master..sp_configure 'show advanced options',1
        reconfigure;
        exec master..sp_configure 'database mail xps',1
        reconfigure;
 
        ----------------------------------------------------------------------------------------
        ----------------------------------------------------------------------------------------
-- 	Add a profile
-- 	添加配置文件
        execute msdb.dbo.sysmail_add_profile_sp
            @profile_name       = 'SQLDatabaseMailProfile'
        ,   @description        = 'SQLDatabaseMail';
 
        ----------------------------------------------------------------------------------------
        ----------------------------------------------------------------------------------------
-- 	Add the account names you want to appear in the email message.
-- 	添加你想显示在邮件信息中的账户名称
        execute msdb.dbo.sysmail_add_account_sp
            @account_name       = 'sqldatabasemail@MyDomain.com'
        ,   @email_address      = 'sqldatabasemail@MyDomain.com'
        ,   @mailserver_name    = 'MySMTPServer.MyDomain.com'  
        --, @port           = ####          --optional
        --, @enable_ssl     = 1         --optional
        --, @username       ='MySQLDatabaseMailProfile' --optional
        --, @password       ='MyPassword'       --optional
 
-- 	Adding the account to the profile
-- 	添加账户到配置文件
        execute msdb.dbo.sysmail_add_profileaccount_sp
            @profile_name       = 'SQLDatabaseMailProfile'
        ,   @account_name       = 'sqldatabasemail@MyDomain.com'
        ,   @sequence_number    = 1;
 
-- 	Give access to new database mail profile (DatabaseMailUserRole)
-- 	授权访问新数据库邮件配置文件(DatabaseMailUserRole)
        execute msdb.dbo.sysmail_add_principalprofile_sp
            @profile_name       = 'SQLDatabaseMailProfile'
        ,   @principal_id       = 0
        ,   @is_default     = 1;
 
        ----------------------------------------------------------------------------------------
        ----------------------------------------------------------------------------------------
-- 	Get Server info for test message
-- 	测试获取服务器信息
 
        declare @get_basic_server_name          varchar(255)
        declare @get_basic_server_name_and_instance_name    varchar(255)
        declare @basic_test_subject_message     varchar(255)
        declare @basic_test_body_message        varchar(max)
        set @get_basic_server_name          = (select cast(serverproperty('servername') as varchar(255)))
        set @get_basic_server_name_and_instance_name= (select  replace(cast(serverproperty('servername') as varchar(255)), '\', '   SQL Instance: '))
        set @basic_test_subject_message     = 'Test SMTP email from SQL Server: ' + @get_basic_server_name_and_instance_name
'从SQL Server发来的测试SMTP电子邮件：'+ @get_basic_server_name_and_instance_name
        set @basic_test_body_message        = 'This is a test SMTP email from SQL Server:  ' + @get_basic_server_name_and_instance_name + char(10) + char(10) + 'If you see this.  It''s working perfectly :)'
 这是来自SQL Server的测试SMTP电子邮件：' + @get_basic_server_name_and_instance_name + char(10) + char(10) + '如果你看到这个，表示它正在完美地工作：）’
        ----------------------------------------------------------------------------------------
        ----------------------------------------------------------------------------------------
-- 	Send quick email to confirm email is properly working.
发送快速邮件来确认邮件正常工作
 
        EXEC msdb.dbo.sp_send_dbmail
            @profile_name   = 'SQLDatabaseMailProfile'
        ,   @recipients = 'SQLJobAlerts@MyDomain.com'
        ,   @subject    = @basic_test_subject_message
        ,   @body       = @basic_test_body_message;
 
-- 	Confirm message send and run this ( select * from msdb..sysmail_allitems )
-- 	确认邮件发送并运行( select * from msdb..sysmail_allitems )
    end
 
----------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------
-- 	get basic server info.
-- 	获取基本的服务器信息
declare @server_name_basic      varchar(255)
declare @server_name_instance_name  varchar(255)
declare @server_time_zone       varchar(255)
set     @server_name_basic      = (select cast(serverproperty('servername') as varchar(255)))
set     @server_name_instance_name  = (select  replace(cast(serverproperty('servername') as varchar(255)), '\', '   SQL Instance: '))
declare @since_midnight         varchar(10) = (select convert(char, getdate(), 110))
declare @since_30_days_back     varchar(10) = (select convert(char, dateadd(day,-30,Getdate()), 110))
declare @tomorrow           varchar(10) = (select convert(char, dateadd(day,1,Getdate()), 110))
----------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------
-- 	set message subject.
-- 	设置短信对象
declare @message_subject    varchar(255)
set @message_subject    = 'Status Report for Job: My_Job_Name_Goes_here on ' + @server_name_instance_name
-- 	工作状态报告
-- 	 
----------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------
-- get total duration of job run across all run-times for the last 30 days.
-- 	 获取作业过去30天内的运行总时间。
if object_id('tempdb..#job_duration_total') is not null
    drop table  #job_duration_total
create table    #job_duration_total
(
    [server]    varchar(255)
,   [job_name]  varchar(255)
,   [day]       varchar(255)
,   [start_time]    varchar(255)
,   [step_id]   varchar(255)
,   [step_name] varchar(255)
,   [status]    varchar(255)
,   [hhmmss]    varchar(255)
)
 
insert into #job_duration_total
select
    [server]        = upper(@@servername)
,   [job_name]      = sj.name
,   [day]           = datename(dw, cast(str(sjh.run_date, 8, 0) as datetime))
,   [start_time]    = replace(replace(left(cast(str(sjh.run_date, 8, 0) as datetime) + cast(stuff(stuff(right('000000' + cast (sjh.run_time as varchar(6)), 6), 5, 0, ':'), 3, 0, ':') as datetime), 19), 'AM', 'am'), 'PM', 'pm')
,   [step_id]       = sjh.step_id
,   [step_name]     = sjh.step_name
--, [end_time]      = replace(replace(left(dateadd(second, ( ( sjh.run_duration / 1000000 ) * 86400 ) + ( ( ( sjh.run_duration - ( ( sjh.run_duration / 1000000 ) * 1000000 ) ) / 10000 ) * 3600 ) + ( ( ( sjh.run_duration - ( ( sjh.run_duration / 10000 ) * 10000 ) ) / 100 ) * 60 ) + ( sjh.run_duration - ( sjh.run_duration / 100 ) * 100 ), cast(str(sjh.run_date, 8, 0) as datetime) + cast(stuff(stuff(right('000000' + cast (sjh.run_time as varchar(6)), 6), 5, 0, ':'), 3, 0, ':') as datetime)), 19), 'AM', 'am'), 'PM', 'pm')
,   [status]        =   case sjh.run_status
                            when 0 then 'Failed'
                            when 1 then 'Succeded'
                            when 2 then 'Retry'
                            when 3 then 'Cancelled'
                            when 4 then 'In progress'
                        end
,   [hhmmss]        = stuff(stuff(replace(str(run_duration, 6, 0), ' ', '0'), 3, 0, ':'), 6, 0, ':')
 
from
    sysjobhistory sjh inner join sysjobs sj on sj.job_id = sjh.job_id
where
    sj.name = 'My_Job_Name_Goes_here'
    and sjh.step_id = '0'
    and msdb.dbo.agent_datetime(run_date, run_time) between @since_30_days_back and @tomorrow
order by
    [start_time] desc
 
----------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------
-- 	get duration per step for current day.
-- 	获取当天每个步骤的持续时间。
 
if object_id('tempdb..#job_duration_steps') is not null
    drop table  #job_duration_steps
create table    #job_duration_steps
(
    [server]    varchar(255)
,   [job_name]  varchar(255)
,   [day]       varchar(255)
,   [start_time]    varchar(255)
,   [step_id]   varchar(255)
,   [step_name] varchar(255)
,   [status]    varchar(255)
,   [hhmmss]    varchar(255)
)
 
insert into #job_duration_steps
select
    [server]        = upper(@@servername)
,   [job_name]      = sj.name
,   [day]           = datename(dw, cast(str(sjh.run_date, 8, 0) as datetime))
,   [start_time]    = replace(replace(left(cast(str(sjh.run_date, 8, 0) as datetime) + cast(stuff(stuff(right('000000' + cast (sjh.run_time as varchar(6)), 6), 5, 0, ':'), 3, 0, ':') as datetime), 19), 'AM', 'am'), 'PM', 'pm')
,   [step_id]       = sjh.step_id
,   [step_name]     = sjh.step_name
--, [end_time]      = replace(replace(left(dateadd(second, ( ( sjh.run_duration / 1000000 ) * 86400 ) + ( ( ( sjh.run_duration - ( ( sjh.run_duration / 1000000 ) * 1000000 ) ) / 10000 ) * 3600 ) + ( ( ( sjh.run_duration - ( ( sjh.run_duration / 10000 ) * 10000 ) ) / 100 ) * 60 ) + ( sjh.run_duration - ( sjh.run_duration / 100 ) * 100 ), cast(str(sjh.run_date, 8, 0) as datetime) + cast(stuff(stuff(right('000000' + cast (sjh.run_time as varchar(6)), 6), 5, 0, ':'), 3, 0, ':') as datetime)), 19), 'AM', 'am'), 'PM', 'pm')
,   [status]        =   case sjh.run_status
                            when 0 then 'Failed'
                            when 1 then 'Succeded'
                            when 2 then 'Retry'
                            when 3 then 'Cancelled'
                            when 4 then 'In progress'
                        end
,   [hhmmss]        = stuff(stuff(replace(str(run_duration, 6, 0), ' ', '0'), 3, 0, ':'), 6, 0, ':')
 
from
    sysjobhistory sjh inner join sysjobs sj on sj.job_id = sjh.job_id
where
    sj.name = 'My_Job_Name_Goes_here'
    and sjh.step_name   not in ('send notification')
    and msdb.dbo.agent_datetime(run_date, run_time) between @since_midnight and @tomorrow
order by
    [start_time] desc
 
----------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------
-- create conditions for html tables in top and mid sections of email.
 在电子邮件的顶部和中间部分为html表格创建条件。
declare @xml_top        NVARCHAR(MAX)
declare @xml_mid        NVARCHAR(MAX)
declare @body_top       NVARCHAR(MAX)
declare @body_mid       NVARCHAR(MAX)
 
----------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------
-- set xml top table td's
-- create html table object for: #job_duration_total
设置xml顶部表格td’s
为#job_duration_steps创建html表格对象
set @xml_top = 
    cast(
        (select
            [server]    as 'td'
        ,   ''
        ,   [job_name]  as 'td'
        ,   ''
        ,   [day]       as 'td'
        ,   ''
        ,   [start_time]    as 'td'
        ,   ''
        --, [step_id]   as 'td'
        ,   ''
        ,   case    [step_name]
                when '(Job Outcome)' then 'Total Job Duration'
然后总的作业时长
                else [step_name]
            end     as 'td'
        ,   ''
        ,   [status]    as 'td'
        ,   ''
        ,   [hhmmss]    as 'td'
        ,   ''
        from  #job_duration_total 
        --order by
        for xml path('tr')
    ,   elements)
    as NVARCHAR(MAX)
        )
 
----------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------
-- set xml mid table td's
-- create html table object for: #job_duration_steps 
设置xml中部表格td’s
为#job_duration_steps创建html表格对象
set @xml_mid = 
    cast(
        (select
            [server]    as 'td'
        ,   ''
        ,   [job_name]  as 'td'
        ,   ''
        ,   [day]       as 'td'
        ,   ''
        ,   [start_time]    as 'td'
        ,   ''
        ,   case    [step_id]
                when '0' then 'Completed'
                else [step_id]
                end as 'td'
        ,   ''
        ,   case    [step_name]
                when '(Job Outcome)' then 'Total Job Duration'
                else [step_name]
            end     as 'td'
        ,   ''
        ,   [status]    as 'td'
        ,   ''
        ,   [hhmmss]    as 'td'
        ,   ''
        from  #job_duration_steps 
        --order by
        for xml path('tr')
    ,   elements)
    as NVARCHAR(MAX)
        )
 
----------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------
-- format HTML email
set @body_top =
        '<html>
        <head>
            <style>
                h1{
                    font-family: sans-serif;
                    font-size: 90%;
                }
 
                h3{
                    font-family: sans-serif;
                    color: black;
                    font-size: 95%
                }
                     
                table, td, tr, th {
                    font-family: sans-serif;
                    border: 1px solid black;
                    border-collapse: collapse;
                    font-size: 88%;
                }
 
                th {
                    text-align: left;
                    background-color: gray;
                    color: white;
                    padding: 5px;
                    font-size: 95%;
                }
 
                td {
                    padding: 5px;
                }
            </style>
        </head>
        <body>
        <H3>' + @message_subject + '</H3>
        <h1>Job Total Durations</h1>
作业总持续时长
        <table border = 1>
        <tr>
<th> Server   </th>
服务器
<th> Job Name </th>
作业名称
<th> Day  </th>
日期
<th> Start Time   </th>
开始时间
<th> Step Name    </th>
步骤名称
<th> Status   </th>
状态
            <th> HHMMSS   </th>
        </tr>'
         
set @body_top = @body_top + @xml_top + '</table>
 
<h1>Job Step Duration History</h1>
作业步骤持续时间记录
 
<table border = 1>
        <tr>
            <th> Server   </th>
            <th> Job Name </th>
            <th> Day  </th>
            <th> Start Time   </th>
            <th> Step ID  </th>
            <th> Step Name    </th>
            <th> Status   </th>
            <th> HHMMSS   </th>
        </tr>'    
         
+ @xml_mid + '</table>'
+ '</body></html>'
 
----------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------
-- 	send email.
发送邮件
 
EXEC msdb.dbo.sp_send_dbmail
    @profile_name   = 'SQLDatabaseMailProfile'
,   @recipients = 'MyAccount@MyDomain.com'
,   @subject    = @message_subject
,   @body       = @body_top
,   @body_format    = 'HTML';
 
drop table #job_duration_total
drop table #job_duration_steps


```

顺便说说,如果要设置更自定义的主题消息，可以将其添加到逻辑流程（就在发送电子邮件过程的上方）
By the way; if you want to set a more custom subject message you can add this to the logic flow (just above the send email process)


 ```SQL
declare @message_subject varchar(255)
set @message_subject =  case
        when exists (select 1 from #job_duration_steps where [step_id] > 0 and [status] in ('Failed', 'Retry', 'In progress')) then ' The Load ETLS job has an Error'
        when exists (select 1 from #job_duration_steps where [step_id] > 0 and [status] in ('Cancelled')) then 'The Load ETLS job was Cancelled'
        else 'The Load ETLS job was Successful'
                end
                +  ' on Server ' + @server_name_instance_name

 ```

这是电子邮件的样子：
Here’s what the email looks like:

![#](images/Create-SQL-Job-Step-Notification-a.png?raw=true "#")


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")


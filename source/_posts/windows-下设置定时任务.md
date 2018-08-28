---
title: windows 下设置定时任务
date: 2018-08-28 16:53:45
categories:
- Windows
tags:
- Windows
---
Linux 系统可以通过crontab -e 设置定时任务，Windows系统没有crontab命令，但是Windows系统有跟crontab命令比较接近的命令： schtasks 命令。

<!--more-->
#### schtasks 语法 ####
##### 创建定时任务 #####
**语法**
`schtasks /create /tn TaskName /tr TaskRun /sc schedule [/mo modifier] [/d day] [/m month[,month...] [/i IdleTime] [/st StartTime] [/sd StartDate] [/ed EndDate] [/s computer [/u [domain\]user /p password]] [/ru {[Domain\]User | "System"} [/rp Password]] /?`

**参数**
/tn TaskName         指定任务的名称。
/tr TaskRun 指定任务运行的程序或命令。键入可执行文件、脚本文件或批处理文件的完全合格的路径和文件名。（如果忽略该路径，SchTasks.exe 将假定文件在 Systemroot\System32 目录下。）
/sc schedule           指定计划类型。有效值为 MINUTE、HOURLY、DAILY、WEEKLY、MONTHLY、ONCE、ONSTART、ONLOGON、ONIDLE。
>值说明
MINUTE、HOURLY、DAILY、WEEKLY、MONTHLY 		指定计划的时间单位。
ONCE 	任务在指定的日期和时间运行一次。
ONSTART 	任务在每次系统启动的时候运行。可以指定启动的日期，或下一次系统启动的时候运行任务。
ONLOGON 	每当用户（任意用户）登录的时候，任务就运行。可以指定日期，或在下次用户登录的时候运行任务。
ONIDLE 		只要系统空闲了指定的时间，任务就运行。可以指定日期，或在下次系统空闲的时候运行任务。

/mo modifier 		指定任务在其计划类型内的运行频率。这个参数对于 MONTHLY 计划是必需的。对于 MINUTE、HOURLY、DAILY 或 WEEKLY 计划，这个参数有效，但也可选。默认值为 1。
>计划类型
修饰符
说明
MINUTE
1 ～ 1439
任务每 n 分钟运行一次。
HOURLY
1 ～ 23
任务每 n 小时运行一次。
DAILY
1 ～ 365
任务每 n 天运行一次。
WEEKLY
1 ～ 52
任务每 n 周运行一次。
MONTHLY
1 ～ 12
任务每 n 月运行一次。
LASTDAY
任务在月份的最后一天运行。
FIRST、SECOND、THIRD、FOURTH、LAST
与 /d day 参数共同使用,并在特定的周和天运行任务。例如，在月份的第三个周三。

/d dirlist 		指定周或月的一天。只与 WEEKLY 或 MONTHLY 计划共同使用时有效。

>计划类型
日期值
WEEKLY
可选项。有效值是 MON ~ SUN 和 * （每一天）。MON 是默认值。
MONTHLY
在使用 FIRST、SECOND、THIRD、FOURTH 或 LAST 修饰符 (/mo) 时，需要 MON ～ SUN 中的某个值。1 ～ 31 是可选的，只在没有修饰符或修饰符为 1 ～ 12 类型时有效。默认值是 1 （月份的第一天）。

/m month[,month...] 		指定一年中的一个月。有效值是 JAN ～ DEC 和 * （每个月）。/m 参数只对于 MONTHLY 计划有效。在使用 LASTDAY 修饰符时，这个参数是必需的。否则，它是可选的，默认值是 * （每个月）。
/i InitialPageFileSize 		指定任务启动之前计算机空闲多少分钟。键入一个 1 ～ 999 之间的整数。这个参数只对于 ONIDLE 计划有效，而且是必需的。
/st StartTime 				以 HH:MM:SS 24 小时格式指定时间。默认值是命令完成时的当前本地时间。/st 参数只对于 MINUTE、HOURLY、DAILY、WEEKLY、MONTHLY 和 ONCE 计划有效。它只对于 ONCE 计划是必需的。
/sd StartDate 				以 MM/DD/YYYY 格式指定任务启动的日期。默认值是当前日期。/sd 参数对于所有的计划有效，但只对于 ONCE 计划是必需的。
/ed EndDate 				指定任务计划运行的最后日期。此参数是可选的。它对于 ONCE、ONSTART、ONLOGON 或 ONIDLE 计划无效。默认情况下，计划没有结束日期。
/s Computer 				指定远程计算机的名称或 IP 地址（带有或者没有反斜杠）。默认值是本地计算机。
/u [domain\]user 			使用特定用户帐户的权限运行命令。默认情况下，使用已登录到运行 SchTasks 的计算机上的用户的权限运行命令。
/p password 				指定在 /u 参数中指定的用户帐户的密码。如果使用 /u 参数，则需要该参数。
/ru {[Domain\]User | "System"} 		使用指定用户帐户的权限运行任务。默认情况下，使用用户登录到运行 SchTasks 的计算机上的权限运行任务。

>值
说明
[domain\}User?
指定用户帐户。
"System" 或 ""
指定操作系统使用的 NT Authority\System 帐户。

/p Password 				指定用户帐户的密码，该用户帐户在 /u 参数中指定。如果在指定用户帐户的时候忽略了这个参数，SchTasks.exe 会提示您输入密码而且不显示键入的文本。使用 NT Authority\System 帐户权限运行的任务不需要密码，SchTasks.exe 也不会提示索要密码。
/? 							在命令提示符显示帮助。

**示例**
（1）计划任务每 20 分钟运行一次。（从脚本创建成功开始计时）
`schtasks /create /sc minute /mo 20 /tn "Security Script" /tr \\central\data\scripts\sec.vbs`

（2）计划命令在每小时过五分的时候运行。
`schtasks /create /sc hourly /st 00:05:00 /tn "My App" /tr c:\apps\myapp.exe`

（3）计划命令每五小时运行一次（它使用 /mo 参数来指定间隔时间，使用 /sd 参数来指定起始日期。）
`schtasks /create /sc hourly /mo 5 /sd 03/01/2001 /tn "My App" /tr c:\apps\myapp.exe`

（4）计划任务每天运行一次
`schtasks /create /tn "My App" /tr c:\apps\myapp.exe /sc daily /st 08:00:00 /ed 12/31/2001`

（5）计划任务每隔一天运行一次（命令使用 /mo 参数来指定间隔天数。使用 /st 参数来指定起始时间， /sd 参数来指定起始日期。）
`schtasks /create /tn "My App" /tr c:\apps\myapp.exe /sc daily /mo 2 /st 13:00:00 /sd 12/31/2001`

（6）计划任务每六周运行一次
`schtasks /create /tn "My App" /tr c:\apps\myapp.exe /sc weekly /mo 6 /s Server16 /ru Admin01`
（该命令使用 /mo 参数来指定间隔。它也使用 /s 参数来指定远程计算机，使用 /ru 参数来计划任务以用户的 Administrator 帐户权限运行。因为忽略了 /rp 参数，SchTasks.exe 会提示用户输入 Administrator 帐户密码。
另外，因为命令是远程运行的，所以命令中所有的路径，包括到 MyApp.exe 的路径，都是指向远程计算机上的路径。）

（7）计划任务每隔一周在周五运行
`schtasks /create /tn "My App" /tr c:\apps\myapp.exe /sc weekly /mo 2 /d FRI`
（下面的命令计划任务每隔一周在周五运行。它使用 /mo 参数来指定两周的间隔，使用 /d 参数来指定是一周内的哪一天。如计划任务在每个周五运行，要忽略 /mo 参数或将其设置为 1。）

（8）计划任务运行一次
`schtasks /create /tn "My App" /tr c:\apps\myapp.exe /sc once /st 00:00:00 /sd 01/01/2002 /ru Admin23 /rp p@ssworD1`
（下面的命令计划 MyApp 程序在 2002 年 1 月 1 日午夜运行一次。它使用 /ru 参数指定以用户的 Administrator 帐户权限运行任务，使用 /rp 参数为 Administrator 帐户提供密码。）

（9）计划任务在每次系统启动的时候运行（下面的命令计划 MyApp 程序在每次系统启动的时候运行，起始日期是 2001 年 3 月 15 日。）
`schtasks /create /tn "My App" /tr c:\apps\myapp.exe /sc onstart /sd 03/15/2001`

（10）计划任务在用户登录到远程计算机的时候运行
`schtasks /create /tn "Start Web Site" /tr c:\myiis\webstart.bat /sc onlogon /s Server23`
（下面的命令计划批处理文件在用户（任何用户）每次登录到远程计算机上的时候运行。它使用 /s 参数指定远程计算机。因为命令是远程的，所以命令中所有的路径，包括批处理文件的路径，都指定为远程计算机上的路径。）

（11）计划某项任务在计算机空闲的时候运行
`schtasks /create /tn "My App" /tr c:\apps\myapp.exe /sc onidle /i 10`
（下面的命令计划 MyApp 程序在计算机空闲的时候运行。它使用必需的 /i 参数指定在启动任务之前计算机必需持续空闲十分钟。）


##### 立即执行计划任务 #####
立即运行计划任务。run 操作忽略计划，但使用程序文件位置、用户帐户和存储在任务中的密码立即运行任务。
**语法**
`schtasks /run /tn TaskName [/s computer [/u [domain\]user /p password]] /?`
**示例：**
在本地计算机上运行任务
下面的命令启动 "Security Script" 任务。
`schtasks /run /tn "Security Script"`

在远程计算机上运行任务
下面的命令在远程计算机 Svr01 上运行 Update 任务：
`schtasks /run /tn Update /s Svr01`


##### 终止由任务启动的程序 #####
**语法**
`schtasks /end /tn TaskName [/s computer [/u [domain\]user /p password]] /?`
**示例**
终止本地计算机上的任务
下面的命令终止由 My Notepad 任务启动的 Notepad 实例：
`schtasks /end /tn "My Notepad"`
终止远程计算机上的任务
下面的命令终止远程计算机 Svr01 上由 InternetOn 任务启动的 Internet Explorer 实例：
`schtasks /end /tn InternetOn /s Svr01`


##### 删除计划任务 #####
**语法**
`schtasks /delete /tn {TaskName | *} [/f] [/s computer [/u [domain\]user /p password]] [/?]`
**示例**
（1）从远程计算机上的计划表中删除任务
下面的命令从远程计算机上的计划表中删除 "Start Mail" 任务。它使用 /s 参数来标识远程计算机。
`schtasks /delete /tn "Start Mail" /s Svr16`

（2）删除所有为本地计算机计划的任务。
下面的命令从本地计算机的计划表中删除所有的任务，包括由其它用户计划的任务。它使用 /tn * 参数代表计算机上所有的任务，使用/f 参数取消确认消息。
`schtasks /delete /tn * /f`


##### 更改计划任务 #####
更改一个或多个下列任务属性。
* 任务运行的程序 (/tr)。
* 任务运行的用户帐户 (/ru)。
* 用户帐户的密码 (/rp)。

**语法**
`schtasks /change /tn TaskName [/s computer [/u [domain\]user /p password]] [/tr TaskRun] [/ru [Domain\]User | "System"] [/rp Password]`
**示例**
（1）更改任务运行的程序
下面的命令将 Virus Check 任务运行的程序由 VirusCheck.exe 更改为 VirusCheck2.exe。此命令使用 /tn 参数标识任务，使用 /tr 参数指定任务的新程序。（不能更改任务名称。）
`schtasks /change /tn "Virus Check" /tr C:\VirusCheck2.exe`

（2）更改远程任务的用户密码
下面的命令更改用于远程计算机 Svr01 上 RemindMe 任务的用户帐户密码。命令使用 /tn 参数标识任务，使用 /s 参数指定远程计算机。它使用 /rp 参数指定新的密码 p@ssWord3。
在用户帐户密码过期或更改的时候需要此过程。如果存储在任务中的密码无效，那么任务不会运行。
`schtasks /change /tn RemindMe /s Svr01 /rp p@ssWord3`


##### 显示计划任务 #####
显示计划在计算机上运行的所有任务，包括那些由其它用户计划的任务。
**语法**
`schtasks [/query] [/fo {TABLE | LIST | CSV}] [/nh] [/v] [/s computer [/u [domain\]user /p password]]`


#### Windows下查看定时任务 ####
![](/uploads/2018/08/windows_schtasks.png)


#### Windows自定义定时任务 ####
（1）自定义脚本文件 ".bat"
```
c:
cd \schtasks
D:\dev\Go\src\myTest\ceshi.exe >> D:\go.txt
```
**说明：**
在定时执行的bat文件开头加几行命令，先进入存放配置文件的目录。
（schtasks的默认其实路径为：C:\Windows\System32）

（2）设置定时任务
`schtasks /create /sc minute /mo 1 /tn "windows_crontab" /tr d:\schtasks\ceshi.bat`

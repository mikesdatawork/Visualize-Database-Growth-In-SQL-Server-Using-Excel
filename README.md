![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Visualize Database Growth In SQL Server Using Excel
**Post Date: July 30, 2015**      



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>How to visualize a database growth chart using Excel. Here's a straight up 'How To' write up to get you started. You'll love this. It's easy I promise. Notice the following graph. 2 lines RED and BLUE. RED represents the current compression ratio, and because compression hasn't been configured for the database they represent the same level as the usual backup size which is in BLUE. As you can see; they are parallel through the history of the database until it gets to July 2015 where I enabled compression.</p>

![Visualize Database Growth With Excel And Excel]( https://mikesdatawork.files.wordpress.com/2015/07/image002.jpg "Visualize Database Growth With SQL")
 
![]( https://mikesdatawork.files.wordpress.com/2015/07/image002.jpg "") 
Below is some SQL logic from a former post about trending your database growth using backups (https://mikesdatawork.wordpress.com/2015/07/27/get-database-size-trending-on-all-databases/) This was inspired by the original author Erin Stellato at SQL Skills (http://www.sqlskills.com/blogs/erin/trending-database-growth-from-backups/). Check that out for more info :)
By the way; you can enable backup compression with the following statement.


## SQL-Logic
```SQL
exec master..sp_configure 'show advanced options' 1 reconfigure;
 
exec master..sp_configure 'backup compression default', 1 reconfigure;
 
go
```
I adjusted the SQL logic so it could be run against all databases on a server ( or group of servers if registered under a single folder in SSMS ). I've also included some modifications from 'Jeffs' post within the original article. Basically I added an extra 365 days to it. From there you can get a pretty good idea how to modify the logic to your liking and reach back as far as needed without too much hassle.</p>   


   
## SQL-Logic
```SQL
use master;
set nocount on
declare @get_dbsize_history varchar(max)
declare @first_day_of_year varchar(25)
set @first_day_of_year = (select dateadd(year, datediff(year, 0, getdate()), 0)) -365 --> Go back to the first of the year plus 1 extra year (365 days)
set @get_dbsize_history = ''
select @get_dbsize_history = @get_dbsize_history +
'
 
select
''database'' = cast(upper([database_name]) as varchar(20))
, ''month'' = convert(varchar(7),[backup_start_date],120)
, ''backup size gb'' = str(avg([backup_size]/1024/1024/1024),5,2)
, ''compressed bu size'' = str(avg([compressed_backup_size]/1024/1024/1024),5,2)
, ''compression ratio'' = str(avg([backup_size]/[compressed_backup_size]),5,2)
 
from
msdb.dbo.backupset
where
[database_name] = ''' + upper(name) + '''
and [type] = ''d''
and backup_finish_date > ''' + @first_day_of_year + '''
and database_name in (select name from master.sys.databases)
group by
[database_name]
, convert(varchar(7),[backup_start_date],120)
order by
[database_name],convert(varchar(7),[backup_start_date],120);' + char(10) + char(10)
from
sys.databases
where
database_id > 4
 
exec (@get_dbsize_history)
```
So here's what you do. You find a server that has a variety of backup sizes through the history you're looking at. These make the best examples to illustrate growth history. Copy the results from SSMS (using a single database history) and paste them straight into Excel. Nothing crazy, or difficult. In this example we are using a server called "MyServerName", and a single database called "MyDatabase" which is basically about 1.5 years history. It might help to insert an extra row and add the Column titles.

![List Backup Info In Excel]( https://mikesdatawork.files.wordpress.com/2015/07/image0032.jpg "list backup info in excel")
 
![](https://mikesdatawork.files.wordpress.com/2015/07/image0032.jpg"")
Perform the following actions.
1. Highlight 3 columns Month, Backup Size, Compressed
2. Click 'Insert'.
3. Click 'Line' chart.
4. Select '3-D Line' chart.

![Select Excel Chart Type]( https://mikesdatawork.files.wordpress.com/2015/07/image005.jpg "Select Excel Chart Type")
 
![](https://mikesdatawork.files.wordpress.com/2015/07/image005.jpg"")
Change the charting.
1. Click 'Chart Tools', and select the 'Design' tab.
2. Select 'Style 42' below.
Feel free to drag the chart out a bit to make it more visible.

![Modify Excel Chart Style]( https://mikesdatawork.files.wordpress.com/2015/07/image006.jpg "modify chart style")
 
Next we want to change the chart layout slightly. Perform the following actions.
1. Select layout '6'.
2. Double click on the inner corner of the chart to get the format properties.
3. Select '3-D Rotation' on the left, and change the [X Rotation] to 30.
4. Click 'Close'.

![Modify Chart Bar Width]( https://mikesdatawork.files.wordpress.com/2015/07/image007.jpg "Modify Chart Bar Width")
 
Looking at the chart it seems like we are almost done. Lets punch up those lines a bit and add some width. To do so; perform the following actions.
1. Double click the line to bring up the Data Point properties.
2. Click on 'Series Options' on the left, and drag the handle all the way to the left towards 'No Gap'.

![Select Excel Chart Scale Options]( https://mikesdatawork.files.wordpress.com/2015/07/image008.jpg "Select Excel Chart Scale Options")
 
Now just change the titles and you're done. Showing your capacity and storage requirements will be much easier now not just with backups, but with any standing database growth or otherwise. This can also be used to track performance over time from sys.dm_os_performance_stats etc.

![Add Chart Titles]( https://mikesdatawork.files.wordpress.com/2015/07/image009.jpg "Add Chart Titles")
 



[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

      
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")


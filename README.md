# backup-force.com

The backup-force.com is a command line tool for automated backups as well as configurable data retrieval from salesforce forked from the [neowit verion](https://github.com/neowit/backup-force.com) so I could upgrade the SF API.
It uses Force.com Web Services/SOAP API and/or Bulk API.  
Amongst other things this tool overcomes a well known shortcoming of Apex DataLoader - with backup-force.com you can automate incremental data retrieval for Objects that support LastModifiedDate.  
Data is retrieved in .csv format.  
  
Usage examples:  
* Backup all (or specific) data from all salesforce objects into .csv files  
* Incremental Backup - backup data from all or specific objects modified since previous backup  
* Extract data (full or incremental) from a list of all/specified objects and post-process each resulting .csv file using user defined scripts  
See [Wiki/Recipes](https://github.com/gigaguy/backup-force.com/wiki/Recipes) and  [./config/sample-configuration.properties](https://github.com/gigaguy/backup-force.com/blob/master/config/sample-configuration.properties) file for further inspiration.


## System requirements

[Java](http://java.com/download) JDK/JRE, Version 6.1 or greater

## Running

* Visit [releases](https://github.com/gigaguy/backup-force.com/releases) page and download `backup-force.com-<version>.jar`
* create config file using [./config/sample-configuration.properties](https://github.com/gigaguy/backup-force.com/blob/master/config/sample-configuration.properties) as an example
* execute process (in this example I use version 0.1)  
  `java -jar backup-force.com-0.1.jar --config /full/path/to/myconf.properties`  
  or if you want to direct output to a file instead of console:  
  `java -jar backup-force.com-0.1.jar --config /full/path/to/myconf.properties > log.txt`  

## Video instructions   
* [Installation](http://youtu.be/eyIkdsSBpLY) - describes initial installation and first launch
* [Basic config example](http://youtu.be/ptMc-7hp5qA) - an example of backup-force.com configuration to extract/backup data from 2 selected Salesforce.com objects
* see [Wiki](https://github.com/gigaguy/backup-force.com/wiki) for further information

## Features

* Extract full content of selected or All objects
* Specify global or per object-type WHERE clause
* Wildcard `*` support for easy field list definition, e.g. `select * from Account`, as well as to include all objects without listing them, e.g. `backup.objects=*`
  - field wildcard is particularly useful because it makes your config resistant to Org config changes (e.g. field removed/added)
* Config file is order of magnitude smaller and simpler than process-conf.xml used by Apex DataLoader.  
	Here is an example of full config (excluding login credentials) which allows to extract 3 fields from Accounts, all fields from Opportunities with Amount > 100 and all fields from all Contacts into 3 separate .csv files:
<pre>
    outputFolder=/home/user/extract
    backup.objects=Account, Contact, Opportunity
    backup.soql.Account=select Id, Name, Description from Account
    backup.soql.Opportunity=select * from Opportunity where Amount > 100
</pre>
* Incremental data retrieval for Objects that support LastModifiedDate field    
    See `$Object.LastModifiedDate` in ./config/sample-configuration.properties
* Multiple config files  
  In some cases you may want to re-use same connection details with multiple extract/backup configurations. For such cases you can specify more than one --config parameters on the command line. Values from all specified config files will be merged (if there are conflicts then last config wins). For example    
   `… --config ./extract.properties --config ./connection.properties`
* Shell Expressions evaluation - shell commands can be embedded in all config values in order to make configuration more dynamic, for example, here is how you could define dynamically created output folder for daily incremental extract/backup:
<pre>
    outputFolder=/home/user/extract/\`date +%Y%m%d-%H%M\`
</pre>
Note: on systems that do not have Unix [date](http://en.wikipedia.org/wiki/Date_(Unix) utility (e.g. MS Windows) you may need to use an alternative, e.g. date.exe from UnixUtils).
    
* Hooks - execute user defined scripts  
	- before/after main process start
	- before/after each object type  
Using hooks you can specify custom scripts to archive retrieved .csv files, write custom log events, validate results, etc. See ./config/sample-configuration.properties

## Limitations

Current version will report `java.lang.OutOfMemoryError` (same as Apex DataLoader) on large files. In some cases this can be mitigated by increasing java heap size, e.g. to make it 2048MB (assuming your computer has enough RAM) use `java -Xmx2048m -jar …`.  
However it may still fail on retrieval of Chatter/Content/Files (ContentVersion object) when they are tens of megabytes in size.  
Present workaround (if your Org has very large files 20MB+ and the above trick with `-Xmx` parameter does not work if your machine does not have enough RAM) is to limit the the file size in a config query, e.g.  
`backup.soql.ContentVersion=select * from ContentVersion where ContentSize < 20000000`  

 

## Building backup-force.com
    git clone git@github.com:gigaguy/backup-force.com.git
    mvn clean package

## Legal stuff

Copyright (c) 2013, Andrey Gavrikov. All rights reserved.

License: LGPL v3 <http://www.gnu.org/licenses/>

Third-Party Licenses:  
* [Force.com Web Service Connector (WSC)](https://github.com/forcedotcom/wsc) - see [LICENSE.md](https://github.com/forcedotcom/wsc/blob/master/LICENSE.md)  
* [scalalogging](https://github.com/typesafehub/scalalogging) - [Apache 2.0 License](https://github.com/typesafehub/scalalogging#license)  
* [slf4j](http://www.slf4j.org/) - [MIT license](http://www.slf4j.org/license.html)  
* [Logback](http://logback.qos.ch/) - dual-licensed under the [EPL v1.0 and the LGPL 2.1](http://logback.qos.ch/license.html)  

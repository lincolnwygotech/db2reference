## DB2 Command Line Processor / db2cmdadmin.exe
Use db2cmdadmin.exe to open an admin cmd prompt and invoke the DB2 CLP
- The IBM install directory should be registed in your PATH
- **`[Win]-R` → `db2cmdadmin`** or open from the IBM install directory

**[db2 - Command line processor invocation command](https://www.ibm.com/docs/en/db2/11.5?topic=clp-db2-invocation)**

The db2 command starts the command line processor and can be started in:
- Interactive input mode, characterized by the `db2 =>` input prompt
- Command mode, where each command must be prefixed by `db2`
- Batch mode, which uses the `-f` file input option.

**This guide will utilize Command mode, where all commands are prefixed by `db2`**

## Variables
*Replace the following values with the proper values in your environment*
- **`<dbname>`** → The name of your database e.g. `TESTDB`
- **`<user>`** → The db2 user - Typically DB2ADMIN
- **`<password>`** → The user's password
- **`<nodename>`** → The name of the note - Typically NODE0000
- **`<hostname>`** → The hostname or IP address of the DB2 server
- **`<port>`** → The listing port for the DB2 service - Typically 50000
- **`<onlinepath>`** → The full path for your online backups e.g. `L:\DB2Backups\<dbname>\Online`
- **`<offlinepath>`** → The full path for your offline backups e.g. `L:\DB2Backups\<dbname>\Offline`
- **`<activelogpath>`** → The full path for your active logs e.g. `L:\DB2Logs\<dbname>\Active`
- **`<archivelogpath>`** → The full path for your archive logs e.g. `L:\DB2Logs\<dbname>\Archive`
- **`<fullbackuppath>`** → The full path for the backup file, including the filename
- **`<apphandle>`** → Application handle ID from the `db2 list applications` command
- **`<dbdrive>`** → The storage drive on the db2 host where the database resides
- **`<templogpath>`** → Temporary folder to store logs for a roll forward operation (The path must exist before running restore/rollforward command)
- **`<takentimestamp>`** → The timestamp from the backup image - e.g. if the backup filename is `<dbname>.0.DB2.DBPART000.20220707230004.001` then the timestamp would be `20220707230004`
- **`[option]`** → Optional parameters

## Basics
- Commands are **NOT** case-sensitive
- [List node directory](https://www.ibm.com/docs/en/db2/11.5?topic=commands-list-node-directory) → `db2 list node directory`
- [List database directory](https://www.ibm.com/docs/en/db2/11.5?topic=commands-list-database-directory) → `db2 list node directory`
- [Catalog remote node](https://www.ibm.com/docs/en/db2/11.5?topic=commands-catalog-tcpip-node) → `db2 catalog tcpip node <nodename> remote <hostname> server <port>`
- [Catalog database on locale host]() → `db2 catalog database <dbname> on <dbdrive>`
- [Catalog database on a remote node](https://www.ibm.com/docs/en/db2/11.5?topic=commands-catalog-database) → `db2 catalog database <dbname> at node <nodename>`
- [Connect to database](https://www.ibm.com/docs/en/db2/11.5?topic=clp-command-line-processor-features) → `db2 connect to db <dbname> user <user> [using "<password>"]`
- [Disconnect from database](https://www.ibm.com/docs/en/db2/11.5?topic=clp-command-line-processor-features) → `db2 connect reset`
- [Activate a database](https://www.ibm.com/docs/en/db2/11.5?topic=commands-activate-database) → `db2 activate db <dbname>`
- [Deactivate a database](https://www.ibm.com/docs/en/db2/11.5?topic=commands-deactivate-database) → `db2 deactivate db <dbname>`
- [Terminate CLP processes and connections](https://www.ibm.com/docs/en/db2/11.5?topic=commands-terminate) → `db2 terminate`
- [Start](https://www.ibm.com/docs/en/db2/11.5?topic=commands-db2start-start-db2)/[Stop](https://www.ibm.com/docs/en/db2/11.5?topic=commands-db2stop-stop-db2) DB2 Service → `db2start` / `db2stop [force]`
- [List active applications](https://www.ibm.com/docs/en/db2/11.5?topic=commands-list-applications) → `db2 list applications [for db <dbname>] [show detail]`
- [List active utilities](https://www.ibm.com/docs/en/db2/11.1?topic=commands-list-utilities) (such as a backup or restore process) → `db2 list utilities [show detail]`
- [Force an application to disconnect](https://www.ibm.com/docs/en/db2/11.5?topic=commands-force-application) → `db2 force application [all / <apphandle>]`
- [Enable restricted mode](https://www.ibm.com/docs/en/db2/11.5?topic=commands-quiesce-database-using-admin-cmd) → `db2 quiesce db <dbname> [immediate force connections]`
- [Disable restricted mode](https://www.ibm.com/docs/en/db2/11.5?topic=commands-unquiesce-database-using-admin-cmd) → `db2 unquiesce db <dbname>`
- [Check Backup](https://www.ibm.com/docs/en/db2/11.5?topic=commands-db2ckbkp-check-backup) → `db2ckbkp "<fullbackuppath>"`
- [Show Database Backup History](https://www.ibm.com/docs/en/db2/11.5?topic=commands-list-history) → `db2 list history [backup / rollforward / archive log] [all / since <timestamp>] for db <dbname>`
- [Exit the DB2 CLP](https://www.ibm.com/docs/en/db2/11.5?topic=commands-quit) → `quit`

## Configs
- [View database manager configuration](https://www.ibm.com/docs/en/db2/11.5?topic=commands-get-database-manager-configuration) → `db2 get dbm cfg [show detail]`
- [View database configuration](https://www.ibm.com/docs/en/db2/11.5?topic=commands-get-database-configuration) → `db2 get db cfg for <dbname> [show detail]`
### Extras
- Export to a file → `... > C:\temp\dbcfg.txt`
- Find specific option (e.g. LOGARCHMETH1) → `... | find "LOGARCHMETH1"`


## [BACKUP DATABASE](https://www.ibm.com/docs/en/db2/11.5?topic=commands-backup-database)
There are two types of backups we can perform using DB2 - offline and online
- Offline backups require users and applications to be disconnected from the databases thereby requiring a certain down time 
- Online backups on the other hand, can be performed while users are still connected to databases

### Online backup
**Confirm your database is configured for online backups**

1. Ensure archive logging is enabled
   - `db2 get db cfg for <dbname> | find "LOGARCHMETH1"`
     - If this value is set to `OFF`, then archive logging is not enabled.
     - If this values is a file path, then archive logging is enabled.
2. Enable archive logging, if necessary
   - `db2 update db cfg for <dbname> using LOGARCHMETH1 "<archivelogpath>"`
   - Setting this parameter enables archive logging and stores the archive logs in a location different from the primary/active logs.
   - ***Once this change is made, a full offline backup must be taken before any connections can be made - The database goes into a 'BACKUP_PENDING' state***

**Generic online backup command**

1. Backup database → `db2 backup database <dbname> online to "<onlinepath>" with 2 buffers buffer 1024 parallelism 1 [compress] without prompting`
2. Verify backup → `db2chkbkp "<fullbackuppath>"`

### Offline backup
1. Connect to the database → `db2 connect to <dbname> user <user> [using "<password>"]`
2. Quiesce database → `db2 quiesce database immediate force connections`
3. Disconnect from database → `db2 connect reset`
4. Deactivate database → `db2 deactivate database <dbname>`
5. Backup database → `db2 backup database <dbname> to "<offlinepath>" with 2 buffers buffer 1024 parallelism 1 compress without prompting`
6. Verify backup → `db2ckbkp "<fullbackuppath>"`
7. Reactivate database → `db2 activate database <dbname>`
8. Connect to the database → `db2 connect to <dbname> user <user> [using "<password>"]`
9. Unquiesce database → `db2 unquiesce database <dbname>`
10. Disconnect → `db2 connect reset`

## [RESTORE DATABASE](https://www.ibm.com/docs/en/db2/11.5?topic=commands-restore-database)
**Generic restore command**

`db2 restore database <dbname> from [<offlinepath> / <onlinepath>] taken at <takentimestamp> to C: into <newdbname> newlogpath with 2 buffers buffer 1024 without prompting`

**Restore to a different drive**

`db2 restore database <dbname> from [<offlinepath> / <onlinepath>] taken at <takentimestamp> on C: into <newdbname> newlogpath with 2 buffers buffer 1024 without prompting`

**Restore and extract logs**

`db2 restore database <dbname> from [<offlinepath> / <onlinepath>] taken at <takentimestamp> to C: into <newdbname> logtarget <templogpath> with 2 buffers buffer 1024 without prompting`

### [ROLLFORWARD](https://www.ibm.com/docs/en/db2/11.5?topic=commands-rollforward-database)

**Generic rollforward command**

`db2 rollforward db <dbname> to end of logs and complete overflow log path (<templogpath>) noretrieve`

# pg_backrest

Simple Postgres Backup and Restore

## planned for next release

* Default restore.conf is written to each backup.

* Able to set timeout on ssh connection in config file.

* Fix bug where .backup files written into old directories can cause the archive process to error.

* Add configurable sleep to archiver process to reduce ssh connections.

## release notes

### v0.18: Return soft error from archive-get when file is missing

* The archive-get function returns a 1 when the archive file is missing to differentiate from hard errors (ssh connection failure, file copy error, etc.)  This lets Postgres know that that the archive stream has terminated normally.  However, this does not take into account possible holes in the archive stream.

### v0.17: Warn when archive directories cannot be deleted

* If an archive directory which should be empty could not be deleted backrest was throwing an error.  There's a good fix for that coming, but for the time being it has been changed to a warning so processing can continue.  This was impacting backups as sometimes the final archive file would not get pushed if the first archive file had been in a different directory (plus some bad luck).

### v0.16: RequestTTY=yes for SSH sessions

* Added RequestTTY=yes to ssh sesssions.  Hoping this will prevent random lockups.

### v0.15: Added archive-get

* Added archive-get functionality to aid in restores.

* Added option to force a checkpoint when starting the backup (start_fast=y).

### v0.11: Minor fixes

Tweaking a few settings after running backups for about a month.

* Removed master_stderr_discard option on database SSH connections.  There have been occasional lockups and they could be related issues originally seen in the file code.

* Changed lock file conflicts on backup and expire commands to ERROR.  They were set to DEBUG due to a copy-and-paste from the archive locks.

### v0.10: Backup and archiving are functional

This version has been put into production at Resonate, so it does work, but there are a number of major caveats.

* No restore functionality, but the backup directories are consistent Postgres data directories.  You'll need to either uncompress the files or turn off compression in the backup.  Uncompressed backups on a ZFS (or similar) filesystem are a good option because backups can be restored locally via a snapshot to create logical backups or do spot data recovery.

* Archiving is single-threaded.  This has not posed an issue on our multi-terabyte databases with heavy write volume.  Recommend a large WAL volume or to use the async option with a large volume nearby.

* Backups are multi-threaded, but the Net::OpenSSH library does not appear to be 100% threadsafe so it will very occasionally lock up on a thread.  There is an overall process timeout that resolves this issue by killing the process.  Yes, very ugly.

* Checksums are lost on any resumed backup. Only the final backup will record checksum on multiple resumes.  Checksums from previous backups are correctly recorded and a full backup will reset everything.

* The backup.manifest is being written as Storable because Config::IniFile does not seem to handle large files well.  Would definitely like to save these as human-readable text.

* Absolutely no documentation (outside the code).  Well, excepting these release notes.

* Lots of other little things and not so little things.  Much refactoring to follow.
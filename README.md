# Web
Web-facing tool/page development

## Setting up an instance of TM's DB and web front end

The server should be running an instance of MySQL server (5.7.22 as of this writing) and the apache web server (2.4.33 as of this writing).  The web server needs to have PHP enabled (version 7.2.5 as of this writing) with the mysqli, ctype, and json extensions, and mod_php72 to get the loadable module.  The remaining instructions assume apache, MySQL, and PHP are all working together properly.  On FreeBSD, this involved installing the correct packages.

### MySQL database

First, create the users needed for the database.  Connect and authenticate as root to the MySQL server.  Create passwords for an account that will have permission to administer the TM database that will be used for DB updates and an account that will have only read permission that will be used by the web front end.  In the example below, these use the TM default names "travmapadmin" and "travmap".

```
CREATE USER 'travmapadmin'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YOURPASSWORDFORTHISUSER';
CREATE USER 'travmap'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YOURPASSWORDFORTHISUSER';
```

To be able to update the DB remotely by ssh, a file ~/.my.cnf can be created on the server with the password for travmapadmin:

```
[clienttmapadmin]
password = YOURPASSWORDFORTHISUSER
```

Be sure the permissions are 600, so only readable by the user.

Next, we create the database and give these users needed permissions.

```
CREATE DATABASE TravelMapping;
GRANT SELECT ON `TravelMapping`.* TO 'travmap'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, REFERENCES ON `TravelMapping`.* TO 'travmapadmin'@'localhost'
```

If you have a TravelMapping.sql file (generated by the site update process), you can now try to use it to populate the TravelMapping database.

```
mysql --defaults-group-suffix=tmapadmin -u travmapadmin TravelMapping < TravelMapping.sql
```

Normally, the above will be run as part of the site update process but it is done here as a way to test the database setup.

### Web server files

The files in this Web repository should be placed in a directory served by the web server.  We assume /home/www/tm and a vhosts entry that Apache uses to direct a URL to use files from this location.  The updateserver.sh can help populate and later update this directory.  Note that the fonts directory is not updated by this script, and those files will need to be transferred separately.

In addition to the files in the repository, the file lib/tm.conf needs to be created.  This file contains five lines:

<pre>
Line 1: DB name (likely TravelMapping)
Line 2: DB read-only user (likely travmap)
Line 3: DB read-only user password
Line 4: DB hostname (likely localhost)
Line 5: Google Maps API key
</pre>

This file needs to be readable by the web server but should not be served by the web server.  Configure Apache to ensure this.

Also create a file motd in the root of the directory server.  This is the "message of the day" for TM.

Finally, create a directory named cache in the shields directory.  This needs to be writable by the web server.

Now test it out.  Hopefully everything will work!

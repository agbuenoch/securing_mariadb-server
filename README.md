# securing_mariadb-server
A step-by-step practical guide to creating secure user accounts, managing privileges, and enabling advanced audit logging in MariaDB.

In today’s threat landscape, databases are prime targets. Whether it’s an insider threat or an external breach attempt, controlling who can access your database and monitoring their actions is essential for any Cybersecurity Analyst.

This project walks through a real-world simulation of how to:
- Create and manage users in MariaDB server.
- Apply the principle of least privilege.
- Restrict remote access to the MariaDB server.
- Enable advanced audit logging.
- Review and analyse logs for security incident detection.

By combining proactive access control with reactive audit strategies, this project demonstrates how a cybersecurity analyst can secure critical systems and generate forensic-ready reports.

Ensure the MariaDB Server is up and running. Read [How to set up Chinook Database on MariaDB](https://www.linkedin.com/pulse/how-set-up-chinook-database-mariadb-server-using-linux-enoch-agbu-8cgff/) on how to start, stop, enable and disable the MariaDB server.


## Step 1: Setup and Import Chinook Database.
If you have the MariaDB Server or MySQL Server already set up, you can skip to `Step 2`; otherwise, read the article [How to set up Chinook Database on MariaDB](https://www.linkedin.com/pulse/how-set-up-chinook-database-mariadb-server-using-linux-enoch-agbu-8cgff/) to implement the `Step 1`, where you will install the MariaDB Server, download and import the Chinook database. 

## Step 2: Create Users and assign Permissions.
The SQL statement 
```
SHOW DATABASES;
```
will display all the databases in the MariaDB server. The 1st arrow points to the `Database column`, and within the column is the list of preinstalled databases that come with the MariaDB server, which hold important files and configuration settings.

The following databases, `information_schema, mysql, performance_schema` and `sys` are all default databases that come pre-installed with the MariaDB server. They contain important files and configurations like user privileges related to the MariaDB server.<br>
[view step2A](screenshots/step2A.png)

## STEP 3: Log in to MariaDB.
First, view the list of existing users who can access and manage the databases in the MariaDB server, but note that only users with root privileges can view the list. This will give the admin or root user an overview of the users currently authorised to access the MariaDB server.

In the second command, we added authentication_strings to retrieve more detailed information as pointed to by the 2nd arrow. You can specify more columns, like the password that you wish to retrieve.

The query 
```
SELECT User, Host FROM mysql.user;
```
will display all users, including any user you created. Notice we have a user called `root` on the `localhost` as pointed to by the 1st arrow.
[view step3A](screenshots/step3A.png)

Note that `mysql` is the database name while `user` is the table name found inside the `mysql database`, hence the reason we used `mysql.user`. 

You can switch back to the mysql database using the query 
```
USE mysql;
```
and view the tables in the mysql database using 
```
SHOW TABLES;
```
then run 
```
SELECT User, Host FROM user;
```
to display all the users. 

From the screenshot below, notice that we did not use `FROM mysql.user` but just `FROM user`, this is because we are currently inside the desired database `mysql` as pointed by the 1st arrow and the 2nd arrow points to the user table inside the mysql database.<br>
[view step3B](screenshots/step3B.png)

## STEP 4: Create New User.
The following commands
```
CREATE USER 'firstuser'@'localhost' IDENTIFIED by firstpassword;

CREATE USER 'seconduser'@'%' IDENTIFIED by secondpassword;

CREATE USER 'testinguser'@'%' IDENTIFIED by testingpassword;
```
will create new users and will be recognised by the password you provide.

The query `CREATE USER` will create a new user called `firstuser` on the `localhost` and assigned the password `firstpassword`.  The same applies to others.

These users are created for the `Chinook` database ONLY, as you can see, it is the Chinook database that is currently mounted while creating the users.<br>
[view step4A](screenshots/step4A.png)

The `@’localhost'` in the query `‘firstuser’@’localhost'` means firstuser can only connect to the database from the local machine.<br>
The `@’%’` in the query `‘seconduser’@’%’` will allow seconduser to connect to the database from ANY host/machine.

Use the query
```
DROP USER 'testing'@'%';
```
to remove the `testinguser`.

To verify if testinguser has been dropped, query
```
SELECT User, Host, password FROM mysql.user;
```
The 1st arrow points to the authorised users, but the `testinguser` has been removed or dropped and deleted the user and all associated permissions. The 2nd arrow points to the columns in the user table we specified in the query. The 3rd arrow points to the hashed password for the remaining new users we created.<br>
[view step4B](screenshots/step4B.png)

The string `*826954EC52E6900DB7AC23C8151ED1A5F8E85715` is a hashed representation of the user's password, stored in an encrypted format for security. 

In MariaDB, passwords are hashed using algorithms like `SHA-1` or `SHA-2` before being saved in the `mysql.user` table. This protects the actual password, ensuring that only the hash, which is computationally difficult to reverse, is stored. This approach enhances database security by keeping users' real passwords hidden.

## STEP 5: Grant Permissions to the Users.
We can grant specific privileges to users of specific databases.

The  following queries
```
GRANT ALL PRIVILEGES ON Chinook.* TO 'firstuser'@'localhost';

GRANT ALL PRIVILEGES ON *.* TO 'thirduser'@'localhost';

GRANT SELECT, INSERT, UPDATE ON Chinook.* TO 'seconduser'@'%';

FLUSH PRIVILEGES;
```
will grant ALL privileges to the `firstuser` for `ONLY` the Chinook database as pointed to by the 1st arrow. The second query will grant ALL permissions to the thirduser for ALL databases in the MariaDB server as pointed to by the 2nd arrow. The `*.*` means all databases in the MariaDB server. The third query grants limited permissions `(SELECT, INSERT, UPDATE)` to the `seconduser` for the Chinook database, pointed to by the 3rd arrow.

The query `FLUSH PRIVILEGES` will apply the changes made as pointed to by the 4th arrow.<br>
[view step5A](screenshots/step5A.png)

We can view users’ privileges in MariaDB by querying the `information_schema.user_privileges` table. The query
```
SELECT * FROM information_schema.user_privileges
WHERE GRANTEE = 'thirduser'@'localhost';
```
will show privileges assigned to the thirduser.

`information_schema` is the database name and `user_privileges` is the table name, hence the `information_schema.user_privileges`. Feel free to mount the `information_schema` database and view its tables and table columns.

NOTE: The `user_privileges` provides a `server-wide permission` view and not a `database-specific permission`. Therefore, the query above will output the global privileges assigned to users across the MariaDB server, not tied to any specific database.<br>
[view step5B](screenshots/step5B.png)

Or we could view all users' privileges in the MariaDB server like this.
```
SELECT * FROM information_schema.user_privileges;
```
[view step5C1](screenshots/step5C1.png)<br>
[view step5C2](screenshots/step5C2.png)

The `GRANTEE` column lists all the users.

The `USAGE` privilege means that the user has no specific or explicit privileges granted at the level being queried `(i.e. server-wide or global)`, other than the ability to connect to the database. This is often seen when a new user is created but hasn’t been assigned any particular privileges like `SELECT, INSERT, UPDATE or ALL PRIVILEGES`. The `USAGE` privilege effectively allows login access without granting any permissions to interact with or modify database content as pointed to by the 1st arrow.

>Notice that we granted `ALL PRIVILEGES` to the `firstuser` for `Chinook` database, but from the screenshot above under the `GRANTEE`, the `‘firstuser’@’localhost` privilege_type is showing `USAGE`, this is because the firstuser have no explicit privileges granted for this level, which is for the entire databases or MariaDB server, it privileges were specifically for the Chinook database and not for all the databases (server-wide) in the MariaDB server. But the thirduser was granted ALL PRIVILEGES for all databases (server-wide) in the MariaDB server. If we want to view user privileges specific to a database, use the SCHEMA_PRIVILEGES table instead of the USER_PRIVILEGES table. This will be demonstrated shortly below.

The `IS_GRANTABLE` column in the user_privileges table indicates whether a user can grant a specific privilege to other users, as pointed to by the 2nd arrow. If `IS_GRANTABLE` is set to `YES`, it means the user can grant that particular privilege to others as pointed to by 4th arrow. If it’s `NO`, the user can use the privilege but cannot pass it on, as pointed to by the 3rd arrow.

This is controlled by the `GRANT OPTION` privilege in MariaDB, which allows a user to grant their privileges to others.

Hence, from the output above, we can see that the `‘mysql’@’localhost'` user can assign privileges to other users, meanwhile `‘thirduser’@’localhost'` cannot because its `IS_GRATABLE` value is `NO` for each privilege.

The `SCHEMA_PRIVILEGES` and `USER_PRIVILEGES` tables in `information_schema` database serve different purposes.<br>
1. `SCHEMA_PRIVILEGES`: Contains privileges specific to individual databases (schemas). It details permissions granted to users or roles for operations on a database, including GRANTOR, PRIVILEGE_TYPE, and IS_GRANTABLE.<br>
2. `USER_PRIVILEGES`: Provides a broader view of global privileges assigned to users across the server, not tied to any specific database. It reflects permissions like SUPER, CREATE USER, or server-wide privileges.

Use `SCHEMA_PRIVILEGES` for database-specific queries and USER_PRIVILEGES for server-wide permission analysis. The term `SCHEMA` is referred to as `DATABASE`, therefore, `SCHEMA_PRIVILEGES` means `database privileges`. The query
```
SELECT * 
FROM information_schema.SCHEMA_PRIVILEGES
WHERE GRANTEE = 'firstuser'@'localhost' AND TABLE_SCHEMA = 'Chinook';
```
will return the `firstuser` database privileges associated with the Chinook database.<br>
[view step5D](screenshots/step5D.png)

**Mount and explore the information_schema database.**
The `SHOW DATABASES` displays all databases in the MariaDB server. The 1st and 2nd arrows point to two important default or preinstalled databases that come with the MariaDB server.<br>
[view step5E](screenshots/step5E.png)

The query 
```
USE information_schema;
```
will mount the database on the MariaDB server as pointed to by the 1st arrow.
```
SHOW TABLES;
```
will display all the tables in the information_schema database as pointed to by the 2nd arrow. The 3rd arrow points to the USER_PRIVILEGES, which is one of many tables in the database.<br>
[view step5F1](screenshots/step5F1.png)<br>
[view step5F2](screenshots/step5F2.png)

Explore the other databases and their tables to see other MariaDB server files and configuration settings. For example, the `mysql` database has tables like `user`, and `db` that you can explore. 

`information_schema` Database: This is a virtual database that provides metadata about the database server. It stores information about databases, tables, columns, data types, privileges, constraints, and other schema-related details. Tables in `information_schema` database are dynamically generated views, not stored on disk.

`mysql` Database: This is a system database containing user accounts, privileges, roles, and other essential system configurations. It stores tables like `mysql.user` for user accounts, mysql.db` for database privileges, and `mysql.tables_priv` for table-level privileges, among others.


## Step 6: Revoking Users' Permissions.
Before we revoke the firstuser permissions, let's have a view of the firstuser permissions by querying
```
SELECT *
FROM SCHEMA_PRIVILEGES
WHERE GRANTEE = 'firstuser'@'localhost';
```
The 1st arrow points to the users column and 2nd arrow points to the privileges with the `SELECT` privilege appearing first in the column.<br>
[view step6A](screenshots/step6A.png)

The query 
```
REVOKE SELECT ON Chinook.* FROM ‘firstuser’@'localhost';
```
Will only revoke the `SELECT` permission. As pointed out by the 2nd arrow, the `SELECT` privilege is no longer available.<br>
[view step6B](screenshots/step6B.png)

Meanwhile, the query:
```
REVOKE ALL PRIVILEGES, GRANT OPTION FROM firstuser@localhost;

FLUSH PRIVILEGES;
```
will revoke all privileges that have been granted to the `firstuser` on the `Chinook` database. As pointed to by the 1st and 2nd arrow, the firstuser is not seen among the user list because all its privileges have been revoked.

This ensures the user can no longer perform any actions like `SELECT, INSERT`, etc., and cannot grant privileges to others.<br>
[view step6C](screenshots/step6C.png)

**Explanation:**
- `REVOKE`: Is the SQL command used to take away privileges.
- `SELECT`: Is the specific privilege being revoked.
- `ON Chinook.*`: Targeting all tables (*) in the Chinook database.
- `FROM firstuser@localhost`: This specifies the user and host combination.

After revoking, it’s good practice to flush privileges so changes take effect `FLUSH PRIVILEGES`;

Let's grant all the privileges back to firstuser and then view all users' privileges to confirm it. Run the two sets of queries separately as shown in the screenshot.
```
GRANT ALL PRIVILEGES ON Chinook.*
TO firstuser@localhost;

SELECT * FROM SCHEMA_PRIVILEGES;
```
[view step6D](screenshots/step6D.png)

As shown above, the `firstuser` have been granted all privileges again.

## STEP 7: Test the New Users.
From the screenshot above in `STEP 3`, we have assigned `ALL PRIVILEGES` to `thirduser` for all databases in the MariaDB server, so let's log out and log in as thirduser.

The command `exit` or `EXIT` will terminate the database console and return to the Linux console as pointed to by the 1st arrow. The command
```
sudo mariadb -u thirdsuer -p;
```
will log in as the `thirduser` after passing its password as pointed to by the 2nd arrow.<br>
[view step7A](screenshots/step7A.png)

Now we are logged in as the `thirduser`. Since the `thirduser` has `ALL PRIVILEGES` with `server-wide` or `global privileges scope`, let's run a `SELECT` command on the Chinook database.<br>
[view step7B](screenshots/step7B.png)

Everything works fine as the `thirduser` was able to log in and run queries as expected.

## Step 8: Harden and Enable Logging for Detection & Analysis.
Let's edit the `50-server.cnf` file and configure any desired security settings of our choice. But let's first understand the contents of this file, and this is very important.

In the `50-server.cnf` file or any MariaDB/MySQL config file, `settings` are grouped under `sections` as pointed to by the 1st, 2nd and 3rd arrow, each identified by a header in square brackets, like `[mysqld]`. Each section applies to a specific component or version.<br>
[view step8A1](screenshots/step8A1.png)

**Here's a quick breakdown:<br>**
- `[server]`: This applies to all server-type programs (e.g., mysqld, mysqld_safe, mariadbd, mongod etc.). It is used to apply or share generic settings across all server modes. It is often used as a global section.
- `[mysqld]`: This section applies only to the main MariaDB/MySQL server daemon (mysqld). It is where you typically define ports, data directory, logging, connection limits, SQL modes, and buffer sizes, among other settings. This is the most important section for database server behaviour, and in this project, this is where most of our configuration settings will be applied.
- `[embedded]`: This applies to the embedded MariaDB library `(libmysqld)`. It is used when MariaDB is embedded inside an application (a rare use case).
- `[mariadb]`: This applies specifically to MariaDB server instances (not the generic MySQL). The settings here will override `[mysqld]` if both sections define the same setting. This section is used when you want MariaDB-specific tuning.
- `[mariadb-10.6]`: This applies only to `MariaDB version 10.6` and `below`. It is useful when running multiple MariaDB versions on the same machine or preparing version-specific tuning. This will override both `[mariadb]` and `[mysqld]` for that version.

### What is a daemon in Linux?
A `daemon`, pronounced `day-mon` is a background process that runs without direct user interaction and starts automatically at boot time or when needed. It performs ongoing system or service tasks quietly.

**Examples of daemons:**
- `mysqld`: Runs the MariaDB/MySQL database server.
- `sshd`: Handles SSH connections.
- `httpd or nginx`: Runs a web server.
- `crond`: Runs scheduled tasks (cron jobs).
- `systemd`: Controls startup and background services.

Basic characteristics of a `daemon` are: It runs in the background, it has a name ending in `d` (like `mysqld, sshd`), it is usually started by `init/systemd` or a `service manager`, and does not have a graphical interface.

In MariaDB context, the MariaDB daemon is `mysqld`. It waits for database connections, and processes queries, manages users, reads config files, etc. We interact with it indirectly via the mysql command-line client.

The screenshot below shows the path/directory to the `50-server.cnf` file, where we can apply configuration settings for MariaDB. The 1st arrow points to the directory inside the `mysql` directory, and the 2nd arrow points to the `50-server.cnf` file inside the `mariadb.conf.d` directory.

Therefore, log out of the database and edit the `50-server.cnf` file by running: 
```
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
[view step8A2](screenshots/step8A2.png)

At the moment, notice the `mysql` directory is currently empty with no files or directories as pointed to by the arrow below.<br>
[view step8A3](screenshots/step8A3.png)

Open the `50-server.cnf` file and provide your `sudo` password for authentication.<br>
[view step8B1](screenshots/step8B1.png)

The 1st arrow points to the file currently opened, and the 2nd arrow points to a brief description about the file and its contents.<br>
[view step8B2](screenshots/step8B2.png)

Within the `[mysqld]` section, restrict MariaDB to only `localhost connections`. Look for where `bind-address` is commented and remove the `#` symbol before it. If the configuration setting cannot be found, type it:
```
bind-address = 127.0.0.1
```
Also, disable `symbolic-links`. In Linux, a symbolic link (or symlink) is like a shortcut or pointer to another file or directory. Within the `[mysqld]` group insert:
```
symbolic-links=0
```
[view step8B3](screenshots/step8B3.png)

**Auditing** actions taken by a MariaDB user is a crucial step in database security. This can be achieved by analysing query logs, enabling the audit plugin, or using MariaDB's general or slow query log, depending on the setup.

While still in the 50-server.cnf file, inside the [mysqld] section as pointed to by the 1st arrow, insert the configuration settings below. But where any settings are already written and commented, you do not have to rewrite them, just uncomment the configurations by erasing the hash symbol #.
```
general_log = 1
general_log_file = /var/log/mysql/query.log
```
Note that this configuration setting only records/captures users' `logging activities`.

Then, restart the MariaDB Server by running the command below:
```
sudo systemctl restart mariadb
```
[view step8B4](screenshots/step8B4.png)

After editing the `50-server.cnf file and restarted the `mariadb server`, let's confirm the `query.log` file is created in the `/var/log/mysql/` directory. The `query.log` file has been created as pointed out by the 1st arrow.<br>
[view step8B5](screenshots/step8B5.png)

Let’s log in as the `seconduser` and perform some activities by querying the `Chinook` database.

Notice that currently there is no single activity recorded in the `query.log` file about the `seconduser` yet, when we run the query
```
grep seconduser /var/log/mysql/query.log
```
as no output was returned, as pointed out by the arrow below.<br>
[view step8C](screenshots/step8C.png)

I will provide two `wrong passwords` and finally log in with the correct password for the `seconduser`, perform some queries and let's find out if it will be captured in the query.log file.<br>
[view step8D](screenshots/step8D.png)

Let's exit the database and view the `query.log` file to confirm if the seconduser logging activities were captured. As pointed out by the 1st and 2nd arrows directly at the logging time, access was denied because I entered the wrong password. But the third attempt, as pointed out by 3rd arrow, the seconduser successfully logged in.<br>
[view step8E](screenshots/step8E.png)

Use the `grep` command to search for activity performed by the `seconduser`.
```
grep seconduser /var/log/mysql/query.log
```
Or to get a real-time view
```
sudo tail -f /var/log/mysql/query.log | grep seconduser
```
This will show what queries were executed, on what database/table, timestamps (if configured), login and logout events.

>The general log can grow quickly because it logs everything. Avoid keeping it on in production for long periods. It's best used for auditing user activity, debugging application-database interactions, security investigations (e.g., detecting `SQL injections` or `misuse`).

### Enable MariaDB Audit Plugin (Advanced Audit Logging).
If not already installed, you can enable the `MariaDB Audit Plugin` for deeper auditing.

Log in to the MariaDB Server and run the query:
```
INSTALL SONAME 'server_audit'
```
As pointed out by the 1st arrow, the query was successfully executed. After the installation, run 
```
SHOW PLUGINS
```
to view the list of all plugins as pointed to by the 2nd, 3rd and 4th arrow.<br>
[view step8F1](screenshots/step8F1.png)<br>
[view step8F2](screenshots/step8F2.png)

`SONAME` stands for `Shared Object Name` `.so`. In MariaDB/MySQL, SONAME refers to a shared library (`.so` file) that extends the server's functionality, often through plugins.

Configure the plugin by editing the file `/etc/mysql/mariadb.conf.d/50-server.cnf` and insert within the `[mysqld]` section, the configuration settings below as pointed to by the 1st arrow.
```
server_audit_logging=ON
server_audit_events=CONNECT,QUERY
server_audit_excl_users=mysql server_audit_file_path=/var/log/mysql/audit.log
```
[view step8G](screenshots/step8G.png)

Then, restart MariaDB by running `sudo systemctl restart mariadb`.<br>
[view step8H](screenshots/step8H.png)

Now we will get detailed logs of all queries and connections made by all users logged in to MariaDB.

### Review the Audit Logs.
Notice that when we extract the `firstuser` activities from the `audit.log` file, it returns nothing as pointed to by the 1st arrow, because since the creation of the `audit.log` we have yet to perform any activities as the firstuser.<br>
[view step8I](screenshots/step8I.png)

Log in as firstuser and provide the firstuser password as pointed to by the 1st arrow. I deliberately provided a wrong password and was denied access as pointed to by the 2nd arrow.<br>
[view step8J](screenshots/step8J.png)

We finally logged in and performed some queries like showing the databases on the MariaDB Server as pointed out by 1st arrow, and mounting the Chinook database on the server as pointed out by 2nd arrow.<br>
[view step8K](screenshots/step8K.png)

Run the command below to extract the firstuser activities from the audit.log file.
```
less /var/log/mysql/audit.log
```
Or use filters:
```
grep firstuser /var/log/mysql/audit.log
```
The 1st arrow points to the first failed login attempt I tried earlier above. The 2nd arrow points to when we successfully logged in. The 3rd arrow points to when the firstuser run the query SHOW DATABASES. The 4th arrow points to when the firstuser mounted the Chinook database (i.e when the Chinook database was selected). The last arrow points to when the firstuser exited the database.<br>
[view step8L](screenshots/step8L.png)

The audit file presents you with the timestamp, hostname, username and the query executed by a user.

### Generate a Report (Optional).
You can create a structured report of the user’s actions by running :
```
grep 'firstuser' /var/log/mysql/query.log > firstuser_activity.log
```
The 1st arrow below points to the file created from the command above, which contains the firstuser activities or footprint in the MariaDB Server.<br>
[view step8M](screenshots/step8M.png)

Auditing user actions is vital for Accountability, Threat hunting, Forensics after incidents, and Security compliance.

## Summary.
By the end of this project, you will have enabled logging and monitoring, practised log forensics and system auditing.

## LinkedIn Article.
- [Securing MariaDB Server](https://www.linkedin.com/pulse/securing-mariadb-server-enoch-agbu-prslf)

## Connect with me.
[🔗 LinkedIn](https://www.linkedin.com/in/agbuenoch)<br>
[🔗 X](https://www.x.com/agbuenoch)


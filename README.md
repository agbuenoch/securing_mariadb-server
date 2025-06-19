# securing_mariadb-server
A step-by-step practical guide to creating secure user accounts, managing privileges, and enabling advanced audit logging in MariaDB.

In today’s threat landscape, databases are prime targets. Whether it’s an insider threat or an external breach attempt, controlling who can access your database and monitoring their actions is essential for any Cybersecurity Analyst.

This project walks through a real-world simulation of how to:
- Create and manage users in MariaDB.
- Apply the principle of least privilege.
- Harden the database by restricting remote access.
- Enable advanced audit logging.
- Review and analyse logs for security incident detection

By combining proactive access control with reactive audit strategies, this project demonstrates how a cybersecurity analyst can secure critical systems and generate forensic-ready reports.

The following steps will guide you in securing the Chinook Database. It is assumed:
The MariaDB server has been started and is enabled.
You know how to view and load databases in the MariaDB server.
If not, refer to this article How to set up Chinook Database on MariaDB on how to start, stop, enable and disable the MariaDB server.

## Step 1: Setup and Import Chinook Database.
Read the article (How to set up Chinook Database on MariaDB)[https://www.linkedin.com/pulse/how-set-up-chinook-database-mariadb-server-using-linux-enoch-agbu-8cgff/) to implement the Step 1, where you will Install MariaDB, Download and Import Chinook DB.


## Step 2: Create Users and assign Permissions.
The SQL statement (SHOW DATABASES;) will display all the databases in the MariaDB server. The 1st arrow points to the “Database” column and within the column is the list of preinstall databases that come with the MariaDB server, which holds important files and configuration settings.
The following databases information_schema, mysql, performance_schema and sys are all default databases that come pre-install with the MariaDB server. They contain important files and configurations like “user privileges” related to the MariaDB server.<br>
[view step2A](screenshots/step2A.png)











## LinkedIn Article.
- [Securing Chinook Database on MariaDB Server]()

## Connect with me.
[🔗 LinkedIn](https://www.linkedin.com/in/agbuenoch)<br>
[🔗 X](https://www.x.com/agbuenoch)


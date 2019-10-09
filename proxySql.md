# Proxy Sql Setup
## Start proxysql
1. start proxysql services and enable run at boot
   ```
   sudo systemctl start proxysql
   sudo systemctl enable proxysql
   ```
2. check the service status with
   ```
   sudo systemctl status proxysql
   ```
## Setting administrative account on proxysql
We need admin user on proxysql to manage the configuration, adding host sql server, etc.
1. Login with default password on proxysql command line
   ```
   mysql -u admin -p -h 127.0.0.1 -P 6032 --prompt='ProxySqlAdmin>'
   ```
   and enter the default `admin` password on prompt
2. change admin password to anything you want
   ```
   UPDATE global_variables SET variable_value='admin:proxyadmin' WHERE variable_name='admin-admin_credentials';
   ```
   enable and write the changes we made
   ```
   LOAD ADMIN VARIABLES TO RUNTIME;
   SAVE ADMIN VARIABLES TO DISK;
   ```
3. now you can login to proxysql with credentials you made
   ```
   mysql -u admin -p -h 127.0.0.1 -P 6032 --prompt='ProxySqlAdmin>'
   ```
## Configuring Proxysql monitoring
on one of mysql server, do these steps below
1. download the script to make setting up mysql server easier
   ```
   curl -OL https://gist.github.com/lefred/77ddbde301c72535381ae7af9f968322/raw/5e40b03333a3c148b78aa348fd2cd5b5dbb36e4d/addition_to_sys.sql
   ```
2. run the script 
   ```
   mysql -u root -p < addition_to_sys.sql
   ```
3. now login to mysql cli, and create new user to monitor mysql status that can be accessed by proxysql
   ```
   mysql -u root -padmin
   ```
   ```
   CREATE USER 'monitor'@'%' IDENTIFIED BY 'monitorpassword';\
   GRANT SELECT on sys.* to 'monitor'@'%';
   FLUSH PRIVILEGES;
   ```
   you can change the password `monitorpassword` to anything you like.
   you can check created user with this.
   ```
   select user, host from mysql.user;
   ```
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2010-09-19.png)
ssh to our proxysql server and do steps below: 
1. set the monitor username the same as what we create befor.
   ```
   UPDATE global_variables SET variable_value='monitor' WHERE variable_name='mysql-monitor_username';
   ```
2. save the changes we made
   ```
   LOAD MYSQL VARIABLES TO RUNTIME;
   SAVE MYSQL VARIABLES TO DISK;
   ```
## Add our server to server pool on proxysql
1. set the grouping and enabling monitoring on the all the host
   ```
   INSERT INTO mysql_group_replication_hostgroups (writer_hostgroup, backup_writer_hostgroup, reader_hostgroup, offline_hostgroup, active, max_writers, writer_is_also_reader, max_transactions_behind) VALUES (2, 4, 3, 1, 1, 3, 1, 100);
   ```
   here we set the group for offline to `1`, writer to `2` and reader to `3`. we can now add all server to any group we want.
2. insert the mysql server to server pool
   ```
   INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (2, '10.1.15.143', 3306);
   INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (2, '10.1.15.144', 3306);
   INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (2, '10.1.15.145', 3306);
   ```
   here we add all 3 of our server to writer group so any read/write query can go to all server
3. dont forget to save the changes
   ```
   LOAD MYSQL SERVERS TO RUNTIME;
   SAVE MYSQL SERVERS TO DISK;
   ```
4. now we can check the status of each mysql server from proxysql
   ```
   SELECT hostgroup_id, hostname, status FROM runtime_mysql_servers;
   ```
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2010-50-04.png)
## Create user for accessing database
1. Create user on mysql server
   ```
   CREATE USER 'adhityairvan'@'%' IDENTIFIED BY 'etsbdt';
   GRANT ALL PRIVILEGES on * to 'adhityairvan'@'%';
   FLUSH PRIVILEGES;
   EXIT;
   ```
   and try to login with your new user
2. Create the same user on your proxysql server
   ```
   INSERT INTO mysql_users(username, password, default_hostgroup) VALUES ('adhityairvan', 'etsbdt', 2);
   LOAD MYSQL USERS TO RUNTIME;
   SAVE MYSQL USERS TO DISK;
   ```
3. Now try connecting to your proxysql using the created user. Dont forget, you need to connect to port 6033 for client connection.
   ```
   mysql -u adhityairvan -p -h 127.0.0.1 -P 6033 --prompt='ProxySQLClient> '
   ```
   you can change the ip to your proxysql ip if you try this command from your host machine (not from your ssh session)
   
   and you can check which mysql server handling your query
   ```
   SELECT @@hostname;
   ```
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2011-03-20.png)

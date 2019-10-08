# Installation
## Preparation
We will use vagrant to create virtual machine and use provision script to install basic mysql and proxysql to the virtual machine

1. Install vagrant
    
    ```
    sudo apt install vagrant -y
    ```
2. install virtualbox
   
   virtualbox can be installed from ubuntu app center!
3. Write vagrantfile 
   
   ```
   # MySQL Cluster dengan 3 node
    (1..3).each do |i|
        config.vm.define "mysql#{i}" do |node|
        node.vm.hostname = "mysql#{i}"
        node.vm.box = "bento/ubuntu-16.04"
        node.vm.network "private_network", ip: "10.1.15.143#{i}"
        
        node.vm.provider "virtualbox" do |vb|
            vb.name = "mysql#{i}"
            vb.gui = false
            vb.memory = "512"
        end
    
        node.vm.provision "shell", path: "provision.sh", privileged: false
        end
    end
    ```
    this is the example of vagrantfile you can make, for full config file, you can see **HERE**
4. write provision script (.sh) to automate set up for each machine. Create two script, one for mysql (Full script **Here**) and one for the proxysql (**Here**)
5. start vagrant machine with command : 
   ```
   vagrant up
   ```

## Setting Multi Master Mysql config
1. Generate UUID for mysql group name
   ```
   uuidgen
   ```
   copy output of this command! we will need it later
2. change configuration on each mysql server.
   1. open and edit /etc/mysql/my.cnf
   2. write this setting
    ```
    [mysqld]

    # General replication settings
    gtid_mode = ON
    enforce_gtid_consistency = ON
    master_info_repository = TABLE
    relay_log_info_repository = TABLE
    binlog_checksum = NONE
    log_slave_updates = ON
    log_bin = binlog
    binlog_format = ROW
    transaction_write_set_extraction = XXHASH64
    loose-group_replication_bootstrap_group = OFF
    loose-group_replication_start_on_boot = OFF
    loose-group_replication_ssl_mode = REQUIRED
    loose-group_replication_recovery_use_ssl = 1

    # Shared replication group configuration
    loose-group_replication_group_name = ""
    loose-group_replication_ip_whitelist = ""
    loose-group_replication_group_seeds = ""

    # Single or Multi-primary mode? Uncomment these two lines
    # for multi-primary mode, where any host can accept writes
    #loose-group_replication_single_primary_mode = OFF
    #loose-group_replication_enforce_update_everywhere_checks = ON

    # Host specific replication configuration
    server_id = 
    bind-address = ""
    report_host = ""
    loose-group_replication_local_address = ""
    ```
    3. on the config, change the group replication name, ip white list and group seed according to our generated uuid and ip of our servers
    
        example config
    ```
    # Shared replication group configuration
    loose-group_replication_group_name = "b5af2ebc-c65e-4a30-b0bc-dfa69a7eb501"
    loose-group_replication_ip_whitelist = "10.1.15.143,10.1.15.144,10.1.15.145"
    loose-group_replication_group_seeds = "10.1.15.143:33061,10.1.15.144:33061,10.1.15.145:33061"
    ```
    4. uncomment these lines to enable multi master replication mode
    ```
    #loose-group_replication_single_primary_mode = OFF
    #loose-group_replication_enforce_update_everywhere_checks = ON
    ```
3. change this specific config on the server
   
   on my.cnf
   ```
   # Host specific replication configuration
   server_id = 1
   bind-address = "10.1.15.143"
   report_host = "10.1.15.143"
   loose-group_replication_local_address = "10.1.15.143:33061"
   ```
   change the ip to your server address, and set the server id to be increamental for each server. do this on each mysql server
4. dont forget to restart all the mysql services!

## Configuring mysql 
1. login to mysql console using your password
   ```
   mysql -u root -p
   ```
2. turn off replication for sometime while we do first time setup on one server. create replication user on each server
   ```
   SET SQL_LOG_BIN=0;
   CREATE USER 'repl'@'%' IDENTIFIED BY 'replication' REQUIRE SSL;
   GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
   FLUSH PRIVILEGES;
   SET SQL_LOG_BIN=1;
   CHANGE MASTER TO MASTER_USER='repl', MASTER_PASSWORD='replication' FOR CHANNEL 'group_replication_recovery';
   ```
   you can change the password with any password you want.
3. install group replication plugin
   ```
   INSTALL PLUGIN group_replication SONAME 'group_replication.so';
   ```
4. Starting group replication
   1. do this on single server only
    ```
    SET GLOBAL group_replication_bootstrap_group=ON;
    START GROUP_REPLICATION;
    SET GLOBAL group_replication_bootstrap_group=OFF;
    ```
    and you can check the group status with this command
    ```
    SELECT * FROM performance_schema.replication_group_members;
    ```
    2. start group replication on other servers
    ```
    START GROUP_REPLICATION;
    ```
    3. you can check group members with same command.
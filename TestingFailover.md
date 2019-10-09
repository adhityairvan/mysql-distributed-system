# Testing dan Simulasi Failover
## Membuat Database 
query akan dijalankan melalui proxysql untuk sekaligus mencoba apakah set up proxy sql nya berhasil
1. Connect melalui proxy sql
   ```
   mysql -u adhityairvan -p -h 127.0.0.1 -P 6033 --prompt='ProxySQLClient> '
   ```
2. Buat sebuah test database beserta satu table dengan data didalam nya
   ```
   CREATE DATABASE bdt;
   USE bdt;
   CREATE TABLE mahasiswa ( id smallint unsigned not null auto_increment, nama varchar(50) not null, nrp varchar(20) not null, constraint pk_example primary key (id) );
   INSERT INTO mahasiswa (id, nama, nrp) VALUES(null, 'Adhitya Irvansyah', '05111540000143');
   ```
3. check di ketiga server apakah pembuatan data berhasil
   ```
   SELECT * from bdt.mahasiswa;
   ```
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2011-35-30.png)
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2011-35-59.png)
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2011-36-25.png)
Pengujian diatas menunjukan bahwa proxysql yang kita buat berjalan dengan baik

## Failover Multi-Master test
1. Lihat status pada proxysql dan mysql group
   ```
   SELECT hostgroup_id, hostname, status FROM runtime_mysql_servers;
   ```
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2012-22-05.png)
   group status pada mysqlserver
   ```
   SELECT * FROM performance_schema.replication_group_members;
   ```
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2012-23-38.png)
2. matikan mysql service pada salah satu server
   ```
   sudo systemctl stop mysql
   ```
   hasil nya, status pada proxysql akan menunjukan bahwa salah satu server mati
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2012-25-30.png)
3. Lakukan penulisan data pada server yang masih hidup
   ```
   USE bdt;
   INSERT INTO mahasiswa (id, nama, nrp) VALUES(null, 'Faris Bahdlor', '05111540000147');
   SELECT * from bdt.mahasiswa;
   ```
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2012-27-53.png)
4. Hidupkan server kembali
   ```
   sudo systemctl start mysql
   ```
   lihat status pada proxysql akan kembali online
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2012-30-02.png)
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2012-34-05.png)
5. Cek apakah perubahan data nya sudah di sesuaikan 
   ```
   SELECT * from bdt.mahasiswa;
   ```
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-09%2012-30-16.png)
   *jangan lupa jalankan query diatas agar status pada proxysql admin nya di refresh

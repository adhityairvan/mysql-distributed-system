# App Installation on Mysql Multimaster Database
## Create VM for Webserver
### What we need
1. Apache
2. PHP7.3
3. Mysql-client
4. composer
### step by step
1. edit vagrantfile to add new VM
2. write provisioning script to automate installing the required app
   ```
   # Changing the APT sources.list to kambing.ui.ac.id
   sudo cp '/vagrant/sources.list' '/etc/apt/sources.list'

   sudo apt-get install software-properties-common
   sudo add-apt-repository ppa:ondrej/php -y

   # Updating the repo with the new sources
   sudo apt-get update -y

   #install app
   sudo apt-get install apache2
   sudo apt-get install php7.3 libapache2-mod-php7.3 php7.3-cli php7.3-mysql php7.3-gd php7.3-imagick php7.3-recode php7.3-tidy php7.3-xmlrpc php7.3-common php7.3-curl php7.3-mbstring php7.3-xml php7.3-bcmath php7.3-bz2 php7.3-intl php7.3-json php7.3-readline php7.3-zip
   sudo apt-get install git

   curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
   ```
3. ssh into created vm
4. symlink our apps to vm storage
   ```
   sudo ln -s /vagrant/ApplicationPOS /var/www/html/laravel
   ```
5. edit apache config to serve our laravel app
   ![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-14%2002-40-28.png)
6. Edit Your laravel .env to point our proxysql database
7. Do some laravel setup
   ```
   php artisan migration:refresh
   php artisan db:seed
   php artisan storage:link
   ```
8. You can access your laravel app from 127.0.0.1:8080

## Laravel Application Detail
### Judul Aplikasi
**SimplePOS by adhityairvan**
### Deskripsi
Aplikasi ini adalah aplikasi sederhana pencatatan penjualan sebuah toko. Pemilik toko bisa menambah barang dagangan beserta stok nya. Pemilik toko juga bisa menambah pegawai dengan hak akses terbatas untuk menjadi kasir/ pencatat transaksi yang ada.
### Use Case
![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/UseCaseDiagram1.png)
### Screenshot Aplikasi
![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-14%2002-49-42.png)
![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-14%2002-50-14.png)
![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-14%2002-50-20.png)
![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/image/Screenshot%20from%202019-10-14%2002-50-23.png)

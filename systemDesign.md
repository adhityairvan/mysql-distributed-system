# System Design
## Design Diagram
![](https://github.com/adhityairvan/mysql-distributed-system/raw/master/DeploymentDiagram1.png)
## Specification
* IP Address Server
  * Mysql Server 1 : 10.1.15.143
  * Mysql Server 2 : 10.1.15.144
  * Mysql Server 3 : 10.1.15.145
  * Mysql Proxy & load Balancer : 10.1.15.146 & 10.10.15.146
  * Application Webserver : 10.10.15.147

* Mysql Server : 
  * RAM 512mb
  * Mysql Community server installed*
  * Connected to private networks
* ProxySql server : 
  * Ram 256mb
  * Installed proxysql
  * connected to private networks and public networks
* Application webserver
  * ram 512MB
  * Apache installed
  * PHP installed
  * connected to public networks

## Networks
* 10.1.15.1/24 (Private networks for mysql server communicating with each other)
* 10.10.15.1/24 (Public networks can be accessed from outside)

## Mysql client-server architecture 

### Overview
This project documents the implementation of a client–server architecture. MySQL Server is a good example of such an architecture, where clients connect to a central database server to perform queries and updates.

### Step1 create and configure two linux-based virtual servers (EC2 instance in AWS):
After creating servers on AWS , download keys to secure the connection. Create mysqlclient.pem and mysqlclient.pem and set 600 permissions.
![alt](images/0.png)
From windows terminal connect to the mysql server 
![alt](images/1.png)
After that update the meta data of the OS
![alt](images/2.png)

### Step2 install mysql server on the the first linux server
Install mysql-server package with
```bash
sudo apt install mysql-server
```
![alt](images/3.png)
Open /etc/mysql/mysql.conf.d/mysqld.conf and replace bind-address 127.0.0.1 by bind-address 0.0.0.0
![alt](images/4.png)
For the changes to take effect :
```bash
sudo systemctl enable mysql
sudo systemctl start mysql
```
![alt](images/5.png)
### Step2 Install mysql client on the second linux 
Connect to the second server using the mysqlclient.pem we have downloaded previously
![alt](images/7.png)
install mysql-client with
```bash
sudo apt install mysql-client
```
![alt](images/8.png)

### Step 3 configure inbound rule on the first server to allow access on port 
On the mysql server , add a new inbound rule and allow communication on port 3306
![alt](images/6.png)

### Step 4 Configure mysql server to allow remote connection
there is a binary named mysql_secure_installation that allow us to secure the mysql server installation. execute it with:
```bash
sudo mysql_secure_installation
```
![alt](images/9.png)
![alt](images/10.png)
to connect to mysql locally , type:
```bash
sudo mysql
```
![alt](images/11.png)
```mysql
Create user 'client'@'%' identified by "Pass"
create database test_db;
grant all on test_db.* to 'client'@'%' with grant option;
flush privileges;
```
![alt](images/12.png)
### Step 5 Connect to mysql server from client linux server
Connect to the remote mysql server by issuing :
```bash
mysql -u client -h 172.31030.57 -p
```
![alt](images/13.png)
![alt](images/14.png)
### Step 6 Perform sql queries
Let us manipulate CRUD sql instructions:
- 1 creation of a table test with
```sql
create table test (id int auto_increment,content varchar(255), primary key(id));
insert into test(content) values ("My first content");
insert into test(content) values ("My second content");
```
![alt](images/15.png)
To query content of a table:
```sql
select * from test;
```
![alt](images/16.png)
### Conculsion:
Through this lab , we have installed , configured and manipulated an  RDBMS system  (Relational DataBase Management System) that implement a client-server architecure. 
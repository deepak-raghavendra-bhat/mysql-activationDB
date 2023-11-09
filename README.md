# mysql-activationDB
activation DB

Build a Docker Image with MySQL Database


Let’s create a custom MySQL which contains your desired tables and data.

Creating SQL scripts
We will create SQL scripts which contain the SQL statements we want to execute on our database. Create a directory in which we will work. Also, create a subdirectory where we will store our .sql scripts.


$ mkdir -p ~/activa/sql-scripts
$ cd ~/activa/sql-scripts

Add a sample table called employees. The table needs to contain one row with an employee (first name, last name, department, and email).

Write a CreateTable.sql. This file contains the SQL statement to create a table called employees. We will add four columns to our table

CREATE TABLE employees (
first_name varchar(25),
last_name  varchar(25),
department varchar(15),
email  varchar(50)
);


Write a InsertData.sql. This file contains our statement to insert data in the table ‘employees’.

INSERT INTO employees (first_name, last_name, department, email) 
VALUES ('Test', 'Test', 'IT', 'test@hvt.com');

Structure should be as below:

└── sql-scripts
    ├── CreateTable.sql
    └── InsertData.sql

---------------------------------------------------------------------------------------------
Creating a Docker Image for Your Customized MySQL Database

Now that the scripts are ready, we can write a Dockerfile to create our own Docker image


$ cd ~/activa/
$ vi Dockerfile

Content of Dockerfile:

# Derived from official mysql image (our base image)
FROM mysql
# Add a database
ENV MYSQL_DATABASE activa
# Add the content of the sql-scripts/ directory to your image
# All scripts in docker-entrypoint-initdb.d/ are automatically
# executed during container startup
COPY ./sql-scripts/ /docker-entrypoint-initdb.d/

Create your Docker image:

$ cd ~/activa/
$ docker build -t activa .
[+] Building 0.4s (7/7) FINISHED                                                                         docker:default
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 362B                                                                               0.0s
 => [internal] load .dockerignore                                                                                  0.1s
 => => transferring context: 2B                                                                                    0.0s
 => [internal] load metadata for docker.io/library/mysql:latest                                                    0.0s
 => [internal] load build context                                                                                  0.1s
 => => transferring context: 380B                                                                                  0.0s
 => [1/2] FROM docker.io/library/mysql                                                                             0.2s
 => [2/2] COPY ./sql-scripts/ /docker-entrypoint-initdb.d/                                                         0.0s
 => exporting to image                                                                                             0.0s
 => => exporting layers                                                                                            0.0s
 => => writing image sha256:ca78ae15a2fb11403aa1589eb5836cc6e7a9867a33ea18ad9fc9a6f3ffc0ad21                       0.0s
 => => naming to docker.io/library/activa                                                                          0.0s

What's Next?
  View summary of image vulnerabilities and recommendations → docker scout quickview


And start your MySQL container from the image:

$ docker run -d -p 3306:3306 --name activa -e MYSQL_ROOT_PASSWORD=activa activa

Now we can verify. We will exec inside the container:

C:\---\codes\github\mysql-activationDB>docker run -d -p 3306:3306 --name activa -e MYSQL_ROOT_PASSWORD=activa activa
6ceba1cb2b27c52525560af1098574e6af9424debcd985b70814ddf90932848a

C:\---\codes\github\mysql-activationDB>docker exec -it activa bash
bash-4.4# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.1.0 MySQL Community Server - GPL

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| activa             |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

mysql> use activa
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+------------------+
| Tables_in_activa |
+------------------+
| employees        |
+------------------+
1 row in set (0.01 sec)

mysql> show columns from employees;
+------------+-------------+------+-----+---------+-------+
| Field      | Type        | Null | Key | Default | Extra |
+------------+-------------+------+-----+---------+-------+
| first_name | varchar(25) | YES  |     | NULL    |       |
| last_name  | varchar(25) | YES  |     | NULL    |       |
| department | varchar(15) | YES  |     | NULL    |       |
| email      | varchar(50) | YES  |     | NULL    |       |
+------------+-------------+------+-----+---------+-------+
4 rows in set (0.00 sec)

mysql> select * from employees;
+------------+-----------+------------+--------------+
| first_name | last_name | department | email        |
+------------+-----------+------------+--------------+
| Test       | Test      | IT         | test@hvt.com |
+------------+-----------+------------+--------------+
1 row in set (0.00 sec)

mysql> exit
Bye
bash-4.4# exit
exit

-------------------------------------------------

<<< If needed, perform this task >>>

Use Bind Mounts to Customize Your MySQL Database in Docker


$ docker run -d -p 3306:3306 --name activa -v ~/activa/sql-scripts:/docker-entrypoint-initdb.d/ -e MYSQL_ROOT_PASSWORD=activa -e MYSQL_DATABASE=activa mysql

verify again! Use the same steps as we did before: exec inside the container and check if the table and data exist!
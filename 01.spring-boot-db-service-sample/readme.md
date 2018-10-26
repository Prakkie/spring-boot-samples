## Using MySQL in Spring Boot via Spring Data JPA and Hibernate

See here for more informations:
http://blog.netgloo.com/2014/10/27/using-mysql-in-spring-boot-via-spring-data-jpa-and-hibernate/

### Build and run

#### Configurations

Open the `application.properties` file and set your own configurations.

#### Prerequisites

- Java 8
- Maven > 3.0

#### From terminal

Go on the project's root folder, then type:

    $ mvn spring-boot:run

#### From Eclipse (Spring Tool Suite)

Import as *Existing Maven Project* and run it as *Spring Boot App*.


### Usage

- Run the application and go on http://localhost:8080/
- Use the following urls to invoke controllers methods and see the interactions
  with the database:
    * `/create?email=[email]&name=[name]`: create a new user with an auto-generated id and email and name as passed values.
    * `/delete?id=[id]`: delete the user with the passed id.
    * `/get-by-email?email=[email]`: retrieve the id for the user with the passed email address.
    * `/update?id=[id]&email=[email]&name=[name]`: update the email and the name for the user indentified by the passed id.

### IMPORTANT FOR mySQL version >= v5.8

Use in pom
<dependency>
           <groupId>mysql</groupId>
           <artifactId>mysql-connector-java</artifactId>
           <version>8.0.11</version>
    </dependency>

If mySQL is >= 5.8

application.properties should have

spring.datasource.url = jdbc:mysql://localhost:3307/sampledb?autoReconnect=true&useSSL=false&allowPublicKeyRetrieval=true

### Instructions for docker

1.
docker run -d -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=sampledb -p 3307:3306 --name mysql_local mysql:latest

sampledb is a db name
—name is the container name
This will run docker container mysql_local on port 3307 of host machine (Mac)
mysql:latest will use image mysql of tag latest

2.
Prakashs-MacBook-Pro:~ prakashs$ docker ps -la
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
795b1534bd1b        mysql:latest        "docker-entrypoint.s…"   21 minutes ago      Up 20 minutes       33060/tcp, 0.0.0.0:3307->3306/tcp   mysql_local

This will list containers

3.
Prakashs-MacBook-Pro:~ prakashs$ docker exec -it mysql_local bash

To interact with container mysql_local

From mySQL v5.8, you will get issues with plugin because default plugin is  caching_sha2_password
ERROR 2059 (HY000): Authentication plugin 'caching_sha2_password' cannot be loaded: /usr/lib64/mysql/plugin/caching_sha2_password.so: cannot open shared object file: No such file or directory

Hence, if you try to reach mySQL in container from host as shown below, you will get error:

root@dse01{~}:mysql -h 192.168.99.1 -u root -P 3307 -p
Enter password:
ERROR 2059 (HY000): Authentication plugin 'caching_sha2_password' cannot be loaded: /usr/lib64/mysql/plugin/caching_sha2_password.so: cannot open shared object file: No such file or directory

To avoid this, follow these steps:
3.1.
Prakashs-MacBook-Pro:~ prakashs$ docker exec -it mysql_local bash

To interact with container mysql_local
root@795b1534bd1b:/# cd /etc/mysql/
root@795b1534bd1b:/# cp my.cnf my.cnf.orig

root@795b1534bd1b:/# cat ./my.cnf

Copy content of my.cnf locally and ADD this line preferably before symbolic-links=0:
default_authentication_plugin=mysql_native_password

Then have updated content in clip board

root@795b1534bd1b:/# cat > ./my.cnf
——paste here—-
Cntl+D


root@795b1534bd1b:/# mysql -u root -p
mysql>alter user root IDENTIFIED WITH mysql_native_password BY 'root';
mysql>exit

Restart docker container from host using:

Prakashs-MacBook-Pro:~ prakashs$ docker restart mysql_local


4.
If there is no docker-machine, then run

Prakashs-MacBook-Pro:~ prakashs$ docker-machine create default

As spring application properties ensure using this IP address from output of

Prakashs-MacBook-Pro:~ prakashs$ docker-machine ip default

>docker-machine ssh default
To ssh to docker-machine

Have a look at
docker-machine env default

To stop:
docker stop mysql_local
docker rm mysql_local
docker rmi is to remove images

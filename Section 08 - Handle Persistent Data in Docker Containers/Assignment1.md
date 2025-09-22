docker run -d -p 8081:80 --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -v mysql-db:/var/lib/mysql mysql:9

docker exec -it mysql mysql -uroot

create database testdb;

use testdb;

create table testtable(id INT PRIMARY KEY, name VARCHAR(50));

insert into testtable values (1, 'Noel');

select * from testtable;

exit

docker stop mysql

docker rm -f mysql

docker run -d --name mysql2 -p 8082:80 -e MYSQL_ALLOW_EMPTY_PASSWORD:yes -v mysql-db:/var/lib/mysql mysql:9

docker exec -it mysql2 mysql -uroot

show databases;

exit

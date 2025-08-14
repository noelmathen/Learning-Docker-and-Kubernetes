docker network create mynetwork


docker run --name mariadb_custom -d -e MARIADB_USER=user1 -e MARIADB_PASSWORD=123 -e MARIADB_ROOT_PASSWORD=123 -e MARIADB_DATABASE=mariadb1 --network mynetwork mariadb


docker run --name wordpress-container -e WORDPRESS_DB_HOST=mariadb_custom -e WORDPRESS_DB_NAME=mariadb1 -e WORDPRESS_DB_USER=user1 -e WORDPRESS_DB_PASSWORD=123 -p 8080:80 -d --network mynetwork wordpress


docker container rm -f mariadb_custom wordpress-container


mkdir ass2

docker run -d -p 8080:80 --name nginx --mount type=bind,source=/root/ass2,target=/usr/share/nginx/html  nginx

docker run -d -p 8081:80 --name nginx2 --mount type=bind,source=/root/ass2,target=/usr/share/nginx/html  nginx

Now when going to the page of both the containers, its default index.htmlpage is shown

cd ass2

vi index.html (and create a new index.html wiht new contents)

Now if we refresh the page of both the containers, the newly created index.html in the host machine is visible in the container in real time, wihtout any redeployment. This is because, the location in the container where the index.html was present was mounted to the host machine through bind mounting. thus changes made in hostt is visible in containers.

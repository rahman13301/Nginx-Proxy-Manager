# Nginx Proxy Manager to put SSL on everything:

One of the best projects I have spined up is Nginx Proxy Manager. My recommendation for those that are getting into home labs is to start with Nginx Proxy Manager as it provides a simple GUI to manage your Docker container SSL certificates for your containers and easily lets you provision Lets Encrypt certs. Having SSL certs on your containers with web interfaces just makes everything more secure and work better with other solutions that may not deal well with self-signed certificates.


Step 1: Create ec2 – server – connect – install docker – check status – start docker  

yum install -y docker

docker --version

systemctl status docker

systemctl start docker

Step 2: 

sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose

 sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

docker-compose version

step 3: create directory: ex - mkdir nginx

Step 4: create yaml file : ex - touch docker-compose.yml

Step 5: edit file and put below code : ex - vi docker-compose.yml

version: '3'

services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: npm
    ports:
      - "80:80"
      - "81:81"
      - "443:443"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    restart: always


Step 6: enter this cmd to run docker compose: ex -  docker-compose up -d

Step 7: To check active images enter cmd: ex-  docker images 

Step 8: This cmd checks the logs for particular container : ex -  docker-compose logs -f npm  

step 9:  
 - security group >> inbound >> all traffic >> anywhere 0.0.0.0
 - Access : http://<your-ec2-public-ip:80
 - <img width="978" height="366" alt="image" src="https://github.com/user-attachments/assets/ad22950a-3a56-4851-8b7c-b78c06314bcb" />

 - Access : http://<your-ec2-public-ip:81
 - <img width="918" height="409" alt="image" src="https://github.com/user-attachments/assets/3f2f7d65-4806-44ff-9575-a969edf09b29" />

 Project is successfully completed.










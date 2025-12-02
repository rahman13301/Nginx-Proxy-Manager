# Nginx Proxy Manager to put SSL on everything by using docker-compose:

One of the best projects I have spined up is Nginx Proxy Manager. My recommendation for those that are getting into home labs is to start with Nginx Proxy Manager as it provides a simple GUI to manage your Docker container SSL certificates for your containers and easily lets you provision Lets Encrypt certs. Having SSL certs on your containers with web interfaces just makes everything more secure and work better with other solutions that may not deal well with self-signed certificates.


## Step 1: Create ec2 – server – connect – install docker – check status – start docker  
```
yum install -y docker
docker --version
systemctl status docker
systemctl start docker
```
## Step 2: Import docker compse latest releases from github, change mode.
```
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose version
```
## step 3: create directory, yaml file and put below dependencies into file.
```
mkdir nginx
touch docker-compose.yml
vi docker-compose.yml
```

- Put below dependencies into file (docker-compose.yml).
```
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
```


## Step 4: cmd to run docker compose, check docker images and logs.
```
docker-compose up -d
docker images
docker-compose logs -f npm
```

## Step 5: Change inbound rules and make port accessible.
 - security group >> inbound >> all traffic >> anywhere 0.0.0.0
 - Access : http://<your-ec2-public-ip:80
 - <img width="978" height="366" alt="image" src="https://github.com/user-attachments/assets/ad22950a-3a56-4851-8b7c-b78c06314bcb" />

 - Access : http://<your-ec2-public-ip:81
 - <img width="918" height="409" alt="image" src="https://github.com/user-attachments/assets/3f2f7d65-4806-44ff-9575-a969edf09b29" />

 Project is successfully completed deploying Ngnix manager with docker compose.


# Also created Jenkins Pipeline for docker-compose setup:
- **Jenkinsfile (to be placed in your repository root and put groovy pipeline script)**
- **Jenkins pipeline script for building and deploying the Nginx Proxy Manager using Docker Compose from the GitHub repository**

 <img width="1633" height="606" alt="image" src="https://github.com/user-attachments/assets/640675fd-2bad-4c35-b5ef-81bbcd466c7e" />

 <img width="938" height="825" alt="image" src="https://github.com/user-attachments/assets/0049a19a-fb41-493c-8fd3-6b70bd1e7a30" />

 - **Here is the output, we have to access with localhost:80 & localhost:81**

 <img width="1253" height="848" alt="image" src="https://github.com/user-attachments/assets/06bedb76-c35b-4a14-b99d-54a8ecf42fb6" />

 <img width="1435" height="496" alt="image" src="https://github.com/user-attachments/assets/2e231e4e-abea-4196-90c6-44647d50e6fc" />

 Pipeline has been build successfully.





 


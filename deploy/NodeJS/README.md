## Create DigitalOcean server

1. Choose an image;
2. Choose a plan;
3. Choose a datacenter region;

   3.1 New york

   3.2 San Francisco

4. Authentication;

   SSH keys method:

   1. Run ssh-keygen
   2. /Users/USER/.ssh/id_rsa
   3. cd ~/.ssh
   4. cat id_rsa.pub
   5. Copy and paste
   6. Give a name

5. Choose a hostname
6. Create.

## Config server

Copy ip address from droplet

Go to terminal and paste:

`ssh root@ipaddress`

LogIn

Run:

`apt update`

`apt upgrade`

Create a new user:

`adduser [deploy]`

Give sudo permissions:

`usermod -aG sudo deploy`

Allow ssh connection via other user:

`cd /home/deploy`

`mkdir .ssh`

`cd .ssh/`

`cp ~/.ssh/authorized_keys .`

Change file owner:

`chown deploy:deploy authorized_keys`

Exit and test connection

##### > Install Node

## Clone application

Copy repository url

`git clone [url] [foldername]`

`cd foldername`

`npm install`

Config environment variables

`cp .env.example .env`

Edit .env file

`nano .env`

Alter NODE_ENV=production

Alter APP_SECRET= \***\*\*\*\***

Copy IP and change APP_URL

## Config Database

Config Docker

`sudo groupadd docker`

`sudo usermod -aG docker $USER`

Logout

Config postgres:

```
docker run --name postgres -e POSTGRES_PASSWORD=bootcampdeploy -p 5432:5432 -d -t postgres
```

Config mongo:

```
docker run --name mongo -p 27017:27017 -d -t mongo
```

Config redis:

```
docker run --name redis -p 6379:6379 -d -t redis:alpine
```

Config .env:

```
# Database

DB_HOST=localhost
DB_USER=postgres
DB_PASS=bootcampdeploy
DB_NAME=bootcampnodejs

# Mongo

MONGO_URL=mongodb://localhost:27017/bootcampnodejs

# Redis

REDIS_HOST=127.0.0.1
REDIS_POST=6379
```

## Create postgres database:

`docker exec -i -t postgres /bin/sh`

Change user:

`su postgres`

`psql`

Create database:

```sql
CREATE DATABASE bootcampnodejs
```

`\q` to exit

## Running server

Add scripts:

```json
"scripts": {
     ...
    "build": "sucrase ./src -d ./dist --transforms imports",
    "start": "node dist/server.js"
  },
```

git pull on server

## Run migrations

`npx sequelize db:migrate`

`npm run build`

`npm run start`

Use insomina to test change url with droplet ip

If firewall is already config this will block access, to allow
external access:

`sudo ufw allow 3333`

## Dicas SSH

If you need to stop your server:

`ssh deploy@ipaddress`

`lsof -i :3333`

`kill -9 PID`

#### Avoid losing connection:

`sudo nano /etc/ssh/sshd_config`

Add:

```
ClientAliveInterval 30
TCPKeepAlive yes
ClientAliveCountMax 99999
```

Restart ssh service:

`sudo service sshd restart`

## Port redirect

#### Config NGINX

Install NGINX:

`sudo apt install nginx`

Allow port 80:

`sudo ufw allow 80`

Changing default port access:

`sudo nano /etc/nginx/sites-available/default`

Alter file:

```
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        server_name _;

        location / {
                proxy_pass http://localhost:3333;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }

}
```

Restart service:

`sudo service nginx restart`

Test if everthing is ok:

`sudo nginx -t`

Now you can discard :3333 suffix

## Keep application running

Install PM2:

`sudo npm install -g pm2`

Start:

`pm2 start dist/server.js`

List process:

`pm2 list`

Monitoring:

`pm2 monit`

#### Keep server running after a restart

`pm2 startup systemd`

Copy PATH command and paste on command line

## Continuos integration

#### Using Buddy works

1. Create project
2. Create pipeline

   2.1 Name: Build & Deploy

   2.2 Trigger mode: on push

3. Add a new pipeline

4. Choose DigitalOcean

5. Change Login to deploy

6. Authentication mode: Buddy`s SSH key

7. Go to server and run generated code on deploy user

8. Remote path (application folder on server)

9. Add this action

10. Click in "Execute SSH commands"

11. Add following commands:

```
npm install
npm run build
npx sequelize db:migrate
pm2 restart server
```

12. Paste droplet ip in Hostname & Port

13. Login: deploy

14. Authentication mode: Buddy`s ssh key

15. Working directory: (application folder on server)

16. Add this action

17. Add Notification Action:

    17.1 Add email body

    17.2 Add email in recipients

18. just push code

# restore-api-server
just documenting how to restore my server

### Update System
```bash
apt update
apt upgrade
```

### Install the Database Framework
#### MySQL 
```bash
wget -qO - https://dev.mysql.com/doc/refman/8.0/en/checking-gpg-signature.html | grep -oP 'gpg_key=\K[^\"]+' | xargs -I {} sudo apt-key adv --fetch-keys {}
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys B7B3B788A8D3785C
```
```bash
wget https://dev.mysql.com/get/mysql-apt-config_0.8.22-1_all.deb
sudo dpkg -i mysql-apt-config_0.8.22-1_all.deb
sudo apt update
sudo apt install mysql-server
mysql_secure_installation
```
```mysql
mysql -u root -p*********
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'miawww';
```
```bash
systemctl start mysql
systemctl enable mysql
```

#### MongoDB
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/debian $(lsb_release -cs)/mongodb-org/6.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
```

### Installing the Server Dependencies
#### Node
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

source ~/.bashrc

nvm install 20
```

#### Puppetteer Dependencies
```bash
apt-get update && apt-get install -y \
    chromium \
    --no-install-recommends \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

sudo apt-get install fonts-liberation2


sudo apt-get install -y gconf-service libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 \
  libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 \
  libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 \
  libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 \
  libxrender1 libxss1 libxtst6 ca-certificates fonts-liberation libappindicator1 libnss3 \
  lsb-release xdg-utils wget

sudo apt-get update
sudo apt-get install -y libgbm-dev
```

#### Nginx
```bash
apt install nginx -y
systemctl start nginx
systemctl enable nginx
nginx -t
```

#### SSL Cert
```bash
sudo apt install certbot python3-certbot-nginx
```

#### WinSCP / CLI SCP
```bash
mkdir alooow
cd alooow
scp root@ip:/root/dir /root/alooow
```

### Restore
```bash
tar -xzvf VPS-SERVER-BACKUP-[DATE].tar.tar.gz
# ls
# mysql_all_databases.sql www/www/* nginx-sites-available otakudesu letsencrypt mongodb/anime_db
```
```bash
cp -r otakudesu ../
cp -r letsencrypt /etc/
cp -r www/www/* /var/www/
cp -r nginx-sites-available/* /etc/nginx/sites-available/
```

### Databases Restore
###### MongoDB

```bash
mongorestore --db anime_db ./mongodb/anime_db
```
###### MySQL
```bash
mysql -u root -p < mysql_all_databases.sql
```

##### Install PM2
```bash
npm i pm2 -g
```

##### Modules & Dependencies Fixing
```
cd /var/www/api/backend
npx puppeteer browsers install chrome
cd /var/www/staging-api/gemini-bot-wa
npm i
npm i express
```

##### Starting the Server

###### API SERVER
```
cd /var/www/api/backend
pm2 start server.js
```

###### STAGING SERVER
```
cd /var/www/staging-api
pm2 start alo.js
```

###### WHATSAPP API SERVER
```
pm2 start connect.js
```

###### PAYMENT SERVER
```
cd /var/www/pay
node connect.js
```

###### ANIMESEACRH SERVER
```
/var/www/animesearch/backend-chiw
pm2 start server.js
```

###### TRANSLATER SERVER
```
apt install git
cd
git clone https://github.com/Songkeys/translateer.git
cd translateer
npm install
npm run build
pm2 start npm --name "translateer" -- run start
```

###### DOCKER SERVER
```
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

docker pull hlohaus789/g4f
docker run -d \
  -p 8080:8080 -p 1337:1337 -p 7900:7900 \
  --shm-size="2g" \
  -v ${PWD}/har_and_cookies:/app/har_and_cookies \
  -v ${PWD}/generated_images:/app/generated_images \
  hlohaus789/g4f:latest
```

###### ENSURE THERE'SNT AN ERROR
```
pm2 logs
```

### Starting the WebServer
```
ln -s /etc/nginx/sites-available/* /etc/nginx/sites-enabled/
systemctl restart nginx
nginx -t
```

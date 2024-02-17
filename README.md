# vps_deploy_commandCI/CD

Create ssh key to connect vps through ssh key

```
ssh-keygen -t rsa -b 4096 -C "your email address"  // it will generate a key

cat ~/.ssh/id_rsa.pub // it will show your key
```

for windows user, you have to install git bash to run this command

go to the folder where you've saved your key and run cmd there 


```
ssh -i [your_ssh_key_name] root@your_ip_address // it will connect your vps

```

After successfully connect to your vps, you have to update necessary packages and install nginx. so let's do it.

```
sudo apt update 
sudo apt upgrade
``` 

after that install nginx

```
apt install nginx

```

to deploy your project with github actions, you have to create a user and give permission to that user to create folder and write file

```

sudo adduser [your_user_name]

```

it will ask you to give the password and some other information, after that you have to give sudo permission to that user

```
 sudo usermod -aG sudo [your_user_name]

```

after that you have to give permission to that user to create folder and write file in your vps directory 

```
sudo chmod -R 777 [your_directory_path] 

```

for me it's [your_directory_path] is /var/www, 

this sudo chmod -R 777 is the command to provide permission to write or create folder as user, inside square bracket you need to put directory path,after that you have to switch to that user

```
su - [your_user_name]

```
now you will have to create a folder where you will do your necessary work, for me i'll work as name streaming, you need to navigate to your directory where you want to create folder, for me it's /var/www, and this directory only available after nginx install.

``` 
cd /var/www 

mkdir [your directory name]

cd  [your directory name]

```

As we are going to use github actions to deoploy our project automatically, so we need to create a repository in github and push our project there. i've already created a repository name streaming. go to your repository setting and find action, you will see a option call runner, click that and you will see a option call add runner, click that and after that click self-hosted runner, here you will see differen os option, choose linux and follow the instruction.
 
At very last step you need to install some packgae to make your runner active all the time. if you run ./run.sh, it will stay active for momentary but if exit it will be in offline. So

here we have install a packgae to make it active all the time

```
sudo ./svc.sh install 
    
```
after that run it 

```
sudo ./svc.sh start

```

now if you check back to your runner, you'll see it's activated but idle. now we need to install some package to run our project in vps, install nodejs lts version 

```
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash - &&\
sudo apt-get install -y nodejsZZZ

```

it will install both node and npm to check

```
node -v
npm -v
```

now we need to install pm2 to run our project in background

```
npm install -g pm2@latest
```


you might get into an error while download this package as user, so you could you switch to root user by run exit command, then install it. after that let's configure nginx. you can do these as root user or as user, i'll do it as root user. so let's go to root cd /root as root user

```
cd /etc/nginx/sites-available/
```

```
ls
```

you'll see default file now let's edit this default file

 location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                proxy_pass http://localhost:5000;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }

proxy_pass http://localhost:5000;: This command tells Nginx to forward requests it receives to http://localhost:5000, which is the address of the application you're running. Replace 5000 with the port number your application is running on.

proxy_http_version 1.1;: This command sets the version of the HTTP protocol that will be used for the proxy request. In this case, it's HTTP/1.1.

proxy_set_header Upgrade $http_upgrade;: This command sets the Upgrade HTTP header to the value of the $http_upgrade variable. This is used for handling WebSocket connections.

proxy_set_header Connection 'upgrade';: This command sets the Connection HTTP header to the value 'upgrade'. This is also used for handling WebSocket connections.

proxy_set_header Host $host;: This command sets the Host HTTP header to the value of the $host variable. This is used to pass the original host header to the proxied server.

proxy_cache_bypass $http_upgrade;: This command tells Nginx to bypass the cache when the $http_upgrade variable is set. This is useful for WebSocket connections, which should not be cached.

After you've replaced localhost:5000 with your port number, you need to save the configuration and restart Nginx for the changes to take effect. You can check your port number by running the pm2 list command, which will show you your project name and port number.

```
systemctl restart nignx
```

and then check nginx status to check nginx running successfully

```
nginx -t
```


## Domain Configuration

 If you want add any domain all you have to do is put your registered domain name as server_name in default file. so let's  go again to default file and add our domain name

```
cd /etc/nginx/sites-available/
nano default 
```

and add your domain name as server_name and save it. 

```
        server_name api.reely.tech;

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                proxy_pass http://localhost:5000;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }

```

## Firewall Configuration

it's very important to configure firewall to protect your vps from any kind of attack. Here we are going to use ufw to configure basic firewall setting.


```
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

## SSL Certification

To have https connection we need to install ssl certification. we are going to use Let's Encrypt to install ssl certification.

```
apt install certbot python3-certbot-nginx
```

Make sure that Nginx Full rule is available

```
ufw status
```

```
certbot --nginx -d example.com -d www.example.com
```

Let’s Encrypt’s certificates are only valid for ninety days. To set a timer to validate automatically:

```
systemctl status certbot.timer
```


there is one single thing to set, basically nginx allow only few mb file to upload so if you want allow more than you have go to

```
sudo nano /etc/nginx/nginx.conf
```
and paste down there and save


```
client_max_body_size 100M;
```


reload nginx

```
sudo systemctl reload nginx
```

## Github Actions

Now we are going to use github actions to deploy our project automatically. so let's go to our github repository and create a folder name .github and inside that folder create a folder name workflows and inside that folder create a file name main.yml. so the path will be .github/workflows/main.yml. now let's copy and paste this code in main.yml file

```
name: Node.js CI
 
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs: 
        build:
        runs-on: self-hosted
        strategy:
        matrix:
                node-version: [14.x]
        steps:
        - uses: actions/checkout@v2
        - name: Use Node.js ${{ matrix.node-version }}
                uses: actions/setup-node@v1
                with:
                node-version: ${{ matrix.node-version }}
        - name: npm install, build, and test
                run: |
                npm install
                npm run build --if-present
                npm run test --if-present
        - name: Deploy to VPS
                pm2 start npm --name "streaming" -- start
        ```

configure githbu actions according to your need. This one is for very 

# WordPress site as my first experience in web DevOps

My first step was download docker and docker-compose

<h2>Requirements: </h2>
Docker 20.10.7 (as last stable)
docker-compose version 1.21.0, build unknown
docker-py version: 3.4.1
CPython version: 3.7.3
wordpress:5.1.1 (container)
mysql:8.0 (container)
nginx:1.15.12-alpine (container)

<h2>Setting up the environment </h2>
First is docker : instruction is https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04

Second is docker-compose is https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-debian-10-ru
Created .env and .gitignore (i didnt push .env in .gitingore)

<h2>Install Wordpress</h2>
<h3>First is - create nginx-conf/nginx.conf in folder in project directory.</h3>
Use config first is for http and change example.com on your domain

    server {
        listen 80;
        listen [::]:80;

        server_name example.com www.example.com;

        index index.php index.html index.htm;

        root /var/www/html;

        location ~ /.well-known/acme-challenge {
                allow all;
                root /var/www/html;
        }

        location / {
                try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass wordpress:9000;
                fastcgi_index index.php;
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_param PATH_INFO $fastcgi_path_info;
        }

        location ~ /\.ht {
                deny all;
        }

        location = /favicon.ico {
                log_not_found off; access_log off;
        }
        location = /robots.txt {
                log_not_found off; access_log off; allow all;
        }
        location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
                expires max;
                log_not_found off;
        }
    }
<h3> Second part is creating docker-compose.yaml. </h3>
  
U need to change example.com on ur domain for correct SSL.

  <pre>
version: '3'

  services:
  db:
    image: mysql:8.0
    container_name: db
    restart: unless-stopped
    env_file: .env
    environment:
      - MYSQL_DATABASE=wordpress
    volumes:
      - dbdata:/var/lib/mysql
    command: '--default-authentication-plugin=mysql_native_password'
    networks:
      - app-network

  wordpress:
    depends_on:
      - db
    image: wordpress:5.1.1-fpm-alpine
    container_name: wordpress
    restart: unless-stopped
    env_file: .env
    environment:
      - WORDPRESS_DB_HOST=db:3306
      - WORDPRESS_DB_USER=$MYSQL_USER
      - WORDPRESS_DB_PASSWORD=$MYSQL_PASSWORD
      - WORDPRESS_DB_NAME=wordpress
    volumes:
      - wordpress:/var/www/html
    networks:
      - app-network

  webserver:
    depends_on:
      - wordpress
    image: nginx:1.15.12-alpine
    container_name: webserver
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - wordpress:/var/www/html
      - ./nginx-conf:/etc/nginx/conf.d
      - certbot-etc:/etc/letsencrypt
    networks:
      - app-network

  certbot:
    depends_on:
      - webserver
    image: certbot/certbot
    container_name: certbot
    volumes:
      - certbot-etc:/etc/letsencrypt
      - wordpress:/var/www/html
    command: certonly --webroot --webroot-path=/var/www/html --email sammy@example.com --agree-tos --no-eff-email --staging -d example.com -d www.example.com

volumes:
  certbot-etc:
  wordpress:
  dbdata:

networks:
  app-network:
    driver: bridge
</code></pre>
Then <pre>docker-compose up -d </pre>
U can check u certs using command
<pre>docker-compose exec webserver ls -la /etc/letsencrypt/live</pre>
U ll see the folder with ur site.
If all ok u need to change in docker-compose.yaml flag <pre>--staging </pre> to <pre>--force-renewal</pre> in certbot part for getting the same domain at last cert.


<h3> Third part is making HTTPS connection. </h3>
At first we need to get security parameters of nginx
<pre>curl -sSLo nginx-conf/options-ssl-nginx.conf https://raw.githubusercontent.com/certbot/certbot/master/certbot-nginx/certbot_nginx/_internal/tls_configs/options-ssl-nginx.conf </pre>

After u need to change ur nginx-conf/nginx.conf file to
<pre>server {
        listen 80;
        listen [::]:80;

        server_name example.com www.example.com;

        location ~ /.well-known/acme-challenge {
                allow all;
                root /var/www/html;
        }

        location / {
                rewrite ^ https://$host$request_uri? permanent;
        }
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name example.com www.example.com;

        index index.php index.html index.htm;

        root /var/www/html;

        server_tokens off;

        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

        include /etc/nginx/conf.d/options-ssl-nginx.conf;

        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header Referrer-Policy "no-referrer-when-downgrade" always;
        add_header Content-Security-Policy "default-src * data: 'unsafe-eval' 'unsafe-inline'" always;
        # add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
        # enable strict transport security only if you understand the implications

        location / {
                try_files $uri $uri/ /index.php$is_args$args;
        }

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass wordpress:9000;
                fastcgi_index index.php;
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_param PATH_INFO $fastcgi_path_info;
        }

        location ~ /\.ht {
                deny all;
        }

        location = /favicon.ico {
                log_not_found off; access_log off;
        }
        location = /robots.txt {
                log_not_found off; access_log off; allow all;
        }
        location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
                expires max;
                log_not_found off;
        }
}
</pre>
And in docker-copmpose.yaml need to add 443 port
<pre>  ports:
      - "80:80"
      - "443:443"
</pre>
Reload and check installing of Wordpress.

<h3> Reload SSL certs </h3>

Create new file ssl_renew.bash in project directory :
<pre>#!/bin/bash

COMPOSE="/usr/bin/docker-compose --no-ansi"
DOCKER="/usr/bin/docker"

cd /WordPress/
$COMPOSE run certbot renew && $COMPOSE kill -s SIGHUP webserver
$DOCKER system prune -af
</pre>
U need to add it to crontab. At first for test can be 5 min:
<pre> */5 * * * * /WordPress/ssl_renew.sh >> /var/log/cron.log 2>&1 </pre>
U can chek logs in /var/log/cron.log
If all is ok u can make it every day and delete key --dry-run

<h2> Mail and domain after install from admin panel </h2>
<h3>Reg.ru and Domain altezza.space </h3>
In DNS need to make A record @ and ip where @ - domain, ip - ip of server

<h3> Mail </h3>
Wordpress can use plugins for many actions. I have used WP Mail SMTP - with the most stars. It let to choose what type of SMTP u can use.
I take biz.mail.ru
<ul>Its easy to make mail with domain:
  <li> create user ***@domain </li>
  <li> make TXT record on DNS that u are owner </li>
  <li> make MX record because it need </li>
  <li> make TXT record for DKIM or u can up it on reg.ru </li>
  <li> all instructions are on biz.mail.ru https://biz.mail.ru/domains/altezza34.space/help/ </li>
</ul>
So, after i had some trouble in wordpress control panel, but mail was ok.






<h1> Another story about creating</h1>
I used guid for
<ul>
  <li>docker: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04</li>
  <li>docker-compose: https://www.digitalocean.com/community/tutorials/how-to-install-docker-compose-on-debian-10-ru</li>
  </ul>
  After i tryed to make container 'Hello world' - OK go next step.
  
 For creating site on WordPress i userd guid: https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-docker-compose-ru
 
 At first, i created my environment. Installed Git, Docker, Docker-compose.
 
 Second time i created my domain name on reg.ru "altezza34.space". So, i get access to DNS and made associate ip and domain name.
 
 Third time i created nginx.conf file and going instruction to create 4 containers "wordpress", "webserver", "certbot" and "db" in docker-compose.yaml.
 I had some troubles with domain name because i tried to use domain from wordpress le1ns13ru.wordpress.com but i couldnt unite it (my local machine and wordpress domain).
 Ater many tries i tryed to use my own domain. 1 Hour and it has worked already! It was http site but ok. 
 I installed wordpress and started to explore it. It was my first experience. So, it s better then joomla which i admined.
 
 Four step was HTTPS. Going to instruction i rebuild nginx.conf and docker-compose.yaml (ports from local to container).
 Op op and it works! ok! Im happy! But i needed to reboot all containers not only webserver. It was strange but ok.
 
 Five step was MAIL. I started to study ways hoow i can do it. I have used plugin "WP MAIL SMTP". Easy way to sent mails from ****@mail.ru or ****@yandex.ru but i wanted to use    mail with my domain. I tryed to use SMTP.COM, MailGUN but it cost some money. After ill try to use biz.mail.ru (russian mail.ru). It let to create domain mail free! 
 I added new user noreply@altezza34.space and added few records (MX) to DNS on reg.ru (for domain verification and forwarding mail) and DKIM.
 Few tests and ok it works!
 
 Six step was testing ssl certs. In instruction i see one script for reactivate SSL and puting it to crontab. So this cert u can see in my repository (ssl_renew.sh)
 
 Between second and third step i tryed to study git actions. As i understand - if ok it should push it to master branch (as it should work and named pull request). But a did everything in master branch and created action to build all containers.
 
 Im very sad that i didnt do it with ansible. I looked few videos about it. It means that i can made the same server like i did by remote commands. As i understand i need to pull main comands in playbook for up all containers and it likes pattern for up servers. I had an idea to make it but i i hadn't,sorry.
 Thank you DeLaWeb for this task and im waiting your comments.
 
 
 

 
 
 

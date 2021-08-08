# WordPress site as my first experience in web DevOps

My first step was download docker and docker-compose

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
 
 Five step was MAIL. I started to study ways hoow i can do it. I have used plugin for mail smtp. Easy way to sent mails from ****@mail.ru or ****@yandex.ru but i wanted to use    mail with my domain. I tryed to use SMTP.COM, MailGUN but it cost some money. After ill try to use biz.mail.ru (russian mail.ru). It let to create domain mail free! 
 I added new user noreply@altezza34.space and added few records (MX) to DNS on reg.ru (for domain verification and forwarding mail) and DKIM.
 Few tests and ok it works!
 
 Six step was testing ssl certs. In instruction i see one script for reactivate SSL and puting it to crontab. So this cert u can see in my repository (ssl_renew.sh)
 
 Between second and third step i tryed to study git actions. As i understand - if ok it should push it to master branch (as it should work and named pull request). But a did everything in master branch and created action to build all containers.
 
 Im very sad that i didnt do it with ansible. I looked few videos about it. It means that i can made the same server like i did by remote commands. As i understand i need to pull main comands in playbook for up all containers and it likes pattern for up servers. I had an idea to make it but i i hadn't,sorry.
 Thank you DeLaWeb for this task and im waiting your comments.
 
 
 

 
 
 

Let’s Encrypt is a free, automated, and open SSL/TLS Certificate authority (CA) that allows you to acquire Free SSL/TLS Certificates for your web server provided your verify your Domain. In this tutorial I will describe how you can download and install a Free Let’s Encrypt SSL/TLS Certificate on Linux for Nginx

Pr-requisites

An Nginx  Web Server running on Linux and  serving a Flask/Python based Web Application or a Wordpress Blog

For running Flask applications on Nginx see Configure Nginx and Gunicorn to serve flask

For running Wordpress Blogs using Nginx see How to install the LEMP Stack for Wordpress

A small overview of  How Let's Encrypt Works

In order to generate a SSL/TLS certificate for your Website from Let’s Encrypt you will need to download Certbot  a certificate management agent and issue a request  to Let's Encrypt's CA server. The CA server will then verify your  Websites Domain by asking the agent to either provision a DNS record under your domain name or provision a file under a specified path on your Domain.

Once your Domain has been verified Let's Encrypt will issue your website a SSL Certificate. Generally the Let's Encrypt client has a web server plugin which will automatically enable the web server to serve your website via HTTPS using the new certificate, but the Nginx plugin is experimental and hence in this tutorial we will configure it manually.

Step 1: Download  Let’s Encrypt's certificate management agent CertBot

wget https://dl.eff.org/certbot-auto
chmod a+x ./certbot-auto

Step 2: Modify Nginx to Display Files From Hidden Folders
The HTTP URI that let's encrypt will ask the agent to generate to verify your domain will contain a hidden folder. Nginx does not display files from hidden folders and you may get a “urn:acme:error:unauthorized  404" error causing your Domain Verification to fail. In order to avoid this you need  to add the below directive inside your server block.

vim /etc/nginx/sites-enabled/yourdomain.com.conf
location ^~ /.well-known/acme-challenge/ {
        allow all;
       }
sudo nginx -t && sudo service nginx reload

Step 3: Run CertBot

Currently the Nginx plugin for certbot is very experimental hence you will need to use the "cert-auto" command with the "certonly option. This will generate the SSL certificate whose location you will need to manually configure.

./certbot-auto certonly --agree-tos --webroot -w /webroot/yourdomain.com/ -d yourdomain.com -d www.yourdomain.com

You will need to provide an email address for lost key recovery and notices purposes.

Step 4:  Add the SSL Certificate location to Nginx

Add the certificate and key inside the server block of your domain

Vim /etc/nginx/sites-enabled/yourdomain.com.conf

server{
 listen 443 ssl default_server;
 server_name yourdomain.com www.yourdomain.com;
 ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
 ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
 ...
}
Now redirect the traffic from HTTP to HTTPs by adding another server block outside the earlier one


server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;
}


 sudo nginx -t && sudo service nginx  reload

If you resolve your website, it should automatically redirect to HTTPS.

You can also verify your certificate at https://www.ssllabs.com/ssltest/analyze.html?d=yourdomain.com

The complete Nginx SSL configuration file is at available at the DevOps Wiki

Step 5: Setup Automatic Let's Encrypt SSL certificate renewals
The Goal of Let’s Encrypt is to make it possible to automatically issue SSL certificates without human intervention.
Once the SSL certificate is issued renewal can be done automatically by adding the following commands to your cron job.

sudo crontab -e



0 0 * * 2  bash /home/user/certbot-auto renew &gt; /var/log/certbot.log
15 0 * * 2 /etc/init.d/nginx reload
The above cron job will run every Tuesday at 12am, The Certbot agent will first check if the certificate is about to expire and then place a request for renewal. You can renew the the certificate if it is less than 30 days away from expiration. Post the renewal attempt the cron will reload Nginx. Any errors will be logged to "/var/log/certbot.log"

If you need to configure your Firewall to open Port 443 then read my earlier tutorial on Config Server Firewall

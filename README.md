# Easily free SSL with Nginx and Rails (via Passanger)
## Works for Debian, Ubuntu, Red Had and CentOS

`Note: For this, you need superuser permission, so make sure your are loged in with root or add 'sudo' before the commands, without quotes, obviously.`

`Oh, and don't copy the '#', please.`

### 1. Install dependencies
 The first thing we need to do is update and install the dependencies, using apt-get or yum.
 
 **Debian/Ubuntu**
 ```bash
    # apt-get update
    # apt-get -y install git bc
 ```
 **Red Hat/CentOS**
 ```bash
    # yum update
    # apt-get -y install git bc
 ```
 
### 2. Clone the repository
  Now we are going to download the latest version of the Let's Encrypt repository:
  Here we`ll save in /opt, but feel free to change the directory. Just make sure to use this directory in the next steps. 
 ```bash
    # git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt
 ```
 
### 3. Generate the certificate
  Before start, let's setup our server config.
  Go to the server block in your nginx configuration and add this, replacing yourHome to your current user:
  ```
   location /.well-known/acme-challenge {
    root /home/yourHome/letsencrypt;
   }
  ```
  Now, let's create the directory, if not exists.
  ```bash
   cd /home/yourHome
   mkdir letsencrypt
   mkdir letsencrypt/.well-known
   mkdir letsencrypt/.well-known/acme-challenge
  ```
  
  Ok, everything is set up to we generate our certificate, so, let's do this!
  For this example, we'll use domain.com as our domain.
  
  If is your first time, or from a long time, this should update, so, it will take a while.
  
  `Replace the 'yourHome' for your current user`
  ```bash
    # cd /opt/letsencrypt
    # ./letsencrypt-auto certonly -a webroot --webroot-path=/home/letsencrypt -d domain.com
  ```
  `If you need more than one domain, just add '-d domain2.com' in the end, as you want.`
  
### 4. Add the certificate to Nginx server
   Now we need to make sure Nginx has the right config, so, let`s add this to your server block.
   
   Just remember to replace domain.com for your first domain when you generated the certificates.
   ```
    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/docmain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/ssl/certs/dhparam.pem;
    ssl_ciphers 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_stapling on;
    ssl_stapling_verify on;
    add_header Strict-Transport-Security max-age=15768000;
   ```
   
### 5. Restart Nginx
   We are ready to recieve https connections, just restart ou reload your Nginx.
   
   **Restart**
   ```bash
    # /etc/init.d/nginx reload
   ``` 
   or
   
   **Reload**
   ```bash
    # /etc/init.d/nginx reload
   ```
   
### 6. Force HTTPS connections
   Now, let's force use https instead http, for this, add the following to your `application.rb`, but only for production, we don`t need this in localhost.
   
   ```ruby
    if Rails.env.production?
        config.force_ssl = true
    end
   ```
   
### 7. Renew your certificates
   The certificates aren`t forever, to we need to renew them. But we can make this automatic, let's just creating an crontab for this (user must be sudo).
   ```bash
    # crontab -e
   ```
   Then, add this line:
   ```
    1 1 1 * * /opt/letsencrypt/letsencrypt-auto renew -a webroot --webroot-path=/home/yourHome/letsencrypt
   ```
   
## Finished!
   Your website now have SSL, now go have fun with your new security.
   

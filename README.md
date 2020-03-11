# Project Overview

You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

## prerequisites
To complete this project, you'll need a Linux server instance. We recommend using [Amazon Lightsail](https://lightsail.aws.amazon.com/) for this. If you don't already have an Amazon Web Services account, you'll need to set one up. Once you've done that, here are the steps to complete this project.

## Access
 IP address : 54.160.176.9
 Grader user : grader
 Grader password : 1234
 Port : 2200
 Project url : http://54.160.176.9.xip.io
 

### 1. Start a new Ubuntu Linux server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com/)

1.1. Log in / sign up
1.2. Create an instance on the home page
1.3. Choose an instance image: Ubuntu, choose "OS Only" then choose Ubuntu as the operating system.
1.4. Choose your instance plan, For this project, the lowest is just fine. 
1.5. Give your instance a hostname
1.6 done creating 
....
Wait for the instance to start up.

### 2. SSH into the server
2.1 From the `Account` menu on Amazon Lightsail, click on `SSH keys` tab and download the Default Private Key.

2.2 Move this private key file named `LightsailDefaultPrivateKey-*.pem` into the local folder `~/.ssh` and rename it `lightsail_key.rsa`.

2.3 To connect to the instance via the terminal: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@54.160.176.9`, where `54.160.176.9` is the public IP address of the instance.


**Update and upgrade installed packages**
```
sudo apt-get update
sudo apt-get upgrade
```
**Enable automatic security updates**

```
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```


### Update Google auth to live server

1.  get your IP address, the one you're deploying at and it's not a public domain, let's say it's  `10.0.0.1`  as an example.
2.  add  `http://10.0.0.1.xip.io`  to your Authorized Javascript Origins on the Google Developer Console.
3.  open your site by visiting  `http://10.0.0.1.xip.io`
    


**Install required**

    pip install --upgrade google-api-python-client oauth2client
    pip install requests
    pip install --upgrade psycopg2
    sudo apt-get install apache2 libapache2-mod-wsgi-py3 git
    
### 3. Secure the server


**3.1 Change the SSH port from 22 to 2200**
-   Edit the  **sshd_config**   file  by typing :  `sudo vi /etc/ssh/sshd_config` then `i` 
-   Change the port number from  `22`  to  `2200`.
-   Save and exit : press **Esc** then type `:wq`
-   Restart SSH:  `sudo service ssh restart`.

**3.2 Configure the Uncomplicated Firewall (UFW)**
-   Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

```
sudo ufw status                  
sudo ufw default deny incoming   
sudo ufw default allow outgoing  
sudo ufw allow 2200/tcp          
sudo ufw allow www               
sudo ufw allow 123/udp           
sudo ufw deny 22        
```
....
- Activate UFW  : `sudo ufw enable`. 
- Check the status of UFW : `sudo ufw status`.
- Click on the `Manage` option of the Amazon Lightsail Instance, then  `Networking` tab,  then change the firewall configuration to match your internal firewall settings : 
  - Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.
...

In local terminal run: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@54.160.176.9`.


### 4. Give  grader  access

**4.1 Create a new user account named  `grader`**

-  Log in as  `ubuntu/root`, add user:  `sudo adduser grader`.
-   Give it password  and  information.

**4.2  Give  `grader`  the permission to sudo**
```
   sudo nano /etc/sudoers.d/grader
   ```
   then write : 

    grader  ALL=(ALL:ALL) ALL
    
 
  
-   Save and exit using CTRL+O  then CTRL+X  then press y.
    ...
    
   Verify that  `grader`  has sudo permissions. Run  `sudo su - grader`, enter the password, run  `sudo -l`  and enter the password again


**4.3  Create an SSH key pair for  `grader`  using the  `ssh-keygen`  tool**


-   **On the local machine ( user ubuntu ) :**
    -   Run  `ssh-keygen`
    -   Enter file in which to save the key in the local directory  `grader`
    -   Enter in a passphrase twice. Two files will be generated (  `grader`  and  `grader.pub`)
    -   Run  `cat grader_key.pub`  and copy the contents of the file
    -   Log in to the grader's virtual machine
-   **On the grader's virtual machine ( `sudo su - grader` ) :**
    -   Create a new directory called  `~/.ssh`  (`mkdir .ssh`)
    -   Run  `sudo nano ~/.ssh/authorized_keys`  and paste the content into this file, save and exit
    -   Give the permissions:  `chmod 700 .ssh`  and  `sudp chmod 600 .ssh/authorized_keys` ```

    -   Check in  `/etc/ssh/sshd_config`  file if  `PasswordAuthentication`  is set to  `no`
    -   Restart SSH:  `sudo service ssh restart`
-   **Move to the local machine** ( `sudo su - ubunto` ), run:  `ssh -i grader -p 2200 grader@YOURPUBLICIP`.

### 5. Deploy the Item Catalog project using gunicorn

**5.1  Cofigure local timezone to UTC**
    
    
    sudo timedatectl set-timezone UTC
    sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8


**5.2 Install required**

    pip install --upgrade google-api-python-client oauth2client
    pip install requests
    pip install --upgrade psycopg2
    sudo apt-get install apache2 libapache2-mod-wsgi-py3 git
**5.4. Install and configure PostgreSQL**

```
sudo apt-get install libpq-dev python3-dev
sudo apt-get install postgresql postgresql-contrib

sudo su - postgres
psql
```

Then

```
CREATE USER catalog WITH PASSWORD 'password';
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
`````


**add this in your catalog project** 

```
engine = create_engine('postgresql://catalog:password@localhost/catalog')

```

### Update Google auth to live server

1.  get your IP address, the one you're deploying at and it's not a public domain, let's say it's  `10.0.0.1`  as an example.
2.  add  `http://10.0.0.1.xip.io`  to your Authorized Javascript Origins on the Google Developer Console.
3.  open your site by visiting  `http://10.0.0.1.xip.io`
    

**5.3 Project Configuration**  

    sudo apt-get install python-pip nginx
    sudo /etc/init.d/nginx start
    sudo rm /etc/nginx/sites-enabled/default
    sudo touch /etc/nginx/sites-available/flask_setting
    sudo ln -s /etc/nginx/sites-available/flask_setting  etc/nginx/sites-enabled/flask_setting
    sudo vi etc/nginx/sites-enabled/flask_setting

then press `i` then write : 

    server {
          location / {
              proxy_pass http:127.0.0.1:8000;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
              }
    }

then save & exit by : 
`Esc` then write `:wq`

    sudo  /etc/init.d/nginx restart
    create virtual environment 
    pip install virtualenv
    mkdir my_app
    cd my_app
    virtualenv env
    source env/bin/activate
    install gunicorn
    pip install flask gunicorn
    which gunicorn              #to chack

**clone project** 

    git clone   < github project link >    catalog
    
    cd catalog



### Launch the Web Application

   Change the ownership of the project directories:  `sudo chown -R www-data:www-data catalog/`

- `cd catalog`
-    `python lotsonmenus.py`
- `gunicorn final:app`
-   Open your browser to  http://54.160.176.9.xip.io

**@aya_gaser**







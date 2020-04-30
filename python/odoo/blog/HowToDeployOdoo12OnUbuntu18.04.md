##  How to deploy Odoo 12 on Ubuntu 18.04  

原文链接[http://www.khcloud.net:4082/?thread-533.htm](http://www.khcloud.net:4082/?thread-533.htm)  


Odoo is the most popular all-in-one business software in the world. It offers a range of business applications including CRM, website, e-Commerce, billing, accounting, manufacturing, warehouse, project management, inventory and much more, all seamlessly integrated.

Odoo can be installed in several different ways. The easiest and quickest way to install Odoo is by using their official APT repositories.

If you want to have more flexibility such as running multiple Odoo versions on a same system then you can either use docker and docker compose or install Odoo in a virtual environment.

This tutorial covers the steps required for installing and configuring Odoo 12 for production using Git source and Python virtual environment on an Ubuntu 18.04 system.

Before you begin
Login to you Ubuntu machine as a sudo user and update the system to the latest packages:

sudo apt update && sudo apt upgrade
Install Git, Pip, Node.js and the tools required to build Odoo dependencies:

sudo apt install git python3-pip build-essential wget python3-dev python3-venv python3-wheel libxslt-dev libzip-dev libldap2-dev libsasl2-dev python3-setuptools node-less
Create Odoo user
Create a new system user for Odoo named odoo12 with home directory /opt/odoo12 using the following command:

sudo useradd -m -d /opt/odoo12 -U -r -s /bin/bash odoo12
You can use any name for your Odoo user as long you create a PostgreSQL user with the same name.

Install and Configure PostgreSQL
Install the PostgreSQL package from the Ubuntu’s default repositories:

sudo apt install postgresql
Once the installation is completed, create a PostgreSQL user with the same name as the previously created system user, in our case that is odoo12:

sudo su - postgres -c "createuser -s odoo12"
Install Wkhtmltopdf
The wkhtmltox package provides a set of open-source command line tools which can render HTML into PDF and various image formats. In order to print PDF reports, you will need the wkhtmltopdf tool. The recommended version for Odoo is 0.12.1 which is not available in the official Ubuntu 18.04 repositories.

Download the package using the following wget command:

wget https://builds.wkhtmltopdf.org/0.12.1.3/wkhtmltox_0.12.1.3-1~bionic_amd64.deb
Once the download is completed install the package by typing:

sudo apt install ./wkhtmltox_0.12.1.3-1~bionic_amd64.deb
Install and Configure Odoo
We will install Odoo from the GitHub repository inside an isolated Python virtual environment.

Before starting with the installation process, change to user “odoo12”:

sudo su - odoo12
Start by cloning the Odoo 12 source code from the Odoo GitHub repository:

git clone https://www.github.com/odoo/odoo --depth 1 --branch 12.0 /opt/odoo12/odoo
Once the source code is downloaded, create a new Python virtual environment for the Odoo 12 installation:

cd /opt/odoo12
python3 -m venv odoo-venv
Next, activate the environment with the following command:

source odoo-venv/bin/activate
Install all required Python modules with pip3:

pip3 install wheel
pip3 install -r odoo/requirements.txt
If you encounter any compilation errors during the installation, make sure that you installed all of the required dependencies listed in the Before you begin section.

Deactivate the environment using the following command:

deactivate
Create a new directory for the custom addons:

mkdir /opt/odoo12/odoo-custom-addons
Switch back to your sudo user:

exit
Next, create a configuration file, by copying the included sample configuration file:

sudo cp /opt/odoo12/odoo/debian/odoo.conf /etc/odoo12.conf
Open the file and edit it as follows:

sudo nano /etc/odoo12.conf
/etc/odoo12.conf

[options] 
; This is the password that allows database operations: 
admin_passwd = my_admin_passwd 
db_host = False 
db_port = False 
db_user = odoo12 
db_password = False 
addons_path = /opt/odoo12/odoo/addons,/opt/odoo12/odoo-custom-addons
CopyDo not forget to change the my_admin_passwd to something more secure.

Create a Systemd Unit File
To run Odoo as a service we need to create a service unit file in the /etc/systemd/system/ directory.

Open your text editor and paste the following configuration:

sudo nano /etc/systemd/system/odoo12.service
/etc/systemd/system/odoo12.service

[Unit] 
Description=Odoo12 
Requires=postgresql.service 
After=network.target postgresql.service 

[Service] 
Type=simple 
SyslogIdentifier=odoo12 
PermissionsStartOnly=true 
User=odoo12 
Group=odoo12 
ExecStart=/opt/odoo12/odoo-venv/bin/python3 /opt/odoo12/odoo/odoo-bin -c /etc/odoo12.conf StandardOutput=journal+console 

[Install] 
WantedBy=multi-user.target
Copy

Notify systemd that a new unit file exist and start the Odoo service by running:

sudo systemctl daemon-reload
sudo systemctl start odoo12
Check the service status with the following command:

sudo systemctl status odoo12
The output should look something like below indicating that Odoo service is active and running.

* odoo12.service - Odoo12
    Loaded: loaded (/etc/systemd/system/odoo12.service; disabled; vendor preset: enabled)
        Active: active (running) since Tue 2018-10-09 14:15:30 PDT; 3s ago  Main PID: 24334 (python3)
             Tasks: 4 (limit: 2319)
             CGroup: /system.slice/odoo12.service
                         `-24334 /opt/odoo12/odoo-venv/bin/python3 /opt/odoo12/odoo/odoo-bin -c /etc/odoo12.conf
Enable the Odoo service to be automatically started at boot time:

sudo systemctl enable odoo12
If you want to see the messages logged by the Odoo service you can use the command below:

sudo journalctl -u odoo12
Test the Installation
Open your browser and type: http://<your_domain_or_IP_address>:8069

Assuming the installation is successful, a screen similar to the following will appear:




Configure Nginx as SSL Termination Proxy
Ensure that you have met the following prerequisites before continuing with this section:

Domain name pointing to your public server IP. In this tutorial we will use example.com.
Nginx installed.
SSL certificate for your domain. You can install a free Let’s Encrypt SSL certificate .
The default Odoo web server is serving traffic over HTTP. To make our Odoo deployment more secure we will configure Nginx as a SSL termination proxy that will serve the traffic over HTTPS.

SSL termination proxy is a proxy server which handles the SSL encryption/decryption. This means that our termination proxy (Nginx) will handle and decrypt incoming TLS connections (HTTPS), and it will pass on the unencrypted requests to our internal service (Odoo) so the traffic between Nginx and Odoo will not be encrypted (HTTP).

Using a reverse proxy gives you a lot of benefits such as Load Balancing, SSL Termination, Caching, Compression, Serving Static Content and more.

In this example we will configure SSL Termination, HTTP to HTTPS redirection, WWW to non-WWW redirection, cache the static files and enable GZip compression.

Open your text editor and create the following file:

sudo nano /etc/nginx/sites-enabled/example.com
/etc/nginx/sites-enabled/example.com

# Odoo servers
 upstream odoo {
   server 127.0.0.1:8069;
 }
 
 upstream odoochat {
   server 127.0.0.1:8072;
 }
 
 # HTTP -> HTTPS 
 server {
      listen 80;
      server_name www.example.com example.com;
      
      include snippets/letsencrypt.conf;
      return 301 https://example.com$request_uri;
 }
 
 # WWW -> NON WWW 
 server {
      listen 443 ssl http2;
      server_name www.example.com;
      
      ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
      ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
      
      include snippets/ssl.conf;
      include snippets/letsencrypt.conf;
      return 301 https://example.com$request_uri;
  }
  server {
       listen 443 ssl http2;
       server_name example.com;
       
       proxy_read_timeout 720s;
       proxy_connect_timeout 720s;
       proxy_send_timeout 720s;
       
       # Proxy headers
       proxy_set_header X-Forwarded-Host $host;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header X-Real-IP $remote_addr;
       
       # SSL parameters
       ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
       ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;
       
       include snippets/ssl.conf;
       include snippets/letsencrypt.conf;
       
       # log files
       access_log /var/log/nginx/odoo.access.log;
       error_log /var/log/nginx/odoo.error.log;
       
       # Handle longpoll requests
       location /longpolling {
                proxy_pass http://odoochat;
       }
       
       # Handle / requests
       location / {
               proxy_redirect off;
               proxy_pass http://odoo;
       }
       
       # Cache static files
       location ~* /web/static/ {
                proxy_cache_valid 200 90m;
                proxy_buffering on;
                expires 864000;
                proxy_pass http://odoo;
       }
       
       # Gzip
       gzip_types text/css text/less text/plain text/xml application/xml application/json application/javascript;
       gzip on
}
CopyDon’t forget to replace example.com with your Odoo domain and set the correct path to the SSL certificate files. The snippets used in this configuration are created in this guide .

Once you are done, restart the Nginx service with:

sudo systemctl restart nginx
Next, we need to tell Odoo that we will use proxy. To do so, open the configuration file and add the following line:

/etc/odoo12.conf

proxy_mode = True
Restart the Odoo service for the changes to take effect:

sudo systemctl restart odoo12
At this point, your server is configured and you can access your Odoo instance at: https://example.com

Change the binding interface
This step is optional, but it is a good security practice.

By default, Odoo server listens to port 8069 on all interfaces. If you want to disable direct access to your Odoo instance you can either block the port 8069 for all public interfaces or force Odoo to listen only on the local interface.

In this guide we will configure Odoo to listen only on 127.0.0.1. Open the configuration add the following two lines at the end of the file:

/etc/odoo12.conf

xmlrpc_interface = 127.0.0.1 
netrpc_interface = 127.0.0.1
Save the configuration file and restart the Odoo server for the changes to take effect:

sudo systemctl restart odoo12
Enable Multiprocessing
By default, Odoo is working in multithreading mode. For production deployments, it is recommended to switch to the multiprocessing server as it increases stability, and make better usage of the system resources. In order to enable multiprocessing we need to edit the Odoo configuration and set a non-zero number of worker processes.

The number of workers is calculated based on the number of CPU cores in the system and the available RAM memory.

According to the official Odoo documentation to calculate the workers number and required RAM memory size we will use the following formulas and assumptions:

Worker number calculation

theoretical maximal number of worker = (system_cpus * 2) + 1
1 worker can serve ~= 6 concurrent users
Cron workers also requires CPU
RAM memory size calculation

We will consider that 20% of all requests are heavy requests, while 80% are lighter ones. Heavy requests are using around 1 GB of RAM while the lighter ones are using around 150 MB of RAM
Needed RAM = number_of_workers * ( (light_worker_ratio * light_worker_ram_estimation) + (heavy_worker_ratio * heavy_worker_ram_estimation) )
If you do not know how many CPUs you have on your system you can use the following command:

grep -c ^processor /proc/cpuinfo
Let’s say we have a system with 4 CPU cores, 8 GB of RAM memory and 30 concurrent Odoo users.

30 users / 6 = **5** (5 is theoretical number of workers needed )
(4 * 2) + 1 = **9** ( 9 is the theoretical maximum number of workers)
Based on the calculation above we can use 5 workers + 1 worker for the cron worker which is total of 6 workers.

Calculate the RAM memory consumption based on the number of the workers:

RAM = 6 * ((0.8*150) + (0.2*1024)) ~= 2 GB of RAM
The calculation above show us that our Odoo installation will need around 2GB of RAM.

To switch to multiprocessing mode, open the configuration file and append the following lines:

/etc/odoo12.conf

limit_memory_hard = 2684354560 
limit_memory_soft = 2147483648 
limit_request = 8192 
limit_time_cpu = 600 
limit_time_real = 1200 
max_cron_threads = 1 
workers = 5
Restart the Odoo service for the changes to take effect:

sudo systemctl restart odoo12
The rest of the system resources will be used by other services that run on this system. In this guide we installed Odoo along with PostgreSQL and Nginx on a same server and depending on your setup you may also have other services running on your server.

Conclusion
This tutorial walked you through the installation of Odoo 12 on Ubuntu 18.04 in a Python virtual environment using Nginx as a reverse proxy. You also learned how to enable multiprocessing and optimize Odoo for production environment.

You may also want to check our tutorial about how to create automatic daily backups of your Odoo databases.

If you have questions, feel free to leave a comment below.

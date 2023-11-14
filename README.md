# automate-loadbalancer-configuration

In our last project, we deployed two backend servers with a load balancer and distributed traffic across the web servers via the terminal. In this project, we will be automating our previous project https://github.com/BusisiwePieterson/Nginx-Loadbalancer using Shell scripting.

With shell scripting, we can write scripts to automate tasks such as installing software packages, configuring systems, setting up network connections, and managing files and directories. This reduces the manual effort required to perform these tasks, which in turn saves time, improves efficiency, and reduces errors.


## Deploying and Configuring the Webservers

1. Provision two EC2 instances and connect to them via the terminal using SSH client.


![image](images/Screenshot_1.png)

2. Open port **8000** on each webserver to allow traffic from anywhere **(0.0.0.0)**

![image](images/Screenshot_2.png)

3. Run `sudo vi install.sh` to open a file and paste the below script:

```
#!/bin/bash

####################################################################################################################
##### This automates the installation and configuring of apache webserver to listen on port 8000
##### Usage: Call the script and pass in the Public_IP of your EC2 instance as the first argument as shown below:
######## ./install_configure_apache.sh 127.0.0.1
####################################################################################################################

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure

PUBLIC_IP=$1

[ -z "${PUBLIC_IP}" ] && echo "Please pass the public IP of your EC2 instance as an argument to the script" && exit 1

sudo apt update -y &&  sudo apt install apache2 -y

sudo systemctl status apache2

if [[ $? -eq 0 ]]; then
    sudo chmod 777 /etc/apache2/ports.conf
    echo "Listen 8000" >> /etc/apache2/ports.conf
    sudo chmod 777 -R /etc/apache2/

    sudo sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8000>/' /etc/apache2/sites-available/000-default.conf

fi
sudo chmod 777 -R /var/www/
echo "<!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: "${PUBLIC_IP}"</p>
        </body>
        </html>" > /var/www/html/index.html

sudo systemctl restart apache2


```



![image](images/Screenshot_3.png)

5. Run `sudo chmod +x install.sh` to change permissions on the file to make it executable.

6. Run `./install.sh PUBLIC_IP` *e.g* `./install.sh  34.29.230.59` do this for each web server.

![image](images/Screenshot_4.png)


7.  Copy and paste each web server's Public IP on your web browser.

![image](images/Screenshot_5.png)


![image](images/Screenshot_6.png)


## Deployment of Nginx as a Load Balancer using Shell Scripting

1. We will first provision an EC2 instance on Ubuntu open port 80 to anywhere and SSH to it via the terminal.

![image](images/Screenshot_7.png)


2. Open `sudo vi nginx.shell` and paste the below script:
```

#!/bin/bash

######################################################################################################################
##### This automates the configuration of Nginx to act as a load balancer
##### Usage: The script is called with 3 command line arguments. The public IP of the EC2 instance where Nginx is installed
##### the webserver urls for which the load balancer distributes traffic. An example of how to call the script is shown below:
##### ./configure_nginx_loadbalancer.sh PUBLIC_IP Webserver-1 Webserver-2
#####  ./configure_nginx_loadbalancer.sh 127.0.0.1 192.2.4.6:8000  192.32.5.8:8000
############################################################################################################# 

PUBLIC_IP=$1
firstWebserver=$2
secondWebserver=$3

[ -z "${PUBLIC_IP}" ] && echo "Please pass the Public IP of your EC2 instance as the argument to the script" && exit 1

[ -z "${firstWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the second argument to the script" && exit 1

[ -z "${secondWebserver}" ] && echo "Please pass the Public IP together with its port number in this format: 127.0.0.1:8000 as the third argument to the script" && exit 1

set -x # debug mode
set -e # exit the script if there is an error
set -o pipefail # exit the script when there is a pipe failure


sudo apt update -y && sudo apt install nginx -y
sudo systemctl status nginx

if [[ $? -eq 0 ]]; then
    sudo touch /etc/nginx/conf.d/loadbalancer.conf

    sudo chmod 777 /etc/nginx/conf.d/loadbalancer.conf
    sudo chmod 777 -R /etc/nginx/

    
    echo " upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server  "${firstWebserver}"; # public IP and port for webserser 1
            server "${secondWebserver}"; # public IP and port for webserver 2

            }

           server {
            listen 80;
            server_name "${PUBLIC_IP}";

            location / {
                proxy_pass http://backend_servers;   
            }
    } " > /etc/nginx/conf.d/loadbalancer.conf
fi

sudo nginx -t

sudo systemctl restart nginx


```


![image](images/Screenshot_8.png)

2. Change the file permissions by running `sudo chmod +x nginx.sh`

3. Next, `./nginx.sh PUBLIC_IP Webserver1 Webserver2` *e.g* `./nginx.sh 3.89.7.102 34.229.230.59:8000 3.90.42.100:8000`  to run the script.


![image](images/Screenshot_11.png)


## Verify the Setup


**Webserver 1**

![image](images/Screenshot_5.png)

**Webserver 2**

![image](images/Screenshot_6.png)


**Load Balancer**

![image](images/Screenshot_14.png)

![image](images/Screenshot_13.png)

# THE END!



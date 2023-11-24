# Project8automatingwebserver
# Automating load balancer configuration with shellscripting

Streamline your load balancer configuration with ease using shellscripting and simple CI/CD on Jenkins. This project demonstrates how to automate the set upt and maintenacce of your load balancer using freestyle job, enhancing efficiency and reducing manual effort.

# Automate the Deployment of Webservers

In Implementing load balancer with Nginx course, we deployed two backend servers, with a load balancer distributing traffic across the webservers. We did that by typng commands right on our terminal

In this project, we will do that by automating the entire process by writing a shell script that when ran,all that we did manaully will be done for us automatically. As Devops engineers, automation is at the heart of the work we do. Automation helps us speed the deployment of services and reduce the chance of making errors in our day to day activity..

# Deploying and Configuring the Webservers

The following steps are involve:

- Step1: Provison and EC2 instance running on Ubuntu 20.04

![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/b36d7e25-bb03-4caf-ba2d-4b889c58a176)


- Step 2: Open port 8000 to allow traffic from anywhere using the security group and editing the in bound rule, then save the rule

![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/a43a464b-9a93-48e5-be05-f0674c4cf6bf)

![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/0b77786c-66c1-484d-8cd3-f5bc954770a3)

- Step3 : Connect to the web server created via the terminal ssh client by coping and pasting the ssh key on the terminal as below image

 ![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/15720c75-25ee-4879-9fc8-c247e6c208c3)

 ![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/42780ac5-bd39-43d1-b91d-06b56c917bf3)

 ![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/8a6c5d12-aed1-49ac-b40a-019066cbc235)


- Step4:  On the terminal, open a file with command **sudo vi install.sh** and copy and paste the below script in it,

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
            <title> My EC2 Instance </title>
        </head>
        <body>
        
           -<h1> Welcome to my EC2 instance </h1>
            <p>Public IP: "${PUBLIC_IP}"</p>
        </body>
        </html>" > /var/www/html/index.html

sudo systemctl restart apache2


- Ensure to adjust the Public IP on the script to your instance Public IP (16.170.230.246) as in below image: 

![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/943bbb32-1c5a-4cfc-a109-339ca2244adb)

- then save by pressing the escape key, then shift + :wq! and enter.

- Step5 : On the terminal, change the file permission to make it executable using the command:**sudo chmod + x install.sh**

- Step6 : Run the script on the terminal with the following command : **./install.sh PUBLIC IP**

  Image shall be as the following display trail:

![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/4e688ca0-7010-492d-9ac5-c45f45c659b3)


![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/20cab21f-8306-4b99-8510-fa8d74c92340)


![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/8723ff6d-d196-4b7e-9370-37266f7854a6)



# DEPLOYMENT OF NGINX AS LOAD BALANCER USING SHELL SCRIPT

Having successfully deployed and configure two webservers, we will move on to the load balancer. As a pre-requisite we need to provision another EC2 instance running ubuntu 22.04 that will work as the NGINX load balancer, we will open port80 to anywhere using the security group on this nginx load balancer and connect to the load balancer via the terminal, after which we follow the below steps.

- Step1: On the terminal, open a file name nginx.sh with the following command **sudo vi nginx.sh**

- Step2: Copy and paste the below script inside the file: 


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

- Step3: Before exiting the file, remember to the change the public IP, the web servers IP and port as in the below image:

  ![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/0710cc5b-a035-4846-9ff1-1e3ef8bcfc43)

- Step4: After inputing the respective IP address and Port for both the server and the nginx load balancer, we press the escape botton, then :wq! and enter to save the file and exit.

- Step5 : On the terminal, change the file permission to make it executable using the command:**sudo chmod + x nginx.sh**

- Step6 : Run the script on the terminal with the following command : **./nginx.sh PUBLIC_IP Webserver-1 Webserver-2**

**The script will load and the following images will be as its display when loading(installing)**                                                                                     

![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/5023a5f6-7396-4dd1-95c2-6daa1d910542)

![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/3049d273-c196-4636-825b-f2d9dc282a6a)

![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/99eb7b48-c502-4c0d-9659-770b7d746165)

![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/2572788e-a277-45ae-a87e-51777ab6d45f)

![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/ba82ed6d-66c2-48c4-858d-ca22016f8518)




   # Verify set up

 - To verify we go to any browser of our choice and paste the webservers IP public address on them.

  Images should be as below:

  ![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/9577effa-9cb4-47f4-9af9-b1d653be9f29)


 
  ![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/69b864fc-ddc1-427b-8de7-8997dc9956aa)



  ![image](https://github.com/DevopMudi/Project8automatingwebserver/assets/149855241/83b17d5f-e65f-45eb-a09d-1d0916046cf1)


# References:

[How to configure load balancing using Nginx] {https://upcloud.com/resources/tutorials/configure-load-balancing-nginx} 

AI BING




















































































































































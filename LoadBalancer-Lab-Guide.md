# Creating a Load Balancer Lab with Nginx and Multiple Vagrant Servers

## Github repo url: https://github.com/Ayomidev/Load-balancer-labguide.git

## Objective

Create a load balancer lab using Nginx, three web servers, and one Nginx server. The web servers should host informative HTML webpages.

### Step 1: Prepare the Lab Directory

Create your project directory and navigate into it:
mkdir load_balancer_lab
cd load_balancer_lab

### Step 2: Set Up Vagrant Configuration

Create a `Vagrantfile` in the `load_balancer_lab` directory with the following content:

Vagrant.configure("2") do |config|
config.vm.define "nginx" do |nginx|
nginx.vm.box = "ubuntu/bionic64"
nginx.vm.network "private_network", type: "dhcp"
nginx.vm.provision "shell", path: "provision/nginx.sh"
end

# Configuration for Web Server 1

config.vm.define "web1" do |web1|
web1.vm.box = "ubuntu/bionic64"
web1.vm.network "private_network", type: "dhcp"
web1.vm.provision "shell", path: "provision/webserver.sh"
end

# Configuration for Web Server 2

config.vm.define "web2" do |web2|
web2.vm.box = "ubuntu/bionic64"
web2.vm.network "private_network", type: "dhcp"
web2.vm.provision "shell", path: "provision/webserver.sh"
end

# Configuration for Web Server 3

config.vm.define "web3" do |web3|
web3.vm.box = "ubuntu/bionic64"
web3.vm.network "private_network", type: "dhcp"
web3.vm.provision "shell", path: "provision/webserver.sh"
end
end

### Step 3: Create Provisioning Scripts

Create a directory named `provision` in your project directory.
mkdir provision

Inside the `provision` directory, create the `nginx.sh` script:

#!/bin/bash
apt-get update
apt-get install -y nginx

Also, create the `webserver.sh` script:

#!/bin/bash
apt-get update
apt-get install -y nginx

# Check the hostname and set content accordingly

if [[$(hostname) == "web1"]]; then
echo "This is Web Server 1" > /var/www/html/index.html
elif [[$(hostname) == "web2"]]; then
echo "This is Web Server 2" > /var/www/html/index.html
elif [[$(hostname) == "web3"]]; then
echo "This is Web Server 3" > /var/www/html/index.html
else
echo "This is a default page" > /var/www/html/index.html
fi

Ensure both scripts are executable:
chmod +x provision/nginx.sh
chmod +x provision/webserver.sh

### Step 4: Initialize and Start Vagrant Machines

Run the following command to initialize and start your Vagrant machines:

vagrant up

### Step 5: Configure Nginx Load Balancer

SSH into the Nginx virtual machine:

vagrant ssh nginx

Edit the Nginx configuration file:

sudo nano /etc/nginx/sites-available/default

Update the file to include:

server {
listen 80;

    location / {
        proxy_pass http://web_servers;
    }

    upstream web_servers {
        server web1:80;
        server web2:80;
        server web3:80;
    }

}
Replace web1, web2, web3 with their correct private IPs.
Restart Nginx to apply the changes:
sudo systemctl restart nginx

### Step 6: Test the Load Balancer

On your local machine, open a web browser and navigate to the private IP address of the Nginx VM. You should see the load-balanced content from the web servers.

### Step 7: Verify Load Balancing

To check the access logs, SSH into each web server and run:
vagrant ssh web1
tail -f /var/log/nginx/access.log

vagrant ssh web2
tail -f /var/log/nginx/access.log

vagrant ssh web3
tail -f /var/log/nginx/access.log

You should see requests being distributed among the web servers.

### Step 8: Check Load Balancing in a Web Browser

Open a web browser and enter the private IP address of the Nginx VM. Refresh the page multiple times to observe the round-robin load balancing in action.

This setup should give you a functional load balancing environment using Nginx and Vagrant.

This ReadMe file is intended to provide a guide for administrators / hands-on technical persons regarding the configuration required to deploy ELK and Beats in an Ansible / Docker environment. Specific IPs referenced suit the Azure Lab environment I constructed.

## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

[ https://github.com/forrestaj64/AZURE_Docker_DVWA_ELK/blob/main/Images/ELK_diagram.pdf ]

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. 
Alternatively, select portions of a comprehensive playbook may be used to install only certain pieces of it, such as Filebeat.

In may case, I chose to create individual playbooks for each elk component (this allowed me more hands-on and an appreciation of the nuances of each product)

filebeat-playbook.yml :-

 - name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

  - name: install filebeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start
 


metricbeat-playbook.yml :-

- name: installing and launching metricbeat
  hosts: webservers
  become: yes
  tasks:

  - name: download metricbeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

  - name: install metricbeat deb
    command: dpkg -i metricbeat-7.4.0-amd64.deb

  - name: drop in metricbeat.yml
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

  - name: enable and configure system module
    command: metricbeat modules enable system

  - name: setup metricbeat
    command: metricbeat setup

  - name: start metricbeat service
    command: service metricbeat start   	|
 
 
This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.
- load balancers protect availability, by directing traffic to available nodes. The failure of a single node need not cause a critical outage.
- A jump box provides us a means to connect to our hosts that are in a secure zone. Using a jump box means we need only allow external connections to a single point.  This provides security advantage to allowing access to each host directly.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the logs and system traffic (load).
- Filebeat monitors the log files or folders specified and collects log events.
- Metricbeat records Operating System and Services information.

The configuration details of each machine may be found below.

| Name     | Function  | IP Address | Operating System |
|----------|---------- |------------|------------------|
| Jump-Box | Gateway   | 10.0.0.6   | Linux            |
| Web-1    | WebServer | 10.0.0.4   | Linux            |
| Web-2    | WebServer | 10.0.0.5   | Linux            |
| Web-3    | WebServer | 10.0.0.7   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump-Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 101.XXX.XXX.XXX (my public IP address through telstra, as shown via https://ip.me/ - obsfucated here and in diagrams for privacy reasons)

Machines within the network can only be accessed by SSH.
- The Jump-Box machine can access the ELK VM, IP address 10.0.0.6 > 10.1.0.4 (SSH)

A summary of the access policies in place can be found in the table below...

|   Name       |  Publicly Accessible | Allowed IP Addresses |
|--------------|-----------------------|----------------------|
| Jump-Box     |      Yes              | 101.XXX.XXX.XXX      | (my home IP address)
|Load Balancer |      Yes              | any tcp port 80 only |
|  Web-1       |       No              |  23.101.213.139      | (Load Balancer)
|  Web-2       |       No              |  23.101.213.139      | (Load Balancer)
|  Web-3       |       No              |  23.101.213.139      | (Load Balancer)
|  ELK VM      |      Yes              | 101.XXX.XXX.XXX      | (my home IP address)

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- Automating configuration with Ansible provides us with consistency of configuration and a central point of administration. 
 Where we need to make a configuration change, we edit the playbook and deploy to the group.  
 This means we can deploy to multiple machines quickly and ensures we have identical configuration across the group of machines.

The playbook implements the following tasks:
- install docker.io (package)
- install python3-pip (python package manager)
- Install pip module (for docker)
- use sysctl module to allocate memory for the elk container
- download and launch a docker elk container

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

[ https://github.com/forrestaj64/AZURE_Docker_DVWA_ELK/blob/main/Images/docker_ps_elk_server.png ]

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- Web-1: 10.0.0.5, Web-2 :10.0.0.4, Web-3: 10.0.0.7

We have installed the following Beats on these machines:
- filebeat, metricbeat.

These Beats allow us to collect the following information from each machine:
- Filebeat allows us to forward and centralise application and service logs. 
	We can then use the Kibana GUI to search for strings, such as error messages across the whole web pool.
- Metricbeat gathers metrics from systems and services so these can be viewed through Kibana.
	CPU activity, memory utilisation etc. from each of the systems can be viewed, searched and visualised.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the playbook file to /etc/ansible/files.
- Update the hosts file to include the IP of the ELK server - add the tag for [elk].
- Run the playbook, and navigate to the Kibana entry page: http://20.37.244.38:5601/app/kibana to check that the installation worked as expected.

 Points to Note:
- install-elk.yml is the playbook. You copy it to /etc/ansible/files
- We update /etc/ansible/hosts.yml to make Ansible run the playbook on a specific machine(s).
  In the install file we specify a hosts entry, which represents a group name in the hosts file.  Ansible references this group name in the hosts file to acquire the IPs of the servers that we wish to deploy to.  For ELK we specified hosts: elk and in the hosts file we have [elk] with 10.1.0.4 listed.  For filebeat we specific hosts: webservers and in the hosts file we have populated [webservers] with 10.0.0.4, 10.0.0.5, 10.0.0.7 (being the IPs of our webservers)
- Check that the ELK server is running via the Kibana entry page: http://20.37.244.38:5601/app/kibana 
- displaying the Kibana page in your browser is dependent upon an inbound rule permitting the traffic to port 5601 on the ELK server (public address).

--------------------------------------------------------------------------------------------------------------------------------------------------------
Commands used in the process; 

Connect to Jump-Box and attach to docker

ssh azadmin@52.237.208.202
sudo systemctl status docker
sudo systemctl start docker
sudo su
docker container list -a
docker start 9bd60c53ba2b
docker attach 9bd60c53ba2b

Set docker to start with the system

sudo systemctl enable docker

Download the filebeat config file to the ansible container

curl https://gist.githubusercontent.com/slape/5cc350109583af6cbe577bbcc0710c93/raw/eca603b72586fbe148c11f9c87bf96a63cb25760/Filebeat > /etc/ansible/files/filebeat-config.yml

Download Metricbeat

curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb

Install metricbeat

sudo dpkg -i metricbeat-7.6.1-amd64.deb

Edit the configuration, eg; metricbeat - /etc/metricbeat/metricbeat.yml

output.elasticsearch:
  hosts: ["<es_url>"]
  username: "elastic"
  password: "<password>"
setup.kibana:
  host: "<kibana_url>"

Integrate the download and install commands into your playbooks, along with the service configuration settings (as shown in the playbooks above)
You should have already edited your ansible hosts file (/etc/ansible/files) to specify the servers for installation.

Run your playbook to install ELK

cd /etc/ansible
 ansible-playbook install_elk.yml
 
Note: your ELK installation should be verified before you proceed with beats.

Run your playbooks to install your beats

 ansible-playbook filebeat-playbook.yml
 ansible-playbook metricbeat-playbook.yml

Verify that you can see your beats in Kibana console

http://20.37.244.38:5601/app/kibana#/management/kibana/index_patterns?_g=()

[ https://github.com/forrestaj64/AZURE_Docker_DVWA_ELK/blob/main/Images/Two_Beats_in_Kibana.png ]

You can now customise your Space (Dashboard) to visualise the metrics and data to best provide your needs in management of your applications and infrastructure.



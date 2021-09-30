# Project1
BCS Project
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![TODO: Update the path with the name of your diagram](Desktop/Images/Diagram.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the _____ file may be used to install only certain pieces of it, such as Filebeat.

  - 
  elk.yml
  
  ---
        - name: Config Web VM with Docker
          hosts: elk
          become: true
          tasks:
          - name: Uninstall apache2
            apt:
              name: apache2
              state: absent
          - name: docker.io
            apt:
              force_apt_get: yes
              update_cache: yes
              name: docker.io
              state: present
          - name: Install pip3
            apt:
              force_apt_get: yes
              name: python3-pip
              state: present
          - name: Install docker python module
            pip:
              name: docker
              state: present
          - name: Increase Virtual Memory
            command: sysctl -w vm.max_map_count=262144
          - name: use more memory
            sysctl:
              name: vm.max_map_count
              value: 262144
              state: present
              reload: yes
          - name: Download and launch the docker web container
            docker_container:
              name: elk
              image: sebp/elk:761
              state: started
              restart_policy: always
              published_ports:
                - '5601:5601'
                - '9200:9200'
                - '5044:5044'
          - name: Enable Docker Service
            systemd:
              name: docker
              enabled: yes


filebeat-playbook.yml

---
- name: installing and launching filebeat
  hosts: webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb
 
  - name: install filebeat deb
    command: dpkg -i filebeat-7.6.1-amd64.deb

  - name: drop in filebeat.yml 
    copy:
      src: /etc/ansible/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: enable and configure system module
    command: filebeat modules enable system

  - name: setup filebeat
    command: filebeat setup

  - name: start filebeat service
    command: service filebeat start

  - name: enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes


metricbeat-playbook.yml

 ---
- name: installing and launching metricbeat
  hosts: webservers
  become: yes
  tasks:
  
  - name: download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb

  - name: install metricbeat
    command: dpkg -i metricbeat-7.6.1-amd64.deb

  - name: drop in metricbeat.yml
    copy:
      src: /etc/metricbeat/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

  - name: enable and configure docker module
    command: metricbeat modules enable docker

  - name: setup metricbeat
    command: metricbeat setup

  - name: start metricbeat
    command: metricbeat -e

  - name: enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes

-


This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly functional, in addition to restricting traffic to the network.
- What aspect of security do load balancers protect? What is the advantage of a jump box?
    Load balancers protect the avalibility of servers. An advantage of having a jumpbox is that it allows another step for authentication for logging in.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the network and system logs.
- What does Filebeat watch for? It monitors log files and location that are specified
- What does Metricbeat record? Records metric and statics

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| JumpBox  | Gateway  | 10.0.0.4   | Linux            |
| Web1     | Server   | 10.0.0.5   | Linux            |
| Web2     | Server   | 10.0.0.6   | Linux            |
| P1VM     | Server   | 10.1.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the JumpBox machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 199.111.213.242

Machines within the network can only be accessed by JumpBox Machine.
- Which machine did you allow to access your ELK VM?_
      JumpBox Machine
- What was its IP address?_
      52.170.90.12

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| JumpBox  | Yes                 | 52.170.90.12         |
| Web1     | No                  | 10.0.0.5             |
| Web2     | No                  | 10.0.0.6             |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- What is the main advantage of automating configuration with Ansible?_ Automating configuration saves time when it is necessary to make configurations to multiple machines

The playbook implements the following tasks:
- In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc._
- ...
Install Docker,
Start Docker container,
Enter Docker container,
Create elk playbook,
Run elk playbook,
- ...


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![TODO: Update the path with the name of your screenshot of docker ps output](Desktop/Images/docker_ps_output.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- _TODO: List the IP addresses of the machines you are monitoring_
10.0.0.5
10.0.0.6

We have installed the following Beats on these machines:
- _TODO: Specify which Beats you successfully installed_

Filebeats
Metricbeats


These Beats allow us to collect the following information from each machine:
- In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._

Filebeat collects log files from machines. An example . Metric Beats collect how well our system does.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the filebeat-configuration.yml file to /etc/ansible/roles.
- Update the filebeat-configuration file to include P1VM private IP
- Run the playbook, and navigate to http://13.77.206.135:5601 to check that the installation worked as expected.

Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_
filebeat-playbook.yml and copy it into the /etc/ansible/roles directory
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_ You use the groups part of the configuration files and put the IPs of the machines you want to use.
- _Which URL do you navigate to in order to check that the ELK server is running? http://13.77.206.135:5601/

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._

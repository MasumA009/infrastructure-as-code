# infrastructure-as-code: 

A way of creating and maanging computer systems through specilised code. instead of having to manually set up servers, we can write code that provisons or describes how the system should be built. Its like using a set of instructions to create an environment or infrastructure. 


# Ansible:
Ansible is a suite of software tools that enables infrastructure as code.


![Alt text](images/Infrastructure-as-code-diagram-57-1024x783.jpg)

In the diagram the user writes code which is then saved, stored and monitored through version control, it is then pushed to an automation server, and then managed and sent it to different architectures. It could be put on the cloud or to an on premises aritecture. 

(Read more here)[https://www.clickittech.com/devops/infrastructure-as-code-tools/]

How is it powerful/ useful? we can create and control as many nodes as we please through the controller. 

How is it simple? we only need to deal with the controller node, not every indivual node. 

Agentless: the software only needs to be installed on the master node, (controller). This makes it agentless. there is no need to install ansible on eevry device. 

## Example (plan):

The best way to understand it is to explain it through an example, 

![Alt text](images/Screenshot%202023-06-05%20135625.png)

1. We (the devloper) can use something like vagrant to create virtual machines, we can then use ssh to connect to a controller. 

2. 1 of the machines we will make is called an Ansible controller. This is like the root node, only 1 device needs this installed as we can control all other virtual machines from here. We connect through it via ssh. 

3. another machine is called the app machine/node, and the last machine is called the db machine/node. we can connect to these through passwords via the controller machine. 

4. The app and db machines work together to populate the data. (Create enviornment variables etc)

5. The app machine must run NodeJS 
6. The db machine must run the database. 

7. The Master node can be installed locally, whereas the other nodes can be on a cloud provider like AWS.

# Setting up:

Ensure the file is in the right folder and that you are in the right directory on your terminal,

run: `Vagrant up`, we can check if it has ran successfully by running `Vagrant status` or by checking the VirtualBox app:

![Alt text](images/Screenshot%202023-06-05%20142412.png)

# step 1: SSH in

run: 
```
vagrant ssh controller
```
to get into the controller node. 

update and upgrade it:
```
sudo apt update -y
sudo apt upgrade -y
```

once in we can connect to the different nodes, i.e: the web and db. 

# step 2: update the nodes

we need to go in and update them:
run the following to get into the web node:
```
ssh vagrant@192.168.33.10
```
the password is `vagrant` and is invisible.
run:
```
sudo apt update -y 
```
to update. 
upgrade aswell:
```
sudo apt upgrade -y
```

run `exit` to leave web.

Now update the db node: go in:
```
ssh vagrant@192.168.33.11
```
as you can see we are in the `db`. 
![Alt text](images/Screenshot%202023-06-05%20152426.png)

update: 
```
sudo apt update -y
```
upgrade aswell:
```
sudo apt upgrade -y
```


![Alt text](images/Screenshot%202023-06-05%20150044.png)

update and upgrade the controller again.

### Check hosts

go into each vm to check if it asks you to add to the known hosts. 

# step 3: install ansible

naviagte to the controller in the terminal and run:
this installs common pacakages
```
sudo apt install software-properties-common
```
Add the repository:
```
sudo apt-add-repository ppa:ansible/ansible
```
now update and install:
```
sudo apt update -y
sudo apt install ansible -y
```

![Alt text](images/Screenshot%202023-06-05%20150533.png)

check the version:
```
sudo ansible --version
```

![Alt text](images/Screenshot%202023-06-05%20150612.png)

# step 4: install tree

naviagte to the correct directory:
```
/etc/ansible
```
should looklike:

![Alt text](images/Screenshot%202023-06-05%20150743.png)

install the tree:
```
sudo apt install tree -y
```

we can run Just `"Tree"` to see it in a different layout:

![Alt text](images/Screenshot%202023-06-05%20150902.png)

# Step 5: change hosts file

We can check connectivity between hosts/ nodes throgh using the `ping` command, if there is a successful connection established, it should return with `pong`. To do this follow:

once in the correct directory (`/etc/ansible`), we can run the following to get into the hosts file to modify it:
```
sudo nano hosts
```

add the lines:
```
[web]
192.168.33.10 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant
```
shown in:

![Alt text](images/Screenshot%202023-06-05%20145739.png)

save it and run 
```
sudo ansible web -m ping
```
with the output:

![Alt text](images/Screenshot%202023-06-05%20151232.png)

PONG!

## db

to get a PONG from the db we need to add a line to the hosts file:
```
[db]
192.168.33.11 ansible_connection=ssh ansible_ssh_user=vagrant ansible_ssh_pass=vagrant
```

![Alt text](images/Screenshot%202023-06-06%20124306.png)

### NOTE
if it does not work, we need to go into the ansible.cfg file and change something:
Look for this and uncomment it:
![Alt text](images/Screenshot%202023-06-06%20124454.png)

Also, copy the line and place is under `ssh`:

![Alt text](images/Screenshot%202023-06-06%20124605.png)

# adhoc commands

we can use these commands to find data about our nodes, some examples are:

`sudo ansible db -a "date"` - shows date
`sudo ansible db -a "free -m"` - shows free memory
`sudo ansible all -a "la -a"`  - shows all files from all nodes
`sudo ansible db -m copy -a "src=/etc/ansible/test1.txt dest=/home/vagrant` - copies file from a source to a destination

(more can be found here)[https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html]

## Creating playbook

This playbook will be used to launch nginx.

make sure you are in the correct location `/etc/ansible`. then to create a file use:
```
sudo nano config_nginx_web.yml
```

here is my file:
```
# this is a playbook to install nginx in the web-server
# indentation matters have 2  spaces
# add 3 dashes "---" this starts the yaml file.
---
# add name of host 
- hosts: web


# optional:
# gather additional facts about setup:
  gather_facts: yes


# add admin access to this file (stops us from having to add sudo to every line)
  become: true

# add instructions/TASKS to add and install nginx
  tasks:
  - name: Installing Nginx
    apt: pkg=nginx state=present

# install nginx and enable nginx - ensure status is running.
```

to run the file, ensure your in the correct directory and run:
```
sudo ansible-playbook config_nginx_web.yml
```
![Alt text](images/Screenshot%202023-06-06%20133524.png)

![Alt text](images/Screenshot%202023-06-06%20133614.png)

## Playbook for nodejs


Update all vms via:
```
sudo apt update -y 
sudo apt upgrade -y
```
web: `ssh vagrant@192.168.33.10`
db: `ssh vagrant@192.168.33.11`
and controller: `vagrant ssh controller` or `exit`.

we can check by connecting to the controller.

in order to copy over the app folder

run:
```
sudo apt update
sudo apt install git
sudo git clone https://github.com/MasumA009/app.git 
```

now to copy the file into the correct directory:
```
sudo ansible web -m copy -a "src=/etc/ansible/app dest=/home/vagrant"
```

now create a new file and input the code:
```
sudo nano db_playbook.yml
```


```
- hosts: web
  gather_facts: yes
  become: true
  tasks:
    - name: Installing Nginx
      apt: 
        pkg: nginx 
        state: present

    - name: Add Node.js PPA
      shell: "curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -"
      args:
        warn: no

    - name: Install Node.js
      apt:
        name: nodejs
        update_cache: yes
        state: present

    - name: Install npm
      apt:
        name: npm
        state: present

    - name: Install npm packages
      command:
        cmd: npm install
        chdir: /home/vagrant/app
      become_user: vagrant

    - name: Start the npm application
      shell:
        cmd: nohup npm start > /dev/null 2>&1 &
        chdir: /home/vagrant/app
      become_user: vagrant

    - name: Restart nginx service
      service:
        name: nginx
        state: restarted


```
run it through:
```
sudo ansible-playbook db_playbook.yml
```

![Alt text](images/Screenshot%202023-06-06%20174618.png)


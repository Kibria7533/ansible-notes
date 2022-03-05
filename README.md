
# Ansible
Ansible is a open-source IT Congfiguration Management,Development and Orchestration Tool.
It aims to provided large productivity gains to wide varicty of automation challengers.

## How to Establish ssh connection between Server and Node
Take acces of all machines Windows (vi putty) and linux terminal

Now go inside ansible server and download ansible package
```sh
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```
Now do "ls"
```sh
yum install epel-release-latest-7.noarch.rpm -y
```
```sh
yum update -y
```
Now we have to install all packages are by one
```sh
yum install git python python-level python-pip openssl ansible -y
```
Now got to hosts file inside ansible server and paste privet ip of nodes1 & node2
```sh
vi /etc/ansible/hosts
```
Now this hosts file is any working after updating ansible cfg file
```sh
vi /etc/ansible/ansible.cfg
```
## uncommented
#inventory  = /etc/ansible/hosts
#sudo-user  = root

Now create one user ,in all the these instances
```sh
adduser ansible
```
Now set password for this user

```sh
passwd ansible
````

Now Switch as ansible user
```sh
su - ansible
```
go to  root user 

Sudo priviledges to ansible User
```sh
visudo
```
Now go to inside this file
visudo file write....
```sh
root ALL = (ALL)    ALL
ansible ALL=(ALL)   NOPASSWD: ALL
```
Now go to ansible server and then to install httpd package as a ansible user
```sh
sudo yum install httpd -y
```

Now estabilish connection betwwen server node and go to ansible server
[ansible@ip]#
```sh
ssh  <node privet ip>
``` 
OUTPUT : permission denied

Now we have to do some changes in sshd_config file go to ansible server
Now go to root user
```sh
vi /etc/ssh/sshd_config
```
## uncomment and comment
```sh
PermitRootLogin yes
PasswordAuthentication yes
#PasswordAuthentication no

```
```sh
service sshd restart
```


Do this work in Node1 & Node2 also Now verify in ansible server
[ansible@ip]#
```sh
su - ansible
ssh  <node privet ip>
``` 

Now go to ansible server and create keys Rule this command as ansible user.
[ansible@ip]#
```sh
ssh-keygen
```
hidden file show 
```sh
ls -a
```
```sh
cd .ssh/
ls
```
O/P -----> id_rsa_pub
Now I need to copy public key in bot nodes 
[ansible@ip]# 
```sh
ssh-copy-id ansible@<node1 privet ip>
ssh-copy-id ansible@<node2 privet ip>
```
## Host Pattern
"all" pattern refers to all the machine in an inventary

```sh
ansible all --list-hosts
ansible demo --list-hosts
ansible demo[0] --list-hosts
ansible demo[0:1] --list-hosts
ansible demo[2] --list-hosts
```
Group sepralid by a colon can be used to use hosts from multiple groups

demo[1:3]:devops[2:8]
# Ad-hoc command
demo----->group
-a-------->argument
-b-------->become
Got to ansible server 
```sh
ansible demo -a "ls"
```
```sh
ansible demo -a "ls al"
```
```sh
ansible demo -a "sudo yum install httpd -y"
```
```sh
ansible demo -ba "sudo yum install httpd -y"
```
```sh
ansible demo -ba "sudo yum remove httpd -y"
```
# Ansible Module
Go to ansible server
Note:
-b = become
-m = module
-a = argument
pkg = package install
demo = group name
state:
install =present
uninstall = absent
pakge = latest
```sh
ansible demo -b -m yum -a "pkg=httpd state=present"
```
```sh
ansible demo -b -m yum -a "pkg=httpd state=latest"
```
yum module
```sh
ansible demo -b -m yum -a "pkg=httpd state=absent"
```
service module
```sh
ansible demo -b -m service -a "name=httpd state=started"
```
user module
```sh
ansible demo -b -m user -a "name=rafiul"
```
copy module
```sh
ansible demo -b -m copy -a "src=file4 dest=/temp"
```
# Ansible module Idenpotency is present
go to ansible server
```sh
ansible demo -m setup
```
```sh
ansible demo -m setup -a "filter=*ipv4*"
```

# Ansible Playbook
Go to ansible server 
Now create one playbook
```sh
vi target.yml
```
playbook name = target.yml
```sh
--- # Target playbook
- hosts: demo
  user: ansible
  become: true
  connection: ssh
  gather_facts: yes

```
```sh
ansible-playbook target.yml
```
# Task
```sh
vi task.yml
```
```sh
--- # Target playbook
- hosts: demo
  user: ansible
  become: true
  connection: ssh
  tasks:
          - name: Install HTTPD On linus
            action: yum name=httpd state=installed                                        
```
```sh
ansible-playbook task.yml
```
# Variable
go to ansbile server
```sh
vi vars.yml
```
```sh
--- # Varibale  playbook
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  vars:
          pkgname: httpd
          tasks:
                  - name: Install HTTPD On linux server
                    action: yum name='{{pkgname}}' state=installed


```
Create one playbook
```sh
ansible-playbook vars.yml
```
# Handlers 
go to ansible server
```sh
vi handlers.yml
```
```sh
--- # Target Handlers
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  tasks:
                - name: install HTTPD on the linux
                  action: yum name=httpd state=present
                  notify: restart httpd
  handlers:
                - name: restart httpd
                  action: service name=httpd state=restared

```
```sh
ansible-playbook handlers.yml
ansible-playbook handlers.yml --check
```
# Loops
Go to ansible server 
```sh
vi loops.yml
```
```sh
--- # My Loops Playbook
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  tasks:
                - name: install HTTPD on the linux
                  user: name='{{item}}' state=present
                  with_items:
                          - rafiul
                          - bappy
                          - mehedi
                          - maruf

```
```sh
ansible-playbook loops.yml
```
Check to the user Node1 % Node2
```sh
cat /etc/passwd
```
# Condition
```sh
vi condition.yml
```
```sh
--- # My Condition
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  tasks:
                - name: Install apache server for debian family
                  command: apt-get -y install apache2
                  when: ansible_os_family == "Debian"
                - name: install apache server for redhet family
                  command: yum -y install httpd
                  when: ansible_os_family == "RedHat"
```

# Vault
Creating a new encrypted playbook
```sh
ansible-vault create vault.yml
```
Edit the encrypted playbook
```sh
ansible-vault edit vault.yml
```
To chnage the password
```sh
ansible-vault rekey vault.yml
```
To encrypt on existing playbook
```sh
ansible-vault encrypt target.yml
```
To decrypted on encrypted playbook
```sh
ansible-vault decrypt target.yml
```
# Roles
Tree package install.......
```sh
sudo yum install tree -y
```
```sh
mkdir -p playbook/roles/webserver/tasks
tree
```
```sh
cd playbook/
```
 ```sh
 ``
 touch roles/webserver/tasks/main.yml
```
```sh
vi roles/webserver/tasks/main.yml
```
```sh
- name: install apache on RedHet
  yum: pkg=httpd state=latest
```
```sh
master.yml
```
```sh
--- # Master playbook
- hosts: demo
  user: ansible
  become: yes
  connection: ssh
  roles:
    - webserver                              
```
```sh
ansible-playbook master.yml
```










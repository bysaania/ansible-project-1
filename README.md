# My First Ansible Project — Linux Server Setup Automation

Automating the setup of a Linux web server using Ansible.  
Instead of manually SSHing into a server and typing commands one by one,  
this playbook does it all automatically in under 2 minutes.

**What it does:**
- Updates all system packages
- Installs Nginx (a web server)
- Creates a custom welcome page
- Configures a firewall (UFW)
- Creates a deploy user with SSH access
- Starts and enables Nginx on boot

**Tech stack:** Ansible · Ubuntu Linux · AWS EC2 (free tier)

---


## Project Structure

```
ansible_project_1/
├── README.md                    ← You are here
├── ansible.cfg                  ← Ansible configuration
├── inventory/
│   └── hosts.yml                ← List of servers to configure
├── playbooks/
│   └── setup.yml                ← The main "recipe" (what to do)
└── roles/
    └── webserver/               ← Reusable tasks, broken into a "role"
        ├── tasks/
        │   └── main.yml         ← The actual steps
        ├── handlers/
        │   └── main.yml         ← Actions triggered by changes
        └── templates/
            └── index.html.j2    ← HTML template for the web page
```

---

## Step-by-Step Setup Guide

### PART 1 — Set Up Your Free AWS Server (20 minutes)

You need a Linux server to run Ansible against.  
We'll use AWS Free Tier — no credit card charges if you follow these steps.

#### 1.1 Create an AWS account

#### 1.2 Launch a free EC2 instance (your Linux server)

Launch the instance:

1.AWS Console → EC2 → Launch Instance
2.Name: ansible-project-1
3.AMI: Ubuntu Server 24.04 LTS (Free tier eligible)
4.Instance type: t3.micro (Free tier eligible)
5.Key pair: Create New Key-Pair 
     - Name it: `ansible-key`
     - Type: RSA
     - Format: `.pem`
     - Click "Create key pair" — this downloads `ansible-key.pem` to your computer
     - **Save this file — you cannot download it again**
6.Network settings:
     - Allow SSH traffic from: Anywhere
     - Allow HTTP traffic from the internet
     - Security Group: Create new -> ansible-project-1-sg   
7. Click **"Launch Instance"**
8. Wait ~2 minutes → click "View Instances"
9. Copy the **Public IPv4 address** <— you'll need this


#### 1.3 Move your key file to a safe place
# On your Mac/Linux laptop, open Terminal:
mkdir -p ~/.ssh
mv ~/Downloads/ansible-key.pem ~/.ssh/
chmod 400 ~/.ssh/ansible-key.pem   # This makes the key file secure (required)


#### 1.4 Test that you can SSH in (confirm the server works)
ssh -i ~/.ssh/ansible-key.pem ubuntu@YOUR_SERVER_IP



---


### PART 3 — Download and Configure This Project (5 minutes)

#### 3.1 Clone this repo
git clone https://github.com/bysaania/ansible_project_1.git
cd ansible_project_1


#### 3.2 Tell Ansible about your server
Open `inventory/hosts.yml` and replace `YOUR_SERVER_IP` with your actual AWS IP:

inventory/hosts.yml — edit this file
all:
  hosts:
    webserver:
      ansible_host: 54.123.45.67   # ← Your AWS IP goes here


#### 3.3 Tell Ansible where your SSH key is
Open `ansible.cfg` and replace the path if needed:
private_key_file = ~/.ssh/ansible-key.pem


#### 3.4 Test Ansible can reach your server
ansible all -m ping -i inventory/hosts.yml

* Expected output:
* webserver | SUCCESS => {
*     "ping": "pong"
* }

If you see `pong`, Ansible can talk to your server. 

---

### PART 4 — Run the Playbook (2 minutes)

##### Always do a dry run first (--check means "show me what WOULD happen, don't actually do it")
ansible-playbook playbooks/setup.yml --check

##### Sample output from my lab

-----------
ansible-playbook playbooks/setup.yml --check                                            

PLAY [Setup and configure webserver] ****************************************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************************************************************************************************************************
[WARNING]: Host 'webserver' is using the discovered Python interpreter at '/usr/bin/python3.14', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.21/reference_appendices/interpreter_discovery.html for more information.
ok: [webserver]

TASK [webserver : update the apt package cache] *****************************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : upgrade the OS packages] **********************************************************************************************************************************************************************************************************************************************************************************
ok: [webserver]

TASK [webserver : Install all software packages] ****************************************************************************************************************************************************************************************************************************************************************************
changed: [webserver] => (item=nginx)
ok: [webserver] => (item=ufw)
ok: [webserver] => (item=curl)
ok: [webserver] => (item=git)

TASK [webserver : Create new user and add to the sudo group] ****************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : Allow SSH through firewall] *******************************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : Allow HTTP through firewall] ******************************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : Enable firewall with default deny policy] *****************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : Deploy a custom web page using a template] ****************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : Delete the default nginx page] ****************************************************************************************************************************************************************************************************************************************************************************
ok: [webserver]

TASK [webserver : Start and Enable Nginx server] ****************************************************************************************************************************************************************************************************************************************************************************
[ERROR]: Task failed: Module failed: Could not find the requested service nginx: host
Origin: ~./Projects/ansible_project_1/roles/webserver/tasks/main.yml:125:3

123 # ---- 6. Start and Enable Nginx Service --------------------
124
125 - name: Start and Enable Nginx server
      ^ column 3

fatal: [webserver]: FAILED! => {"changed": false, "msg": "Could not find the requested service nginx: host"}

PLAY RECAP ******************************************************************************************************************************************************************************************************************************************************************************************************************
webserver                  : ok=10   changed=7    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0   



-----------


** Note: This last error is happening because you're running in --check mode
--check mode means "don't actually do anything"
       ↓
Nginx never actually gets installed (just simulated)
       ↓
Start Nginx task runs and looks for nginx service
       ↓
Nginx doesn't exist because it was never actually installed
       ↓
ERROR: Could not find nginx service










##### If it looks good, run it for real
ansible-playbook playbooks/setup.yml


Before running the playbook, you can verify the status of the AWS EC2 instance
From your Mac terminal — open a second terminal window:
ssh -i ~/.ssh/ansible-key.pem ubuntu@YOUR_EC2_IP 

##### Once inside the server run these one by one:
*Check Nginx is installed and running:
systemctl status nginx


* Check your custom web page is there:
cat /var/www/html/index.html

* Check if the 'deploy' user exists:
id deploy

* Check firewall is active:
sudo ufw status

* Check Nginx responds locally:
curl http://localhost

* From your browser Open:
http://YOUR_EC2_IP

##### Sample output from my AWS EC2 Instance  (before running the playbook)

$ systemctl status nginx
Unit nginx.service could not be found.

ubuntu@ip-172-31-23-172:~$ cat /var/www/html/index.html
cat: /var/www/html/index.html: No such file or directory

ubuntu@ip-172-31-23-172:~$ id deploy
id: 'deploy': no such user

ubuntu@ip-172-31-23-172:~$ sudo ufw status
Status: inactive

ubuntu@ip-172-31-23-172:~$ curl http://localhost
curl: (7) Failed to connect to localhost port 80 after 0 ms: Could not connect to server



Now run the playbook and verify all the above commands again. 
You should see different results from before and after running the playbook.



##### Sample screen output of working ansible playbook from my lab

-------------------------
% ansible-playbook playbooks/setup.yml

PLAY [Setup and configure webserver] ****************************************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************************************************************************************************************************
[WARNING]: Host 'webserver' is using the discovered Python interpreter at '/usr/bin/python3.14', but future installation of another Python interpreter could cause a different interpreter to be discovered. See https://docs.ansible.com/ansible-core/2.21/reference_appendices/interpreter_discovery.html for more information.
ok: [webserver]

TASK [webserver : update the apt package cache] *****************************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : upgrade the OS packages] **********************************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : Install all software packages] ****************************************************************************************************************************************************************************************************************************************************************************
changed: [webserver] => (item=nginx)
ok: [webserver] => (item=ufw)
ok: [webserver] => (item=curl)
ok: [webserver] => (item=git)

TASK [webserver : Create new user and add to the sudo group] ****************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : Allow SSH through firewall] *******************************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : Allow HTTP through firewall] ******************************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : Enable firewall with default deny policy] *****************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : Deploy a custom web page using a template] ****************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : Delete the default nginx page] ****************************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

TASK [webserver : Start and Enable Nginx server] ****************************************************************************************************************************************************************************************************************************************************************************
ok: [webserver]

TASK [webserver : Check if nginx is working] ********************************************************************************************************************************************************************************************************************************************************************************
ok: [webserver]

TASK [webserver : Print the verification when successful] *******************************************************************************************************************************************************************************************************************************************************************
ok: [webserver] => {
    "msg": "Nginx webserver is Up and running at http://52.91.50.135"
}

TASK [webserver : Print when it fails] **************************************************************************************************************************************************************************************************************************************************************************************
skipping: [webserver]

RUNNING HANDLER [webserver : Restart Nginx] *********************************************************************************************************************************************************************************************************************************************************************************
changed: [webserver]

PLAY RECAP ******************************************************************************************************************************************************************************************************************************************************************************************************************
webserver                  : ok=14   changed=10   unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   





-------------------------


Run those commands from your EC2 Instance and compare the results after the successful playbook run
##### Sample output from my lab

ubuntu@ip-172-31-23-172:~$ systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-06-19 01:16:23 UTC; 4min 46s ago


ubuntu@ip-172-31-23-172:~$ cat /var/www/html/index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Ansible Server</title>
    <style>


ubuntu@ip-172-31-23-172:~$ id deploy
uid=1001(deploy) gid=1001(deploy) groups=1001(deploy),27(sudo)
ubuntu@ip-172-31-23-172:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
22/tcp (v6)                ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             

ubuntu@ip-172-31-23-172:~$ curl http://localhost
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First Ansible Server</title>





#### 4.1 See your web server live
Open a browser and go to: `http://YOUR_SERVER_IP`

You should see your custom welcome page!

---

### PART 5 — Run It Again (This is the Magic)

ansible-playbook playbooks/setup.yml

Run it a second time. Notice:

PLAY RECAP
webserver : ok=8  changed=0  failed=0

`changed=0` — Ansible checked everything and said "already done, nothing to change."  
This is called **idempotency** — a core concept in infrastructure automation.  
Your server is always in the exact state your code describes.

---

### PART 6 — Push to GitHub

git add .
git commit -m "Working Ansible playbook 1 — automated Nginx setup on Ubuntu EC2"
git push

---

###  Remember: Terminate Your AWS Instance When Done. 
### Stopping alone won't help, you will still incur costs for EBS Volumes
### I learn't it the hard way : P


To avoid any charges, stop your EC2 instance when you're not using it:
1. AWS Console → EC2 → Instances
2. Select your instance → Instance State → **Stop**
3. Or **Terminate** if you're done with it completely

---


## Next Steps

- **Project B:** Use Terraform to create this EC2 instance automatically (no clicking in AWS Console)
- **Project C:** Combine Terraform + Ansible — Terraform builds the server, Ansible configures it

---

*Part of my cloud/DevOps learning journey. Built as a beginner — intentionally simple.*  
*[linkedin.com/in/saania-khanna](https://linkedin.com/in/saania-khanna)*
## My First Ansible Project — Linux Server Setup Automation





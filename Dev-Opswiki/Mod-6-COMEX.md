# Step 1 getting your server and agent made with the proper network configuration/storage

## Server 1 

- CPU: 4
- RAM: 8
- Disk: 100 GB
- IP: X.X.X.51
- Hostname: stud<student-number>-s1
- SELinux permissive PERMANENTLY
- Firewalld stopped and disabled
- `/etc/chrony.conf` configured to use the network NTP server
- `/etc/hosts` file configured with **server**, **agent**, and **vip** IPs and hostnames

## Agent 1

- CPU: 4
- RAM: 8
- Disk: 100 GB
- IP: X.X.X.51
- Hostname: stud<student-number>-s1
- SELinux permissive PERMANENTLY
- Firewalld stopped and disabled
- `/etc/chrony.conf` configured to use the network NTP server
- `/etc/hosts` file configured with **server**, **agent**, and **vip** IPs and hostnames

## Cluster variables

- NTP IP: 44.44.44.11
- Domain: `stud<student-number>.soroc.mil`
- VIP IP: 172.16.3.50
- VIP Hostname: vip
- VIP Range: 172.16.3.54-172.16.3.59
- coredns IP: 172.16.3.53
- Upstream DNS: 10.10.44.50 (Use this one for DNS when building VMs)
- Harbor Hostname: harbor.vip.ark1.soroc.mil
- Rancher Hostname: rancher.vip.ark1.soroc.mil

1. On student laptop open git bash and navigate to `.ssh/`
1. Run `ssh-keygen -t rsa -b 2048`
1. Don't enter passphrase, just press `Enter` three times
1. Run `ssh-copyid -i ~/.ssh/id_rsa.pub root@172.16.7.51`
   * Enter password and type `yes` if prompted
1. Run `ssh-copyid -i ~/.ssh/id_rsa.pub root@172.16.7.52`
   * Enter password and type `yes` if prompted
## if you run into issue refer to the building a linux vm notes under RKE2

# Step 2

- scp your tar file with your rke2 automated script to your server node
- unpack and run the script (dont forget to change any variables if something changed)
- if your running into issue's refer to the troubleshooting page on gitlab under RKE2

# Install longhorn
once your server and agent are up and running navigate to rancher and follow the steps below

- after logging in click on the home page and select the tab 'Import Existing'
- click on the 'Generic' method
- enter the name you want the cluster to populate as
- once the cluster is created a new page will open up to the registration tab, there will be 3 options on this page use the second one starting with 'curl --insecure etc..'
- copy that option and paste it onto your server node
- if the terminal says 'kubectl: command not found' run the command 'source ~/.bashrc or source /root/.bashrc' now you will have kubectl commands
- once you paste the curl command from rancher you should see cattle cluster pods being created
- run the following command to ensure that your pods are coming to a ready state 'watch kubectl get pods -A'
- now longhorn should be installed

# Add harbor repository to rancher
- select your cluster
- on the left side of the page, select 'apps'
- under 'apps' select 'repositories'
- select create
- name your repository 'harbor'
- in the url block, add 'https://harbor.vip.ark1.soroc.mil/chartrepo/stud3'
- select create
You will get an error and the fix is below
- to the right of the harbor repo select the 3 dots
- select 'edit yaml'
- add the line 'insecureSkipTLSVerify: true' under 'spec'
- then save that file and it should turn green/active

# Before you install the 'django application under charts'
- run the following command to create the namespace 'kubectl create ns django-system'
- now on rancher you can select your application under charts and install it on that namespace that you created

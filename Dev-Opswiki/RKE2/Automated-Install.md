# Automated install
> Although the following is a little bit beefy, it will walk you through how to automate the process
# Automate Everything

So far, we have only prepared the server downloaded the files needed for the installation of RKE2. These processes take time and you will be doing it many times.  Something which used to take you an hour can take only a few seconds when you have the processing power of a computer doing it for you.

---

## Step 1 - Revert the Server Node to baseline

I the first lab, you created a snapshot of the Server Node VM before anything was changed.  Revert back to that baseline snapshot before continuing with this lab

## Step 2 - Log in to your student laptop

All of the following steps will be completed on your student laptop

## Step 3 - Create the file structure

In Edge-Compute we use a standard file structure for continuity between teams and projects.

We started this file structure when we pulled the dependencies. Now we will add to it.

--- 

Create a directory to hold files you will source as environment variables on your server.

```shell
mkdir -p ~/<student-kubernetes-project>/rke2/scripts/source/
```

This directory will store environment variables for the cluster such as IP addresses, hostnames, functions, or whatever else you want to create a variable for.

Your file structure should look like the following:

```shell
rke2/
    scripts/
        source/
        install/
            rke2-images.linux-amd64.tar.zst
            rke2.linux-amd64.tar.gz
            sha256sum-amd64.txt
            install.sh
```

## Step 4 - Build initiation script

We will be creating an initiation script which will be calling the `main.sh` bash script which will be doing all the work.  This will be more important later on when we want the `main.sh` to run in the background but we want the initiation script to ask for credentials for ssh.

---

Add the following the initiation script:

```shell
vim ~/<student-project>/rke2/scripts/rke2-install.sh
```

Every script you write will begin with the following:

```shell
#Example
#!/bin/env bash

set -o errexit
set -o nounset
#set -o xtrace # Uncomment for verbose output
```

These are best practices for bash scripting.

- `set -o errexit` ensures the script does NOT continue to run when a command fails.
- `set -o nounset` will exit the script if it comes accross and undeclared variable.
- `set -o xtrace` prints the all the standard input to standard out.  this is a good option for troubleshooting but should only be used when needed because the output can get very messy and difficult to read.

We will then ensure the `main.sh` script is set to be executable and run it with the following:

```shell
#These will be added to rke2-install.sh
chmod +x /usr/lib/rke2/scripts/install/main.sh

/usr/lib/rke2/scripts/install/main.sh &
```

The final initiation script will look like the following:

```shell
#!/bin/env bash

set -o errexit
set -o nounset
#set -o xtrace # Uncomment for verbose output

chmod +x /usr/lib/rke2/scripts/install/main.sh

/usr/lib/rke2/scripts/install/main.sh &
```

Save and exit the file.

Make the `rke2-install.sh` executable:

```shell
chmod +x ~/rke2/scripts/rke2-install.sh
```

## Step 5 - Build the `main.sh` script

The `main.sh` script will call the functions used for the preparation of the server and the installation of RKE2.

---

Create the main script and open it for editing:

```shell
vim ~/<student-project>/rke2/scripts/install/main.sh
```

Add the following to the `main.sh` file (notice how comments have been placed before every action):

```shell
#!/bin/env bash

set -o errexit
set -o nounset
# set -o xtrace # Uncomment for verbose output

# Redirects all standard output to the file /usr/lib/rke2/scripts/install.log
exec > /usr/lib/rke2/scripts/install.log 2>&1 

# sets the evironment variables source to all files in the source directory
source <(cat /usr/lib/rke2/scripts/source/*)

# Calls the function to prepare the server for the installation of RKE2
prep_server
```

The function `prep_server` is the function which will be used to prepare the server for the installation of RKE2

## Step 6 - Create the `prep_server` function

We will create a file for each function we call on in the `main.sh`

---

Since all of our installation steps are done on Linux and we are using a bash shell to install RKE2 when we do it manually, most of our commands can be written just as plain as we would normally write our commands

---

1. **Functions**

We have already created a main script which will call the functions we write to files in our source directory.

There are a few ways to write functions in bash. We will use the following template in this course:

```shell
#EXAMPLE
function_name () {
    set -o errexit
    set -o nounset
    # set -o xtrace # Uncomment for verbose output
    <command>
    <command>
}

```

Create a file named `prep_server.sh` and open it for editing.

```shell
vim ~/<student-project>/rke2/scripts/source/prep_server.sh
```

We will not be adding the headers to this file because everything in the source directory is only getting added to environment variables.  We will never actually run this file as an executable and even if we did, nothing would happen because we are only defining the function and not calling on it.

Create a function named `prep_server` at the beginning of the file

***NOTE*** Always leave a space at the end of a file or bash will see the last line as an unexpected termination of the file.

```shell
#Example
prep_server () {
    set -o errexit
    set -o nounset
    # set -o xtrace # Uncomment for verbose output
}

```

2. **Add a command**

Now that we have created the function, we need to go back and look at the commands we ran to prepare the server.

The first thing we did in the previous lab was create an interface configuration for the RKE2 cni by adding the following line to a new file named `/etc/NetworkManager/conf.d/rke2-canal.conf` with the contents:

```shell
#Example
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
```

There are a few ways to automate this in bash, but we will use redirects to tell bash when to begin and terminate the file.

```shell
#EXAMPLE
cat >> /file/path/filename.ext <<EOF
<file contents>
EOF
```

To make bash create this file for us in the script, can write it as follows:

```shell
cat > /etc/NetworkManager/conf.d/rke2-canal.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
EOF
```

This will create the file and add the contents between the two `EOF` statements.

to add this to your function, place it on a line in between the curly braces `{}`

***NOTE*** Be sure to indent every new line in the function or it will not be read with the function and **ALWAYS LEAVE COMMENTS BEFORE A NEW ACTION**

```bash
prep_server () {
    set -o errexit
    set -o nounset
    # set -o xtrace # Uncomment for verbose output
# Create the NetworkManager cni configuration
    cat > /etc/NetworkManager/conf.d/rke2-canal.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
EOF
}
```

There was one more action we had to take to ensure Network Manager applied the config in the previous lab.  We also need to tell the function to reload Network Manager

```bash
prep_server () {
    set -o errexit
    set -o nounset
    # set -o xtrace # Uncomment for verbose output
# Create the NetworkManager cni configuration
    cat > /etc/NetworkManager/conf.d/rke2-canal.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
EOF

# Reload NetworkManager to apply the configuration
    systemctl reload NetworkManager
}
```

3. Adding variables

The next step requires us to create the `hosts` file in the server.  This will contain the IP address and hostname of all servers in the cluster.  Right now it is just the one.

write out a script to create the file for you:

```bash
#EXAMPLE
cat > /etc/hosts <<EOF
127.0.0.1   localhost
<Server1 IP> <Server1 hostname> <Server1 hostname>.<domain>
EOF
```

This will work great for your server, but if you want to use the script for other servers, you would need to come to this line individually and change the IP and hostname.  We will add these as `variables` instead.

Always surround your variables with `${}`.  The `{}` are not neccessary, but prevent parsing errors when bash is reading your variables when using underscores.

```bash
cat > /etc/hosts <<EOF
127.0.0.1   localhost
${server1_ip} ${server1_hostname} ${server1_hostname}.${domain}
EOF
```

If we ran the script right now, it would fail because it does not know the variables so we will create a variables file which will be sourced with the functions.

Create the variables file:

```shell
vim ~/rke2/scripts/source/variables
```

All hostnames and IP addresses will be added as variables to ensure the the installation script can be used on a different kit with different IPs and hostnames

 > ***NOTE:*** All self defined variables will be defined with ALL lowercase letters.  System defined variables will be defined with capital letters.

Add the following to your variables file:

```shell
#EXAMPLE
# Variables for the cluster configuration
server1_ip=<server1-ip>
server1_hostname=<server1-hostname>
domain=stud<number>.soroc.mil
```

We will also add the kubernetes environment variables to this file and it should look like this when you're done

```bash
# Variables for the cluster configuration
server1_ip=<server1-ip>
server1_hostname=<server1-hostname>
domain=soroc.mil

# Environment variables for kubernetes
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
export PATH=$PATH:/var/lib/rancher/rke2/bin:/usr/local/bin
export CRI_CONFIG_FILE=/var/lib/rancher/rke2/agent/etc/crictl.yaml
alias ku=kubectl
alias kuebctl=kubectl
alias k=kubectl
```

4. Set SELinux and firewalld (For training purposes only)

The final step for the preparation of the server will be to overwrite the SELinux file with our permissive setting, set the running SELinux status to permissive and disable firewalld

```bash
cat > /etc/selinux/config <<EOF
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
EOF
```

```bash
setenforce 0
```

```bash
systemctl disable --now firewalld
```

Now you can add it to your prep_server function:

```bash
prep_server () {
    set -o errexit
    set -o nounset
    # set -o xtrace # Uncomment for verbose output
# Create the NetworkManager cni configuration
    cat > /etc/NetworkManager/conf.d/rke2-canal.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
EOF

# Reload NetworkManager to apply the configuration
    systemctl reload NetworkManager

# Overwrite the current hosts file on the server
    cat > /etc/hosts <<EOF
127.0.0.1   localhost
${server1_ip} ${server1_hostname} ${server1_hostname}.${domain}
EOF

# Overwrite the selinux config file
    cat > /etc/selinux/config <<EOF
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
EOF

# Set the running SELinux config
    setenforce 0

# Disable and stop firewalld
    systemctl disable --now firewalld

}
```

For every action the script accomplishes write use the echo command to print a string to the Standard Output and log the completion status.

```bash
prep_server () {
    set -o errexit
    set -o nounset
    # set -o xtrace # Uncomment for verbose output
# Create the NetworkManager cni configuration
    cat > /etc/NetworkManager/conf.d/rke2-canal.conf <<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:flannel*
EOF
    echo "Created NetworkManager cni configuration"

# Reload NetworkManager to apply the configuration
    systemctl reload NetworkManager
    echo "reloaded NetworkManager"

# Overwrite the current hosts file on the server
    cat > /etc/hosts <<EOF
127.0.0.1   localhost
${server1_ip} ${server1_hostname} ${server1_hostname}.${domain}
EOF
    echo "Configured /etc/hosts"

# Overwrite the selinux config file
    cat > /etc/selinux/config <<EOF
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
EOF
    echo "Configured the SELinux config file"

# Set the running SELinux config
    setenforce 0
    echo "Sucessfully set the running selinux config to Permissive"

# Disable and stop firewalld
    systemctl disable --now firewalld
    echo "Successfully stopped and disabled SELinux"

}
```

## Step 7 - Move the installation to the server

Everything which will be used to install of RKE2 will be in the `~/<student-project>/rke2` directory.  We will package this into a compressed archive and move it to our server.

---

We will be turning this rke2 directory into a compressesed archive file using tar.  This gives us the ability to preserve the permissions of the files inside the directory and move it as a single file to the server.

Because we are using windows and archive files are a feature primarily used in linux, we will be creating a ubuntu container in docker and mounting it to our directory on Windows to create the tar file.

***NOTE*** The docker run command used here will need to be run in either **Command Prompt or Powershell** because we will need to use the Windows filepath format to mount the container.

1. Open Command Prompt

2. Navigate to your student project folder

```shell
cd <student-project>
```

3. Find the filepath to your current directory

```shell
echo %cd%
```

4. Run a Ubuntu container

- To find your current directory type:

```shell
pwd
```
- Change to Command Line on Student Laptop to run
```shell
docker run -it -v <current-directory>:/mnt ubuntu:latest
```

- The -v option creates a bind mount at the specified directory

    ```shell
    docker run -it -v <host-directory>:<directory-inside-container> <image-name>:<tag>
    ```

5. Package the rke2 directory into a compressed archive file.

Move to the directory where we mounted our student project

```shell
cd /mnt
```

Create the compressed archive file:

```shell
tar -czvf rke2-stud-<student-number>.tar.gz rke2/
```

6. Exit the container

```shell
exit
```

7. Return to your gitbash terminal
Copy the compressed archive file to your server:

```shell
scp rke2-stud-<student-number>.tar.gz <username>@<server-ip>:/home/<username>/
```

## Step 8 - Unpack the script

You have already moved installation to your server. Now we will unpack it and run it on the server.

#

**ssh into your server node**

You should now see the tar gz file in your /home/<username>/ directory

```bash
cd /home/<username>/
```

List the directory contents:

```shell
ls -al
```

The installation of RKE2 reqiures root access so we will change to the root user to run the script.

```bash
sudo su
```

Unpack the compressed archive file to the `/usr/lib/` directory

```shell
tar xzvf rke2-stud-<student-number>.tar.gz -C /usr/lib/
```

## Step 9 - Run the script

Run the script.  It should already be executable because we did that before we packaged it into the compressed archive file.

```shell
/usr/lib/rke2/scripts/rke2-install.sh
```

## Step 10 - Watch the status

We wrote the script to output all standard out to a log file in the `/usr/lib/rke2/scripts/` directory.  The script will continuously write to this file.

To watch the output, tail the file

```shell
tail -f /usr/lib/rke2/scripts/install.log
```
- If your .sh fails run
```shell
dos2unix *
#Run on each directory with files
```

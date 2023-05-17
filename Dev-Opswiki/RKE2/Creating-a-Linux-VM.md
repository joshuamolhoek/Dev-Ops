# Creating a Linux VM
> Building a Rocky Linux Virtual Machine on an ESXI Host

## In this lab you will cover

- [Creating a Virtual Machine in ESXi](#task-1-creating-a-virtual-machine-in-esxi)
- [Installing Rocky Linux from the UI](#task-2-installing-rocky-linux-from-the-ui)
- [Verifying the installation](#task-3-verifying-the-installation)

### Resources

- See instructional material

### Task 1 - Creating a Virtual Machine in ESXi

#### Step 1 - Log in to your ESXi host

- Open a web browser.
- Navigate to the webpage of the ESXi host you were assigned by the instructors.
- Login to the ESXi host with your student username and password.
- You should be logged into the ESXi host and see it's main page displayed in your web browser.

#### Step 2 - Open the new virtual machine configuration menu

- If the `Navigator` tab is not open, click on the `Navigator` tab on the left side of the ESXi webpage.
- In the `Navigator` tab, click on `Virtual Machines`.
- In the `Virtual Machines` window, click on `Create / Register VM`.
  > A box will pop up to create your virtual machine.

#### Step 3 - Select a creation type

- Click on `Select a creation type`.
- In the `Select a creation type` menu, click on `Create a new virtual machine`.
- Click `Next` in the lower right corner.

#### Step 4 - Configure a name and guest OS

- Enter `<student username>_VM` for the name of the VM.
- In the `Compatibility` drop-down leave the selection the default, which should match the version of ESXi you are using.
- In the `Guest OS family` drop-down, select `Linux`.
- In the `Guest OS version` drop-down, select `Rocky Linux (64-bit)`.
- Click `Next` in the lower right corner.

#### Step 5 - Select your storage

- Select the storage labeled with your student number using the format `ESXIxx_DS`.
- Click `Next` in the lower right corner.

#### Step 6 - Customize hardware

- In the `CPU` drop-down, select `4`.
- In `Memory`, enter `4096 MB`.
- In `Hard disk 1`, enter `50 GB`.
- Expand the Hard disk 1, in the Disk Provisioning section select the radio button for `Thin provisioned`.
- In the `Network Adapter 1` drop-down change the Network adapter 1 to  `VM Network` selected and check the `"Connect"` box.
- In the `CD/DVD Drive 1` drop-down, click `Datastore ISO file`.
- In the `Datastore browser` window, click on `ISOs`, open the Rocky Linux folder, click the .iso file `Rocky-8-6-x86_64-dvd1.iso`, and click `Select` in the bottom right.
- Ensure the `"Connect"` box next to the `Datastore ISO file` is checked.
- Click `Next` in the lower right corner.
- Click `Finish` in the lower right corner.
  > You should be returned to the Virtual Machines window.

### Task 2 - Installing Rocky Linux 8 from the UI

#### Step 1 - Accessing the VM's UI

- In the `Virtual Machines` window right-click on the VM that you just created.
- In the drop-down menu, select `Power` and `Power on`.
  > The VM should power on in a few minutes.
- In the `Virtual Machines` window right-click on the VM that you just created.
- In the drop-down menu, select `Console` and `Open browser console`.
- Select "Install Rocky Linux 8" and press enter.
- On the "WELCOME TO ROCKY LINUX 8.", Select English and click `Continue'.

#### Step 2 - Configure the `Installation Destination`

- In the `Installation Summary` page, click on `Installation Destination`.
- Select `Custom` for Storage Configuration.
- Click the blue `Done` button.
- On the `Manual Partioning` page, click `Click here to create them automatically`.
- We will leave them default for now but read the NOTES below for examples of when you would customize the configuration here.
  > **NOTE** The screen will display what the default storage configuration would be. /, /boot/efi, and /boot are required.
  > **NOTE** on **Security Policies**: If we were applying the DISA STIG Security Policy, Additional mounts would need to be created for the following directories: /tmp, /var, /var/log, /var/tmp, and /var/log/audit.
  > **NOTE** on **Kubernetes**: If you plan on using the device for Kubernetes **Version 1.21 or earlier** the mount for `swap` can be removed here.  A swap partition allows Linux to run infrequently used programs in this partition instead of using RAM.  This feature frees up RAM for traditional server processes but can cause conflicts with certain Kubernetes processes. Kubernetes **release 1.22 introduced support for configuring swap memory usage**.
- Click the blue `Done` button.
- In the `Summary of changes` window, click `Accept Changes`.
  > You should be returned to the `Installation Summary` page.

#### Step 3 - Configure the `Network & Host Name`

- In the `Installation Summary` page, click on `Network & Host Name`.
- Enter `<student username>-VM` as the hostname in the `Host Name` block in the bottom left.
- Select the interface you would like to configure in the column on the left half of the window.
- Select `Configure` in the bottom right.
- Select the `IPv4 Settings` tab.
- In the `IPv4 Settings` tab, change the Method to `Manual`.
- In the `IPv4 Settings` tab, click the `Add button in the`Addresses block`.
- Enter the following data:
  - IPv4 address: 172.16.<student#>.10
  - Network mask: 255.255.255.0
  - Gateway: 172.16.<student#>.254
  - DNS: 10.200.99.30 & 31
  - Leave `Search domains` blank
- Move to the `IPv6 Settings` tab and change the method to `Disabled`.
- Now Click the `Save` button to save the settings.
- Change the switch at the top right from `off` to `on`.
- Click the blue `Done` button.

#### Step 5 - Configure the `Software Selection`

- In the `Installation Summary` page, click on `Software Selection`.
- For your **Base Environment** select `Server`.
- In the section: **Additional software for Selected Environment** Select:
  - Network File System Client
  - Development Tools
  - Headless Management
  - RPM Development Tools
  - System Tools
  > You may not use everything on this list, but at a minimum, having these tools installed should be able to give you everything you need on the server to do dev work on the fly or troubleshoot any sort of issues in an Air-Gapped environment.
- Click the blue `Done` button.

#### Step 6 - Configure the `Security Policy`

- In the `Installation Summary` page, click on `Security Policy`.
- Scroll all the way to the bottom of the list of security Policies, and select the option named **DISA STIG for Red Hat Enterprise Linux 8**.
- Click on `Select Profile`.
  > Pay attention to the errors which show up in the box at the bottom.
- `REMOVE` the security policy by toggling the switch at the top labeled `Apply security policy`, to `OFF`.
  > We will `NOT` be using the DISA STIG for training.
- Click the blue `Done` button.

#### Step 7 - Configure the `Root Password`

- In the `Installation Summary` page, click on `Root Password`.
- Enter the Root password `P@ssw0rd` in the `Root Password` box.
- Re-enter the Root password `P@ssw0rd` in the `Confirm` box.
- Click the blue `Done` button.

#### Step 8 - Configure the `User Creation`

- In the `Installation Summary` page, click on `User Creation`.
- Enter your student username in the user's full name in the `Full name` box.
- Enter your student username in the user's name that will shows in the shell in the `User name` box.
- Check the box: `Make this user administrator`.
- Enter `P@ssw0rd` in the user's password in the `Password` box.
- Re-enter `P@ssw0rd` in the user's password in the `Confirm password` box.
- Click the blue `Done` button.

#### Step 9 - Begin installation

- Click `Begin Installation` in the bottom right corner.
  > Then installation will take about 5-10 minutes.
- When the installation is complete, you will see a prompt to reboot.
- Click the Blue `Reboot System` to reboot the VM.
- When your VM has successfully rebooted, move on to the next Task.

### Task 3 - Verifying the Installation

- In the previous tasks, we configured the server with everthing we needed to run the server.  This task will walk you through where to check the settings you have configured during installation and where to change any of these after the installation.
- The following Steps will also give you a starting point for troubleshooting most of the common issues you will have with the server.

#### Step 1 - Accessing your student VM from the Command prompt

- login to VM by supplying a username and password

```shell
<studentxx-VM>Login:studxx
```

```shell
Password: P@ssw0rd
[studxx@studentxx-VM~]$
```

#### Step 2 - View the Anaconda Kickstart file

The initial configuration settings for the server will all be stored in what is called the `anaconda kickstart` file.

- View the settings you used in the steps above to verify your configuration:

  ```shell
  sudo cat /root/anaconda-ks.cfg
  ```

  > You can also use this file as a baseline for creating your own kickstart file to be used off for a hands-off installation.

#### Step 3 - View the Networking configuration

There are many ways to check your network configuration and troubeshoot if it is not working correctly.

- Display the connection status and IP address:

  ```shell
  ip address
  ```

- Check ARP table for connection to the gateway

  ```shell
  ip neighbor show
  ```

- Ping the gateway or another device on the network

  ```shell
  ping <gateway-ip>
  ```

There are multiple ways to configure your network connection on Rocky Linux 8.  NetworkManager is the default networking service on Rocky Linux 8. The two main methods to configure NetworkManager are `nmcli` and `nmtui`

#### Step 4 - View the Network Manager Text User Interface (`nmtui`)

- Access the `nmtui` (Network Manager Text User Interface):

  ```shell
  sudo nmtui
  ```

- You should see a screen similar to the screen below where you can edit connection settings or the system hostname:

  ```shell
  ┌─┤ NetworkManager TUI ├──┐
  │                         │ 
  │ Please select an option │ 
  │                         │ 
  │ Edit a connection       │ 
  │ Activate a connection   │ 
  │ Set system hostname     │ 
  │                         │ 
  │ Quit                    │ 
  │                         │ 
  │                    <OK> │ 
  │                         │ 
  └─────────────────────────┘
  ```

  > Any time you make a change to NetworkManager, you must restart the NetworkManager service to ensure the changes are configured.

- Restart the NetworkManager service:

  ```shell
  sudo systemctl restart NetworkManager.service
  ```

 > All of these settings can also be changed from the commandline using `nmcli`.

- View the man page for nmcli for detail information or using it to configure the network connections:

  ```shell
  sudo man nmcli
  ```

#### Step 5 - View the Hostname configuration

- View the current hostname by looking at the name to the left of your cursor in the terminal window. This name is stored in the file `/etc/hostname`.

- View the contents of the `/etc/hostname` file:

  ```shell
  sudo cat /etc/hostname
  ```

  > You may change your hostname by editing this file or by changing it with nmtui but you will need to reboot
- Set the hostname using the `hostnamectl` command:

  ```shell
  sudo hostnamectl set-hostname <new-hostname>
  
  hostnamectl
  
  su <username>
  ```

  > Using hostnamectl will eliminate the need for a reboot but you will still need to open a shell session to update your terminal bar.

#### Step 6 - View the Domain Name System (DNS) configuration

- View the DNS settings in the file `/etc/resolv.conf`:

  ```shell
  sudo cat /etc/resolv.conf
  ```

  > You may add or change dns settings by editing this file but it will be overwritten by NetworkManager the next time it restarts or the server is rebooted.
  > Edit the DNS configuration by using `nmtui` or `nmcli` and remember to restart the NetworkManager service.

#### Step 7 - View the drives, partitions, and LVMs configuration

The file **/etc/fstab** will contain the drive mounts which will mount on their own every time the system boots.  If you add new drives to the system and want them to mount automatically when the server boots, you would need to add them here.
> You may also need to remove the line for the `swap` partition if you will be running kubernetes 1.21 or earlier.

- View the drive mounts in the `/etc/fstab` file:

  ```shell
  sudo cat /etc/fstab
  ```

- View the block storage devices, the size of the disk, the type, and the mount point of partitions or LVMs:

  ```shell
  lsblk
  ```

  > The command `lsblk` stands for: `list block devices`.
- Use the `lvdisplay` command to view more detailed information about an LVM:

  ```shell
  sudo lvdisplay
  ```

- Use the `df` (disk free) command will show the total space and available space on all partions:

  ```shell
  df -h
  ```

  > The `-h` (human-readable) will display the sizes in a more readable format.

#### Step 8 - View the installed packages

- Display a list of installed packages on the system:

  ```shell
  yum list installed
  ```

- Use the `grep` command to search for a specific package:

  ```shell
  yum list installed | grep nfs-utils
  ```

  > The grep command will be covered in more detail in a later lesson.

#### Step 9 - Verify client-side Network Time Protocol (NTP)

Timing is important for the accuracy of logs, use of external services such is updates, and distrubuted storage with Kubernetes.

Chronyd is the default tool for NTP on RHEL 8 and is what we will use to set the time on our servers.

- View the NTP sources configured:

  ```shell
  chronyc sources
  ```

  > If you see an asterix `"*"` next to a name in your server, it has synchronized to the server.
- View the time on your server:

  ```shell
  chronyc tracking
  ```

#### Step 10 - Editing the NTP configurations if necessary

- If you do not have any sources and the time is not correct, edit the chrony.conf using the nano editor with sudo privilege:

  ```shell
  sudo nano /etc/chrony.conf
  ```

  > Editing this file changes where your VM looks for the NTP server.
- You should see this at the top of the file:

  ```shell
  # Use public servers from the pool.ntp.org project.
  # Please consider joining the pool (http://www.pool.ntp.org/join.html).
  pool 2.rhel.pool.ntp.org iburst
  ```

- Remove the NTP pool and replace it with our NTP server on the local network:

  ```shell
  # Use public servers from the pool.ntp.org project.
  # Please consider joining the pool (http://www.pool.ntp.org/join.html).
  server 44.44.44.10 iburst
  ```

- Save the file by holding down the control button and pressing the `o` button.
- When prompted `File Name to Write: chrony.conf`, press enter.
- Exit `nano` by holding down the `ctrl` key and pressing the `x` key.
- Restart Chronyd

  ```shell
  systemctl restart chronyd.service
  ```

- Now that the NTP server was changed, view the NTP sources configured:

  ```shell
  chronyc sources
  ```

- Again, view the time on your server:

  ```shell
  chronyc tracking
  ```

#### Step 11 - Upgrade installed software

- Update all packages installed on your system with the following command:

  ```shell
  sudo yum update -y
  ```


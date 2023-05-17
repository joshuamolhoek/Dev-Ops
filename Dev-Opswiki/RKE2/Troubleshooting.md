# Troubleshooting
> All notes for troubleshooting your cluster
# Everything to uninstall when resetting your node

- /usr/local/bin/rke2-killall.sh
- /usr/local/bin/rke2-uninstall.sh
- rm -rf /var/lib/rancher
- rm-rf /run/k3s/
- /usr/local/bin/rke2-uninstall.log

# How to SCP your tar file from your linux server to your windows machine

- use this command to create the tar file 
```
tar -czvf rke2-stud-<name of file>.tar.gz rke2/
```
- you can push that tar file by using the following command
```
scp rke2-stud-<name of file>.tar.gz <username of server>@<serverip>:/home/<username on server>/
```
- you can also use scp to pull the file to your windows machine from the linux server

```
scp <username of server>@<server ip>:/usr/lib/<name of tar file that your created> ~ (to move it to the current directory you are in, or specify the directory instead)
```

# Run the following command to unpack the tar file

```
tar xzvf rke2-stud-<name of file>.tar.gz -C /usr/lib/
```

# You will then run the run that file using the following command
```
/usr/lib/rke2/scripts/rke2-install.sh
```
# Use the following command to follow along with your script
```
journalctl -u rke2-server-service -f
```
# Commands to run after changing your /etc/chrony.conf/ 
```
first cd into /etc/chrony.conf/ and change the Allow NTP client access so it looks like this 'allow 44.44.44.10'

then run 'systemctl stop chronyd.service'
followed by the command 'systemctl start chronyd.service'
and now check the status 'systemctl status chronyd.service'
you can also run 'systemctl restart chronyd.service'
you can see if your NTP server is active by typing the command 'chronyc sources'
```

# if you run the agent with the wrong node token you can change it in the following directory

```
/etc/rancher/rke2/config.yaml on agent

then run the command 'systemctl restart rke2-agent.service'
```
# How to force delete a pod stuck in terminating state

```
kubectl delete pod <name of pod> --grace-period=0 --force --namespace <pods namespace>
```

# Using the following commands to help with kubectl commands
```
kubectl create secret -h <shows all the possible outcomes>
kubectl create namespace test --dry-run=client -o yaml > test-namespace.yaml <puts the information into a test yaml that you can modify>
```
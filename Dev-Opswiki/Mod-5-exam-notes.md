make a ssh key on your remote vm - ssh-keygen -t rsa -b 2048
copy your public key you just made - cat ~/.ssh/id_rsa.pub
copy the outcome and put it in your ssh keys on GitLab
clone the repository down using the ssh link- git clone git@10.10.44.20:DevOps_1-23/Student_Resources.git(wherever it is located)
Step 1
make a ssh key on your remote vm - ssh-keygen -t rsa -b 2048

copy your public key you just made - cat ~/.ssh/id_rsa.pub

copy the outcome and put it in your ssh keys on GitLab

clone the repository down using the ssh link- git clone git@10.10.44.20:DevOps_1-23/Student_Resources.git(wherever it is located)

once that is complete move on to building your API call by inputing the variables

nautobot_token = 'e1c4922e1d4888e08acd76a88bfe1cd9fc41dca1'(found under profile> API keys)
devices_API_URL = 'https://10.10.44.21/api/dcim/devices' (given in the practice)
method = 'GET'
headers = {
    "Accept": "application/json",
    "Content-Type": "application/json",
    "Authorization": f"TOKEN {NAUTOBOT_TOKEN}"
}
parameters = {"site": "orko_mod_5_comex"} (given)

Step 2
input the information that you just inputed in step 1
method=METHOD,
url=DEVICES_API_URL,
headers=HEADERS,
params=PARAMETERS,
verify=False

devices_json = devices.json()

Step 3
add your for loop to the hostvars 

for device in devices_json['results]:
    hostvars['_meta']['hostvars'][device['name']] = {}
    hostvars['_meta']['hostvars'][device['name']]['ansible_host'] = device['primary_ip4']['address'].split('/')[0]
    hostvars['_meta']['hostvars'][device['name']]['device_type'] = device['device_type']['model]
Step 4,5,6
define your groups accordingly

for device_name in hostvars['_meta]['hostvars'].keys():
    if 'RED in device_name:
        groups['red_devices']['hosts'].append(device_name)
    if 'YELLOW' in device_name:
        groups['yellow_devices']['hosts'].append(device_name)
    if 'ROUTER' in device_name:
	groups['routers']['hosts'].append(device_name)
    if 'SWITCH' in device_name:
	groups['switches']['hosts'].append(device_name)
for key in groups.keys():
    if key != 'all':
	groups['all']['children'].append(key)

Step 7
updating your inventory to combine with inventory components into one variable

inventory = {}
inventory.update(hostvars)
inventory.update(groups)

with open("my_output.json", mode='w') as file:
    file.write(json.dumps(inventory))

Step 8
print the inventory in JSON format
print(json.dumps(inventory, indent=4))

Step 9 
test the script using pprint()

Step 10
set the executable permission for all users 
using pwd, ls, and ll cd into the .py file you want to change and do chmod a+x something_inventory.py
then ll to verify that it has the proper executables

once your done with that move into creating the group level variable files
update the all.json file under ], with add the coma after bracket
"ansible_network_os": "ios",
"ansible_user": "orko",
"ansible_ssh_pass": "P@ssw0rd"

create 4 new files under the group_vars directory named
red_devices
{
    "dns_servers": [
        "10.10.20.98",
        "10.10.20.99"
    ]
}

yellow_devices
{
    "dns_servers": [
        "10.10.30.98",
        "10.10.30.99"
    ]
}

routers
{
    "management_interface": "Loopback0" (might change to interface like john said check the config to make sure)
}

switches
{
    "management_interface": "Vlan1000"
}

once all the new group_vars are updated/created go ahead and update the jinja2 templates
base.j2 should look something like
hostname {{ inventory_hostname }}
!
{% include 'dns_servers.j2' %}
!
{% include 'physical_interfaces.j2' %}
!
{% include 'management_interface.j2' %}
!
logging host {{ syslog_server }}
!
line vty 0 4
  logging synchronous
  transport input ssh
  transport output ssh
  no login

dns_servers.j2 should look like
{% for ip_address in dns_servers %}
ip name-server {{ ip_address }}
{% endfor %}

physical_interfaces.j2 should look like 
{% for interface in interfaces %}
interface {{ interface.name }}
  ip address {{interface.address}} {{interface.mask}}
!
{% endfor %}

after your all done with your jinja2 file creation/updating 
build your playbook and name it build.yml
it should look like 
---
- name: Build Device Configuration
  gather_facts: false 
  hosts: all
  connection: network_cli

  tasks:
    - name: Set Fact
      delegate_to: 127.0.0.1
      set_fact:
        start_time: "{{ lookup('pipe', 'date +%Y%m%d%H%M%S') }}"

    - name: New directory 
      delegate_to: 127.0.0.1
      file:
        path: "{{ playbook_dir }}/{{ start_time }}-configs"
        state: directory

    - name: Generate device configuration files
      template:
        src: base.j2
        dest: "{{ playbook_dir }}/{{ start_time }}-configs/{{ inventory_hostname}}.cfg"

once all of this is configured run the command
ansible-playbook -e syslog_server=10.10.10.10 build.yml






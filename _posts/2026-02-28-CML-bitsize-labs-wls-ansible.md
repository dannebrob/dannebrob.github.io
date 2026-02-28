This guide walks you through installing and testing Ansible on WSL2 and a CML instance running on Proxmox, as well as creating a node inside a running lab. Cisco never released an official Ansible integration for CML, but the community‑built collection on Ansible Galaxy works well—and we’ll use it together with the required virl2-client library. In short, you’ll set up Ansible, the CML collection, and virl2-client so you can automate CML using both playbooks and Python scripts. It may sound like a lot, but the steps are straightforward, and I’ll guide you through everything. All files for this lab, including playbooks and Python scripts, are available in the labs/wls folder of the repository: https://github.com/dannebrob/CML-bitsize-labs..

## Installing Ansible on Ubuntu 22.04.5 LTS

# Step 1: Install Ansible and the required libraries
Make sure that you check which version of Ubuntu you are running, this guide will be compatible with the 22.0.4.5 LTS release of Ubuntu. To check the current version installed on WLS run this command

```bash
lsb_release -a

expected output:
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.5 LTS
Release:        22.04
Codename:       jammy
```

Now you can move on and update the system,adding the required repos in the apt installation reposotories and installing ansible.

```bash
sudo apt install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
```

Install the cml collection to be able to automate in with Ansible in CML. You will find the documentation [here](https://galaxy.ansible.com/ui/repo/published/cisco/cml/).

```bash
ansible-galaxy collection install cisco.cml
```

To be able to connect with CML, you need to have VIRL2. 
First install pip and the VIRL2 Client:

```bash
sudo apt install python3-pip -y
pip3 install virl2-client

# check if if virt2-client is installed property
python3 -c "import virl2_client; print('OK')"
```

Now its time to check so the correct versions are installed and that the filepaths are ok:

```bash
which ansible 
ansible --version
which python3
```

Expected output with comments for clarification:
```bash
#which ansible, where the ansible config files are stored
/usr/bin/ansible
#ansible --version, which version of ansible is running an data associated with it
ansible [core 2.17.14]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /root/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.10.12 (main, Jan 26 2026, 14:55:28) [GCC 11.4.0] (/usr/bin/python3)
  jinja version = 3.0.3
  libyaml = True
#which python3, where the python config files are stored
/usr/bin/python3
```
To make it work on my WLS I had to install, since the default python3 on WLS is 3.10, the latest versions of httpx, httpcore and h11 to be able to use the virl2-client library. SSH and paramiko are also required to be able to use ansible with cml, but they should be installed with the virl2-client library.
```
python3 -m pip install --upgrade httpx httpcore h11
```

# Step 2: Test the Ansible to ping the CML server
Now all the packages are upp to date and operational.
So now you, if you don't already have it, have to make a folder where you will store the python scripts, hosts.ini and playbooks, and cd into it. 

```bash
mkdir ansible
cd ansible
```
We will start with a Ansible playbook to test the connection to CML, but before that we will need to create a host ini file and a playbook to be able to test the connection.

Lest start with the hosts.ini file, this is where you will store the information about the devices you want to connect to in CML. In my case its the localhost since I am running the playbooks on WLS, but if you are running it on a different machine you will have to change the ip to the ip of the machine where CML is running.

```ini
# hosts.ini file for ansible with cml
[cml]
localhost ansible_connection=local
```
Now we will create a playbook to test the connection to CML, this playbook will use the cml collection to connect to CML and get the version of CML we are running.

```yaml
---
# playbook to test the connection to CML
- name: Test connection to CML
  hosts: cml
  gather_facts: no

  tasks:
    - name: Ping the CML server
      ansible.builtin.ping:
```

To run the playbook, use the following command:

```bash
ansible-playbook -i hosts.ini test_cml_connection.yml
```
If everything is working correctly, you should see a successful ping response from the CML server in the output of the playbook run. This confirms that Ansible is able to ping the CML server correctly.


# Step 3: Create a node in a running lab with Ansible
Now that we have confirmed that Ansible can connect to CML, we can move on to creating a node in a running lab with Ansible. For this we will need to create a new python file that will create a new node in a running lab.

```python
# create_node.py file to create a new node in a running lab
import urllib3
urllib3.disable_warnings()

from virl2_client import ClientLibrary

CML_URL = "https://YOUR_IP_TO_CML_SERVER"
USERNAME = "YOUR_USERNAME"
PASSWORD = "YOUR_PASSWORD"

LAB_NAME = "Enterprice Network Lab"   # The name of the lab you want to work with, make sure it is running before you run the script
NODE_NAME = "NewRouter1"    # The name of the node you want to create, make sure it is unique in the lab
NODE_DEFINITION = "iol-xe"    # The id of the node. For exampel: iosv, iosvl2, alpine, server, etc.

client = ClientLibrary(
    url=CML_URL,
    username=USERNAME,
    password=PASSWORD,
    ssl_verify=False
)

# Get the list of labs and find the lab we want to work with
labs = client.all_labs()
lab = None
for l in labs:
    if l.title == LAB_NAME:
        lab = l
        break

if lab is None:
    print(f"Labb '{LAB_NAME}' hittades inte.")
    exit(1)

# Create a new node in the lab. Make sure to adjust the x and y coordinates to place the node where you want it in the lab topology.
node = lab.create_node(
    label=NODE_NAME,
    node_definition=NODE_DEFINITION,
    x=100,
    y=100
)

lab.sync()

print(f"Node '{NODE_NAME}' created in lab '{LAB_NAME}'.")
```
Make sure to replace the CML_URL, USERNAME, PASSWORD, LAB_NAME, NODE_NAME and NODE_DEFINITION with the correct values for your lab. You can find the node definitions by looking at the nodes in your lab (Tools->Node and Image definitions -> Node definitions).

Then we will make a new playbook that will run the python script to create a new node in the lab.

```yaml
---
- name: Create a new node in a running lab
hosts: cml
gather_facts: no


tasks:
  - name: Run the create_node.py script
    ansible.builtin.command: python3 create_node.py
    registers: output
    
  - name: Print the output of the script
    ansible.builtin.debug:
      msg: "{{ output.stdout }}"

```
So the last thing is to run the playbook to create a new node in the lab, use the following command:

```bash
ansible-playbook -i hosts.ini create_node.yml
```

If everything is working correctly, you should see a message in the output confirming that the node has been created in the lab. You can then go to the CML web interface and check that the new node has been added to the lab topology.


## Conclusion
In this lab, we have successfully installed Ansible on Ubuntu 22.04.5 LTS, set up the necessary libraries to connect with CML, and created a new node in a running lab using Ansible and a Python script. This demonstrates the power of automation with Ansible and how it can be used to manage and interact with network labs like CML efficiently.
I hope this guide has been helpful in getting you started with Ansible and CML, and that you can now explore more advanced automation tasks in your network labs!

I would love to hear your feedback on this lab and if you have any suggestions for improvements or additional topics you would like to see covered in future labs. Feel free to reach out to me on [LinkedIn](https://www.linkedin.com/in/danielbroback/) or through the [GitHub](https://github.com/dannebrob/CML-bitsize-labs) repository.

Happy automating!
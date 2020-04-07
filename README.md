# LACCHAIN #

--- This network is not active now. If you want to deploy your application in an ethereum-based LACChain network, check our [Besu network](https://github.com/lacchain/besu-network) ---

## NOTES

This work was done by EVERIS and is completely donated to LACChain Consortium.

## References

* This Quorum network uses [IBFT](https://github.com/ethereum/EIPs/issues/650) consensus with validator and regular nodes located around Latin America and the Caribbean. 

* In this installation we will use **Ubuntu 18.04** as the operating system and all the commands related to this operating system. In addition, links of the prerequisites will be placed in case it is required to install in another operating system.

* An important consideration to note is that we will use Ansible, for which the installation is done from a local machine on a remote server. That means that the local machine and the remote server will communicate via ssh.

* The Github of Alastria can be followed as a reference [https://github.com/lacchain/lacchain-node](https://github.com/lacchain/lacchain-node)

## System Requirements

Characteristics of the machine for the nodes of the testnet:

* **CPU**: 2 cores

* **RAM Memory**: 4 GB

* **Hard Disk**: 30 GB SSD

* **Operating System**: Ubuntu 16.04, Ubuntu 18.04, CentOS 7.4 or Red Hat Enterprise Linux 7.4, always 64 bits

It is necessary to enable the following network ports in the machine in which we are going to deploy the node:

* **8443**: TCP - Port to monitor

* **9000**: TCP - Port for communication for Constellation.

* **21000**: TCP/UDP - Port to establish communication between geth processes.

* **22000**: TCP - Port to establish RPC communication. (this port is used for applications that communicate with Lacchain and may be leaked to the Internet)

## Prerequisites

### Install Ansible ###

For this installation we will use Ansible. It is necessary to install Ansible on your **local machine** that will perform the installation of the node on the **remote machine**.

Following the instructions to [install ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) in your local machine.

```
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
```

### Clone Repository ####

To configure and install Quorum and Constellation, you must clone this git repository in your **local machine**.

```
$ git clone https://github.com/lacchain/Lacchain
$ cd lacchain/
```

### Install Python ###

* In order for ansible to work, it is necessary to install Python on the **remote machine** where the node will be installed, for this reason it is necessary to install python 2.7 and python-pip.

* If you need to install python-pip in Redhat use [https://access.redhat.com/solutions/1519803](https://access.redhat.com/solutions/1519803)

```
$ sudo apt-get update
$ sudo apt-get install python2.7
$ sudo apt-get install python-pip
```

## Quorum + Constellation Installation ##

### Creation of a new Node ###

* There are two types of nodes (Validator / Regular) that can be created in the Quorum network.

* After cloning the repository, enter this.

    ```
	$ cd lacchain/
	``` 

* First change the IP located within the **inventory file** by the **public IP** of the remote server where you are creating the new node.

	```
	$ vi inventory
	[test]
	192.168.10.72
	```

* To deploy a **validator node** execute the following command in your **local machine**, without forgetting to set the private key in the option --private-key and the ssh connection user in the -u option:

	```
	$ ansible-playbook -i inventory -e first_node=false --private-key=~/.ssh/id_rsa -u vagrant site-lacchain-validator.yml
	```

* To deploy a **regular node** execute the following command in your **local machine**, without forgetting to set the private key in the option --private-key and the ssh connection user in the -u option:

	```
	$ ansible-playbook -i inventory --private-key=~/.ssh/id_rsa -u vagrant site-lacchain-regular.yml
	```
* When starting the installation, it will request that some data be entered, such as the public IP, geth account password and node name. The name of the node will be the one that will finally appear in the network monitoring tool.

* At the end of the installation, if everything is correct, a GETH service will be created in the case of a **validator node** managed by Systemctl with **stop** status.

* In the case of a **regular node** if everything is correct, a CONSTELLATION service and a GETH service managed by Systemctl will be created with **stop** status.

* Now, it is necessary to configure some files before starting up the node. Please, follow the next steps:

## Docker ##

* If you did not clone the LACCHAIN repository in previous steps, then you need to clone it now.

```
$ git clone https://github.com/lacchain/Lacchain
$ cd lacchain/ 
```

### Docker Node Validator ###

* First, you need to create a folder that will contain the files genesis.json, permissioned-nodes.json and static-nodes.json.
* This folder will be the volume of the container on the host machine.

```
$ mkdir /lacchain/data
```

* Copy the following files from lacchain:

```
$ cp /lacchain/roles/lacchain-validator-node/files/genesis.json /lacchain/data/genesis.json 
$ cp /lacchain/roles/lacchain-validator-node/files/permissioned-nodes_validator.json /lacchain/data/permissioned-nodes.json
$ cp /lacchain/roles/lacchain-validator-node/files/static-nodes.json /lacchain/data/static-nodes.json
```

* Now pull the docker image and run the container, setting your node identity and the folder location that will be the volume 

```
$ docker pull lacchain/validator:1.0.0 
$ docker run -dit -e IDENTITY={YOUR_NODE_IDENTITY} -v {QUORUM_DIR}/lacchain/data:/lacchain/data -p 21000:21000 -p 30303:30303 lacchain:1.0.0
```

### Docker Node Regular ###

* First, you need to create a folder that will contain the configuration files to geth and constellation.
* This folder will be the volume of the container on the host machine.

```
$ mkdir /lacchain/data
```

* Copy the following files from lacchain:

```
$ cp /lacchain/roles/lacchain-regular-node/files/genesis.json /lacchain/data/genesis.json 
$ cp /lacchain/roles/lacchain-regular-node/files/permissioned-nodes_general.json /lacchain/data/permissioned-nodes.json
$ cp /lacchain/docker/regular/configuration.conf /lacchain/configuration.conf
$ cp /lacchain/docker/regular/start-node.sh /lacchain/start-node.sh
```

* Create a password file to generate the constellation keys 

```
$ echo "Passw0rd" > /lacchain/.account_pass
```

* Now pull the docker image and run the container, setting your node identity and the folder location that will be the volume 

```
$ docker pull lacchain/regular:1.0.0 
$ docker run -dit -e IDENTITY={YOUR_NODE_IDENTITY} -v {QUORUM_DIR}/lacchain:/lacchain -p 9000:9000 21000:21000 -p 22000:22000 -p 30303:30303 lacchain/regular:1.0.0
```

## Node Configuration

### Configuring the Quorum node file ###

If the node was created from scratch, then the step _Creating a new node_ modified several files in the local repository, namely:

* If the node was a validator, the following files were modified:
	* [`roles/lacchain-regular-node/files/permissioned-nodes_general.json`](roles/lacchain-regular-node/files/permissioned-nodes_general.json)
	* [`roles/lacchain-validator-node/files/permissioned-nodes_validator.json`](roles/lacchain-validator-node/files/permissioned-nodes_validator.json)
	* [`roles/lacchain-validator-node/files/static-nodes.json`](roles/lacchain-validator-node/files/static-nodes.json)
* If the node was regular, these files were modified instead:
	* [`roles/lacchain-regular-node/files/constellation-nodes.json`](roles/lacchain-regular-node/files/constellation-nodes.json)
	* [`roles/lacchain-validator-node/files/permissioned-nodes_validator.json`](roles/lacchain-validator-node/files/permissioned-nodes_validator.json)

Note that the names of the files refer to the nodes that use them, **not** the nodes that have modified them during their creation.

In addition to these changes, which occur automatically when executing the node creation, there are two other files that must be modified manually, depending on the type of node created, to indicate the contact data of the Administrator of the nodes (name, email, etc): [DIRECTORY_VALIDATOR.md] (DIRECTORY_VALIDATOR.md) or [DIRECTORY_REGULAR.md] (DIRECTORY_REGULAR.md)

### Start up Regular Node ###

Once we have modified these files, you can start up the node with this command in **remote machine**:

```
<remote_machine>$ systemctl start constellation
<remote_machine>$ systemctl start geth
```

### Start up Validator Node ####

On the other hand, if the node is a validator, the rest of the nodes in the network must execute to update permissioned-nodes.json file:

```
$ ansible-playbook -i inventory -e validator=yes -e regular=no --private-key=~/.ssh/id_rsa -u adrian site-lacchain-update.yml
```

after the validator nodes add to the new validator node in their *permissioned-nodes.json* file of nodes allowed, execute in **remote machine** the next command:

```
<remote_machine>$ systemctl start geth
```

Then, the file `~/lacchain/logs/quorum-XXX.log` of the new validator node will have the following error message:
```
ERROR[12-19|12:25:05] Failed to decode message from payload    address=0x59d9F63451811C2c3C287BE40a2206d201DC3BfF err="unauthorized address"
```
This is because the rest of the validators in the network have not yet accepted the node as a validator. To request such acceptance we must take note of the node's address(0x59d9F63451811C2c3C287BE40a2206d201DC3BfF).

### Proposing a new validator node ###

* Once the files have been modified, you must do a **pull request** against **this** repository

* If it's validator node and it has an **unauthorized address**, then you must indicate this **address** in the pull request.

* To include this node as a validator node, the administrators of the **other validator nodes** must use the RPC API to vote if they **agree** with your node becoming a validator node.

```
> istanbul.propose("0x59d9F63451811C2c3C287BE40a2206d201DC3BfF", true);
```
or
```
$ cd /lacchain/data
$ geth --exec 'istanbul.propose("0x59d9F63451811C2c3C287BE40a2206d201DC3BfF",true)' attach geth.ipc
```

Thus, the new node will be raised and synchronized with the network **if and only if** over **50%** of the validator nodes vote in your nodes favor.

> **NEVER MAKE A PROPOSAL WITHOUT FIRST UPDATING THE FILES MENTIONED IN: "Configuring the Quorum node file"**,after updating the files, you **need** to run the command:

```
ansible-playbook -i inventory -e validator=yes -e regular=no --private-key=~/.ssh/id_rsa -u adrian site-lacchain-update.yml
```

> **A VALIDATOR NODE MUST NEVER BE ELIMINATED WITHOUT PROPOSING THE REMOVAL THROUGH A PULL REQUEST SO THAT THE REST OF THE VALIDATING MEMBERS WILL REMOVE THEM FROM THEIR FILES (PERMISSIONED-NODES.JSON, STATIC-NODES.JSON) FIRST AND THEN PROCEEED TO A VOTING ROUND:**

```
> istanbul.propose("0x59d9F63451811C2c3C287BE40a2206d201DC3BfF", false);
```
or
```
$ cd /lacchain/data
$ geth --exec 'istanbul.propose("0x59d9F63451811C2c3C287BE40a2206d201DC3BfF",true)' attach geth.ipc
```
### Node Operation ###

 * Faced with errors in the node, we can choose to perform a restart of the node, for this we must execute the following commands:
```
<remote_machine>$ systemctl restart constellation
<remote_machine>$ systemctl restart geth
```

 * The next statement allows you to back up the node's state. It also makes a backup copy of the keys and the enode of your node. All backup copies are stored in the home directory as `~/lacchain-keysBackup`.
 
```
$ ansible-playbook -i inventory -e validator=true --private-key=~/.ssh/id_rsa -u vagrant site-lacchain-backup.yml 
```

**NOTE**
If we want to generate the node using an enode and the keys of an existing node we must make a backup of the keys
of the old node:

```
$ ansible-playbook -i inventory -e validator=true --private-key=~/.ssh/id_rsa -u vagrant site-lacchain-backup.yml 

```

This will generate the folder ~/lacchain-keysBackup whose contents should be moved to ~/lacchain/data/keys.
The keys of this directory (which has to keep the folder structure of the generated backup) will be the ones used
in the image of the node that we are going to generate.

## Disclaimer test-net ##

The public-permissioned LACChain network offered by LACChain is currently at a test-net stage. The access to this network is described in this GitHub. The LACChain team is currently defining a road-map with both technology and legal requirements to release networks in production. From now on, we will refer to the LACChain test-net as ¨The Network¨.

Any natural or legal person that uses or operates The Network becomes a User. The User agrees to these terms and acknowledges that The Network is at an early stage of development. The User acknowledges and accepts that the use of The Network is entirely at the User’s sole risk and discretion.

To use The Network, The User must be authenticated, guaranteeing that every regular/access node operating The Network is associated to a physical or legal person. Every regular node must indicate the contact information of the natural person that is responsible and accountable for the operation of the regular node, including name and e-mail. 

The User acknowledges and agrees that has an adequate understanding of the blockchain technology and the programming languages. The User also understands the risks associated with the use of The Network, which could present interruptions or malfunctions as it is at a est-net stage.  

The user understands that all the information and materials published, distributed or, otherwise, made available on The Network are provided with no guarantee by the Allies,  Members and Users of the LACChain program. Despite The Allies will struggle to offer robust technology, at this est-net stage they are not responsible nor accountable for the reliability of The Network.

All the content related to The Network is provided on an ‘as is’ and ‘as available’ basis, without any representations or warranties of any kind. All implied terms are excluded to the fullest extent permitted by law. No party involved in, or having contributed to the development of The Network, including but not limited to IDB, IDB lab, everis, ConsenSys, NTT Data, io.builders, LegalBlock and any of their affiliates, directors, employees, contractors, service providers or agents (The Parties Involved) accept any responsibility or liability to Users or any third parties in relation to any materials or information accessed or downloaded via The Network. 

The User acknowledges and agrees that all the Parties Involved are not responsible for any damage to the User’s computer systems, loss of data, or any other loss or damage resulting from the use of The Network.

To the fullest extent permitted by law, in no event shall The Parties Involved have any liability whatsoever to any person for any direct or indirect loss, liability, cost, claim, expense or damage of any kind, whether in contract or in tort, including negligence, or otherwise, arising out of or related to the use of all or part of The Network.

The User understands and accepts that none of the physical and/or legal persons operating a core (validator or boot) node in The Network and, therefore, being part of the consensus protocol, is legally committed to maintain those nodes. When a percentage of these nodes are suddenly turned off, the network may cause iinterruptions of service. The User operating a regular/access node and any third party using the network through The User´s regular/access nodes understands that LACChain will not be either responsible nor accountable for any malfunction or damage caused by the disconnections of the core nodes.

The User will not run any application or solution in The Network when The Network is a necessary component for the either application or the solution. 

The User will not run any application or solution nor register any information or data in The Network that incurs illegal activities or practices that can be considered against the Law.

The User must preserve the data privacy of all the physical and legal persons whose information is directly or indirectly registered in The Network through The User´s regular node.

The User is not allowed to promote the use of The Network for commercial or institutional purposes without including the following disclaimer:

*All the operation and use of the LACChain test-net blockchain network by ________ (The User) is carried out under the entire responsibility and discretion of ________ (The User). Under no circumstance, any responsibility or accountability is extended to any other physical or legal person operating the network, or the community behind LACChain. The LACChain blockchain network is currently at a test-net stage and, therefore, it must not be used for applications or solutions in production, or for monetization purposes. The expected use of the network is for testing and learning purposes, as there could be occasional malfunctioning or interruption of service.
Over the coming months, LACChain will develop and incorporate the legal policies, the governance frameworks and the technology upgrades for the release of the pre main-net and the main-net that will enable the operation in production.*


&nbsp;
&nbsp;

**LICENSE**

<center><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></center>

This work is licensed under a [license](http://creativecommons.org/licenses/by-nc/4.0/)

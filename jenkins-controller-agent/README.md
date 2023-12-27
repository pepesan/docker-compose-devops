# Launch

* docker-compose up -d

# Jenkins URL
http://localhost:8081

# Installation
* docker logs jenkins
* get the secret key and put it in the web installer
```shell
sudo cat jenkins_home/secrets/initialAdminPassword
```
* Install the default plugins suggestions
* Configure the admin account

## Agent configuration

### Create SSH Key
* ssh-keygen -t ed25519 -f id_ed25519
```shell
ssh-keygen -t ed25519 -f id_ed25519
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in id_ed25519
Your public key has been saved in id_ed25519.pub
The key fingerprint is:
SHA256:T1yGiGBEYAI+BXTvxrCETP3MuepURG2sfFOIE12x2QY pepesan@moria
The key's randomart image is:
+--[ED25519 256]--+
|=+=*=o* oE.      |
|+o+oo+ B o=.     |
| = o=o* oo.oo    |
|  o *B o ..o     |
|   . =o S o      |
|    o.   o       |
|   ..     .      |
|  ..             |
|  ..             |
+----[SHA256]-----+

```
* Enter the passphrase if needed
* two file: id_ed25519 (private key) and id_ed25519.pub (public key)
### Create credential in Controller
* Goto Manage Jenkins->Security->Credentials
* Select the System Store
* Select the Domain->Global credentials (unrestricted)
* Add Credentials
* Kind: SSH Username and private key
* id: jenkins-docker-agent
* description: jenkins-docker-agent
* username: jenkins
* Private key-> Enter Directly (checked)
* Key-> Add
* Paste the private file content
* Enter the passphrase id needed
* Push Create
### Change the .env file
* Into the .env file change the variable value
* for example
```.env
JENKINS_AGENT_SSH_PUBKEY="ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIA7Fuq5QayqayvakXPbprwOBanqvhlhZye8LjU2dDtqS pepesan@moria"
```

### Load the .env file
```shell
source .env
```
### Reload the agent container
```shell
docker compose restart jenkins_agent
```
### Create a new node
* Into the jenkins controller goto Manage Jenkins->Nodes
* Push New Node
* Choose a node name: jenkins-agent-docker
* Type->Permanent Agent (checked)
* Push Create
* Number of Executors: 1
* RemoteRootDirectory: /home/jenkins/agent
* Labels: linux docker # Space separated values, Can be useful to restrict jobs to run on a particular agent
* Usage: Use this node as much as possible
* Launch Method: Launch agents via SSH
* Host: jenkins_agent
* Credentials: Select the Credential created in previous Step
* HostKeyVerificationStrategy: Non verifying Verification Strategy
* Launch Method > Advanced 
* Port: 22
* ConnectionTimeoutInSeconds: 60 
* MaximumNumberOfRetries: 10 
* SecondsToWaitBetweenRetries: 15
* Avalilability: Keep this agent online as much as possible
* Push Save
* Change the jenkins_ssh_agent permissions
```shell
sudo chmod -R 777 jenkins_ssh_agent
```
* Check in the Node log if all goes right

### Change the controller node configuration
* Goto Manage Jenkins->System Configuration->Nodes
* Select Main Node
* Configuration
* Number of executors: 0
* Labels: none
* Use: Only binded jobs
### Check a new job
* In a job try to execute the build on the new node
## Reference
* [Jenkins controller-agent (master-slave) setup in 10 minutes using Docker](https://dev.to/ashiqursuperfly/jenkins-controller-agent-master-slave-setup-in-10-minutes-using-docker-2a78)
* [Docker agent ssh image](https://hub.docker.com/r/jenkins/inbound-agent/)




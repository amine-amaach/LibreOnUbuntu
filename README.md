# ⚙️ Setting up the Libre Technologies Platform on Ubuntu 22.04

This repository will allow you to spin up the Libre Technologies Platform on Ubuntu 22.04 local machine or development server.

>To successfully install the platform, please follow the steps below.
## 📒 Table of Contents
- [Step 1 - Installing Docker](#Step1)
- [Step 2 - Installing Docker Compose](#Step2)
- [Step 3 - Installing the Libre Technologies Platform](#Step3)
- [Demo](#demo)

<span id="Step1"></span>
## 🪜 Step 1 - Installing Docker
The Docker installation package available in the official Ubuntu repository may not be the latest version. To ensure we get the latest version, we’ll install Docker from the official Docker repository. To do that, we’ll add a new package source, add the GPG key from Docker to ensure the downloads are valid, and then install the package.

First, update your existing list of packages:
```sh
sudo apt update
```
Next, install a few prerequisite packages which let apt use packages over HTTPS:
```sh
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```
Then add the GPG key for the official Docker repository to your system:
```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
Add the Docker repository to APT sources:
```sh
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
```
This will also update our package database with the Docker packages from the newly added repo.
Make sure you are about to install from the Docker repo instead of the default Ubuntu repo:
```sh
apt-cache policy docker-ce
```
You’ll see output like this, although the version number for Docker may be different:
```
docker-ce:
  Installed: (none)
  Candidate: 5:20.10.17~3-0~ubuntu-jammy
  Version table:
     5:20.10.17~3-0~ubuntu-jammy 500
        500 https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
```
Notice that `docker-ce` is not installed, but the candidate for installation is from the Docker repository for Ubuntu 22.04 (`jammy`).
Finally, install Docker:
```sh
sudo apt install docker-ce
```
Docker should now be installed, the daemon started, and the process enabled to start on boot. Check that it’s running:
```sh
sudo systemctl status docker
```
The output should be similar to the following, showing that the service is active and running:
```
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2022-08-04 16:20:02 +01; 23h ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 1448 (dockerd)
      Tasks: 31
     Memory: 121.8M
        CPU: 5min 19.842s
     CGroup: /system.slice/docker.service
             └─1448 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```
By default, the `docker` command can only be run by the root user or by a user in the docker group, which is automatically created during Docker’s installation process.
If you want to avoid typing `sudo` whenever you run the docker command, add your username to the docker group:
```sh
sudo usermod -aG docker ${USER}
```
To apply the new group membership, log out of the server and back in, or type the following:
```sh
su - ${USER}
```
<span id="Step2"></span>
## 🪜 Step 2 - Installing Docker Compose
As mentioned in the [official docker docs](https://docs.docker.com/compose/#compose-v2-and-the-new-docker-compose-command), we'll use `docker compose` instead of `docker-compose` which is _already installed_ with `docker-ce`:
```
● Important
The new Compose V2, which supports the compose command as part of the Docker CLI, is now available.
Compose V2 integrates compose functions into the Docker platform, continuing to support most of the previous docker-compose features and flags. You can run Compose V2 by replacing the hyphen (-) with a space, using docker compose, instead of docker-compose.
```
<span id="Step3"></span>
## 🪜 Step 3 - Installing the Libre Technologies Platform
- **First, clone this repository to your preferred location:** :
```sh
git clone https://github.com/amine-amaach/LibreOnUbuntu.git && cd LibreOnUbuntu
```
- **Give the Grafana container access to the configs** :
```sh
sudo chmod 777 -R ./grafana
```
- **Run the following command to start the platform** :  
```sh
docker compose up -d
```
> For the first time, this command will take some time to run because it first needs to pull the images.

- **_The way I recommand to run the Platform is the following :_**
    * Run these services first : `docker compose up -d mqtt influxdb alpha zero`
    * Then : `docker compose up -d`

Check if the containers are running :
```sh
docker compose ps
```
> You should see that all services are running, except the `dgraph-init` service which is only used to load data into dgraph and then exists.

To check the logs, run the following command :
```sh
docker compose logs -f --tail 50 #50 lines from the end of the logs for each service.
```
To get the logs of a specific service, add the service name as follows :
```sh
docker compose logs -f --tail 50 mqtt #You can add multiple services seperated by space.
```
To stop the services, run the following command :
```sh
docker compose stop #To remove the services, replace `stop` by `down`.
```
 🔗 **Connecting to a server on the host machine :** : 
  By default, the service `libre-edge-agent` is connecting to a remote OPCUA server provided by PROSYS.
 ```
   "PlcConnectorOPCUA" : {
    "ENDPOINT": "opc.tcp://UADEMO.prosysopc.com:53530",
    "aliasSystem": "REMOTE-OPCUA",
    "OVERRIDE_ENDPOINTURL": "opc.tcp://UADEMO.prosysopc.com:53530/OPCUA/SimulationServer"
  }
 ``` 
 Config file : `LibreOnUbuntu/libre-edge-agent/config/config.json`
 
If you want to connect to an OPCUA server on the host machine (Kepware for example), change the host name of `ENDPOINT` and `OVERRIDE_ENDPOINTURL` in the config file to `host.docker.internal` instead of `localhost` or `127.0.0.1` :
```
"PlcConnectorOPCUA" : {
    "ENDPOINT": "opc.tcp://host.docker.internal:49320",
    "aliasSystem": "LOCAL-OPCUA",
    "OVERRIDE_ENDPOINTURL": "opc.tcp://host.docker.internal:49320"
  }
```


<span id="demo"></span>
### 👀 Demo
To see how to access these services, check out this demo on [YouTube](https://youtu.be/lmzXMiQELoo?t=275).

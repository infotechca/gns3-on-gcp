# Install GNS3 Server on Google Cloud (**_KVM Enabled VM_**)

**Tested on Debian10 and Ubuntu 20**

## _Steups to do_

- [x] 1. Prepare your Google Cloud environment
- [x] 2. Enabling nested virtualization on an instance
  - [x] 2.1 Create a boot disk from a public image
  - [x] 2.2 Create a custom image with special license key for virtualization
  - [x] 2.3 Create a VM instance using the new custom image with the license
  - [x] 2.4 Create a Firewall ruel to allow GNS3 on port **`3080`**
  - [x] 2.5 Confirm that nested virtualization is enabled in the VM
- [x] 3. Prepare your instance for GNS3-server Installtion
  - [x] 3.0 Update, Upgrade and Reboot your instance
  - [x] 3.1 uBridge - Download, Compile and Install
  - [x] 3.2 Dynamips - Download, Compile and Install
  - [x] 3.3 VPCS - Download and Install
  - [x] 3.4 QEMU and NAT (libvirt) - Install
  - [x] 3.5 Docker - Install
  - [x] 3.6 Installing i386-libraries for IOU -(IOS on UNIX)
  - [x] 3.7 Create a user for GNS3Server - User: **`gns3`** Pass: **`gns3`**
- [x] 4. Install GNS3 Server
  - [x] 4.1 Set GNS3 server as a daemon (auto start at boot time)
- [x] 5. Testing GNS3Server
- [x] 6. Installing terminal based system monitoring tool
---
## 1. Prepare your Google Cloud environment
Before you begin Set you project default region and zone.<br/>
First, click the Activate Cloud Shell button at the top right of the Google Cloud Console.<br/>
<br/>
Find you Goolge Cloud Projects by typing below command.
```bash
gcloud projects list
```
To set the default project for all gcloud commands, run the command: change ~**`itca-2020`**~ with your **`project_id`**
```bash
gcloud config set project itca-2020
```
initialize the Google Cloud
```bash
gcloud init
```
---
## 2. Enabling nested virtualization on an instance
**Restrictions**
- Nested virtualization can only be enabled for L1 VMs running on **Haswell processors** or later.
- E2 machine types do not support nested virtualization.
- Nested virtualization is supported only for KVM-based hypervisors running on Linux instances.
  - Hyper-V, ESX, and Xen hypervisors are not supported.
- Windows VMs do not support nested virtualization; that is, host VMs must run a Linux OS.
---
### 2.1 Create a boot disk from a public image
There are two steps required to used nested virtualization:<br/>
- The VM instances for which you want to use nested virtualization must use a custom image with a special license key.<br/>
- To enable nested virtualization on a VM instance, create a custom image with a special license key that enables VMX in VM.<br/>
- Create a boot disk from a public image or from a custom image with an operating system.<br/>
```bash
gcloud compute disks create disk1 --image-project debian-cloud --image-family debian-10 --zone asia-southeast1-b
```
### 2.2 Create a custom image with a special license key for virtualization
Using the boot disk that you created, create a custom image with the special license key required for virtualization.<br/>
change zone ~**`asia-southeast1-b`**~ to your zone _example:_ **`us-east1-b`**
```bash
gcloud compute images create kvm-image \
  --source-disk disk1 --source-disk-zone asia-southeast1-b \
  --licenses "https://www.googleapis.com/compute/v1/projects/vm-options/global/licenses/enable-vmx"
```
Note: After you create the image with the necessary license, you can delete the source disk if you no longer need it.<br/>
### 2.3 Create a VM instance using the new custom image with the license
You must create the instance in a zone that supports the **Haswell** CPU Platform or newer.<br/>
Check your zone for Haswell CPU support
```bash
gcloud compute zones describe asia-southeast1-b
```
Create a VM instance using the new custom image with the license<br/>
Minimum Alternatives:
- ~**`Intel Skylake`**~ to **`Intel Haswell`**<br/>
- ~**`n1-standard-2`**~ to **`n1-standard-1`**<br/>
- ~**`pd-ssd`**~ to **`pd-standard`**<br/>

```bash
gcloud compute instances create gns3server --zone asia-southeast1-b \
              --min-cpu-platform "Intel Skylake" --machine-type=n1-standard-2 \
              --boot-disk-size=30GB --boot-disk-type=pd-ssd \
              --tags http-server,https-server \
              --image kvm-image
```
### 2.4 Create a Firewall rule to allow GNS3 on port **`3080`**
You must have to allow TCP port `3080` in the GCP firewall to access GNS3 Server.
```bash
gcloud compute firewall-rules create gns3 --action=ALLOW --rules=tcp:3080
```
### 2.5 Confirm that nested virtualization is enabled in the VM
Connect to your newly created instance with SSH.
```bash
gcloud compute ssh gns3server
```
Find Out CPU Supports for **Intel VT** Virtualization For KVM
```bash
grep -cw vmx /proc/cpuinfo
```
If the output of the above command is Greater-than zero then we have Virtualization technology enabled on our system.<br/>

---
## 3. Prepare your instance for GNS3-server Installation

- uBridge is required, it interconnects the nodes.<br/>
- Dynamips is required for running IOS routers (using real IOS images) as well as the internal switches and hubs.<br/>
- VPCS is recommended, it is a builtin node simulating a very simple computer to perform connectivity tests using ping, traceroute.<br/>
- Qemu is strongly recommended on Linux, as most node types are based on Qemu, for example, Cisco IOSv and Arista vEOS.<br/>
- libvirt is recommended (Linux only), as it's needed for the NAT cloud<br/>
- Docker is optional (Linux only), some nodes are based on Docker.<br/>
---
### 3.0 Update, Upgrade and Reboot your instance
```bash
sudo apt update && sudo apt upgrade -y && sudo reboot
```
### 3.1 uBridge - Download, Compile and Install
**Dependencies**
For Ubuntu or other Debian based Linux you need to install this package:
- libpcap-dev
```bash
sudo apt install make gcc libpcap-dev git wget -y
```
uBridge - Download, Compile and Install
```bash
cd ~
git clone https://github.com/GNS3/ubridge.git
cd ubridge/
make
sudo make install
cd ~

```
### 3.2 Dynamips - Download, Compile and Install
Dynamips now uses the CMake build system. To compile Dynamips you will need CMake and a working GCC or Clang compiler, as well as the build dependencies.

**Build Dependencies**<br/>
On Debian based systems the following build dependencies are required and can be installed using apt:

- libelf-dev
- libpcap0.8-dev
```bash
sudo apt install cmake libelf-dev libpcap0.8-dev -y
```
Dynamips - Download, Compile and Install
```bash
cd ~
git clone git://github.com/GNS3/dynamips.git
cd dynamips/
mkdir build
cd build/
cmake ..
cmake .. -DDYNAMIPS_CODE=stable  -DCMAKE_C_COMPILER=/usr/bin/gcc
make
sudo make install
cd ~

```

### 3.3 VPCS - Download and Install

```bash

cd ~
wget https://github.com/GNS3/vpcs/releases/download/v0.6.1/vpcs
chmod +x vpcs
sudo cp vpcs /usr/bin/vpcs
cd ~

```

### 3.4 QEMU and NAT (libvirt) - Install

```bash
sudo apt install qemu-kvm qemu-system-x86 cpulimit ovmf uml-utilities bridge-utils virtinst libvirt-daemon-system libvirt-clients -y
```

### 3.5 Docker - Install
```bash
sudo apt install docker.io -y
```

### 3.6 Installing i386-libraries for IOU -(IOS on UNIX)
**Dependencies:**
- libc 
- libcrypto

First add i386 architecture support then update your system and install requirements.

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install libssl1.1:i386 -y
cd ~
```

Installing license key to run Cisco IOU on syste.<br/>
Make sure you are login with gns3 user
```bash
sudo bash -c 'printf "\0\0\0\0" > /etc/hostid'
cd ~
wget https://github.com/infotechca/gns3-on-gcp/blob/main/scripts/ciscoIOUKeygen_Python3.py
chmod +x ciscoIOUKeygen_Python3.py
python3 ./ciscoIOUKeygen_Python3.py
cat < ~/iourc.txt > ~/.iourc
sudo bash -c 'printf "\n127.0.0.127\txml.cisco.com  # Added by Me\n" >> /etc/hosts'
cat /etc/hosts

```
### 3.7 Create a user for GNS3Server - User: **`gns3`** Pass: **`gns3`**
```bash
sudo adduser gns3

sudo adduser gns3 sudo
sudo adduser gns3 kvm
sudo adduser gns3 docker

```
---
##  4. Install GNS3 Server
**Dependencies:**
```bash
sudo apt install python3-setuptools python3-pip python3-aiohttp python3-psutil python3-jsonschema -y
```
Finally Download and Install GNS3 Server.
```bash
cd ~
git clone https://github.com/GNS3/gns3-server.git
cd gns3-server
sudo python3 setup.py install
cd init
sudo cp gns3.service.systemd /lib/systemd/system/gns3.service
sudo chown root /lib/systemd/system/gns3.service
cd ~

```
### 4.1 Set GNS3 server as a daemon (auto start at boot time)
```bash
sudo systemctl enable gns3
sudo systemctl enable docker
sudo virsh net-autostart default
```
It is better to reboot your instance at this time.
```bash
sudo reboot
```
---
## 5. Testing GNS3Server
login with gns3 user:
```bash
sudo su gns3
cd ~
sudo systemctl status gns3

```
Download and Extract the GNS3 Sample Project for testing
```bash
cd ~
wget https://archive.org/download/gns3-on-gcp/GNS3.tar.gz
tar -xf GNS3.tar.gz -C ~/GNS3/
wget https://github.com/infotechca/gns3-on-gcp/blob/main/conf/gns3_controller.conf
mv ~/.config/GNS3/2.*/gns3_controller.conf
sudo systemctl restart gns3
ls ~

```
Find your instance Public IP and connect to it using browser<br/>
_example:_ http://0.0.0.0:3080

 ## 6. Installing terminal based system monitoring tool
 Install gotop for system monitoring.
 ```bash
 cd ~
wget https://github.com/xxxserxxx/gotop/releases/download/v4.0.1/gotop_v4.0.1_linux_amd64.tgz
tar -xf gotop_v4.0.1_linux_amd64.tgz
sudo mv gotop /usr/bin/
rm gotop_v4.0.1_linux_amd64.tgz
sudo gotop

 ```
---
▀▄▀▄▀▄ [ Follow us on ] ▄▀▄▀▄▀<br/>
Website:    https://www.infotechca.com<br/>
YouTube:    https://youtube.com/infotechca<br/>
Twitter:    https://twitter.com/infotechca<br/>
Facebook:   https://www.facebook.com/infotechca.hyd<br/>
Instagram:  https://www.instagram.com/infotechca<br/>
Pinterest:  https://pinterest.com/infotechca<br/>
Github:     https://github.com/infotechca<br/>

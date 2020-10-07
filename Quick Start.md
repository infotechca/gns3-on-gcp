# GNS3 Server Quick Start
<br/>
```bash
sudo apt update && sudo apt upgrade -y && sudo reboot
```
<br/>
<br/>
<br/>
```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install make gcc libpcap-dev git wget cmake libelf-dev libpcap0.8-dev qemu-kvm qemu-system-x86 cpulimit ovmf uml-utilities bridge-utils virtinst libvirt-daemon-system libvirt-clients docker.io libssl1.1:i386 python3-setuptools python3-pip python3-aiohttp python3-psutil python3-jsonschema -y
cd ~
git clone https://github.com/GNS3/ubridge.git
cd ubridge/
make
sudo make install
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
wget https://github.com/GNS3/vpcs/releases/download/v0.6.1/vpcs
chmod +x vpcs
sudo cp vpcs /usr/bin/vpcs
cd ~
:
```
<br/>
<br/>
<br/>
```bash
sudo adduser gns3
```
<br/>
<br/>
<br/>
```bash
sudo adduser gns3 sudo
sudo adduser gns3 kvm
sudo adduser gns3 docker
cd ~
git clone https://github.com/GNS3/gns3-server.git
cd gns3-server
sudo python3 setup.py install
cd init
sudo cp gns3.service.systemd /lib/systemd/system/gns3.service
sudo chown root /lib/systemd/system/gns3.service
sudo systemctl enable gns3
sudo systemctl enable docker
sudo virsh net-autostart default
sudo reboot
:
```
<br/>
<br/>
<br/>
```bash
sudo su gns3
sudo systemctl status gns3
:
```
<br/>
<br/>
<br/>
```bash
wget https://archive.org/download/gns3-on-gcp/GNS3.tar.gz
tar -xf GNS3.tar.gz -C ~/
cp -r ~/home/gns3/GNS3/* ~/GNS3/
rm -rf ~/home/
mkdir ~/gns3_config_backup/
cp ~/.config/GNS3/2.*/* ~/gns3_config_backup/
wget https://archive.org/download/gns3-on-gcp/gns3_controller.conf
mv gns3_controller.conf ~/.config/GNS3/2.*/
sudo bash -c 'printf "\0\0\0\0" > /etc/hostid'
cd ~
wget https://archive.org/download/gns3-on-gcp/ciscoIOUKeygen_Python3.py
chmod +x ciscoIOUKeygen_Python3.py
python3 ./ciscoIOUKeygen_Python3.py
cat < ~/iourc.txt > ~/.iourc
sudo bash -c 'printf "\n127.0.0.127\txml.cisco.com  # Added by Me\n" >> /etc/hosts'
cd ~
wget https://github.com/xxxserxxx/gotop/releases/download/v4.0.1/gotop_v4.0.1_linux_amd64.tgz
tar -xf gotop_v4.0.1_linux_amd64.tgz
sudo mv gotop /usr/bin/
rm gotop_v4.0.1_linux_amd64.tgz
sudo systemctl restart gns3
sudo gotop
:
```

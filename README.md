**Keep calm and build an SSI cluster**

Team:
- Krish Jain
- Marcus
- Thomas
- Isa

The IP addresses with corresponding MAC addresses - check MAC address by "ip addr" in terminal and then see it next to wlan... (for wifi) or eth.. (for ethernet)

![image](https://user-images.githubusercontent.com/75043245/167693745-a349a76a-73e1-4756-9962-7deabe60f717.png)

![image](https://user-images.githubusercontent.com/75043245/167693757-d0b0a638-a1c2-4dd0-baae-dc91c1d52283.png)


Main computer password: qKaT%*&6R41111


Hostname formats

nodes (hostname, username & password):
- rpi-node1 (labeled physically as 1)
- rpi-node2 (labeled physically as 3)
- rpi-node3 (labeled physically as M)
- rpi-node4 (yes, dragonboard so not really, but for uniformity)

To learn more about Linux:

- https://www.youtube.com/watch?v=wBp0Rb-ZJak
- https://www.youtube.com/watch?v=ZtqBQ68cfJc
- https://www.youtube.com/watch?v=ROjZy1WbCIA

To learn more about ARM and x86 machine architectures:

- https://www.arm.com/architecture
- https://en.wikipedia.org/wiki/ARM_architecture_family
- https://www.techopedia.com/definition/5334/x86-architecture
- https://en.wikipedia.org/wiki/X86
- https://stackoverflow.com/questions/14794460/how-does-the-arm-architecture-differ-from-x86
- https://en.wikipedia.org/wiki/64-bit_computing

How does git work? Version control for code
- https://rogerdudler.github.io/git-guide/

To learn more about the academic research project our project is powered by:
- https://popcornlinux-doc.readthedocs.io/en/latest/build_kernel.html 
- http://www.popcornlinux.org/index.php/overview
- https://github.com/ssrg-vt/popcorn-kernel
- (Papers) http://www.popcornlinux.org/index.php/publications


To learn more about Linux kernel development
- Learn the fundamentals of the C programming language - https://www.tutorialspoint.com/cprogramming/index.htm
- https://www.youtube.com/watch?v=_JQAve05o_0
- https://www.youtube.com/watch?v=juGNPLdjLH4
- https://www.oreilly.com/library/view/linux-kernel-development/9780768696974/ (Book)
- https://www.cs.utexas.edu/~rossbach/cs380p/papers/ulk3.pdf (Book)
- https://www.youtube.com/watch?v=598Xe7OsPuU

Thread migration:
- https://github.com/ssrg-vt/popcorn-kernel/wiki/Compiler-Setup#initiate-thread-migration
- https://github.com/ssrg-vt/popcorn-kernel/wiki/Compiler-Setup#build-your-own-applications


To learn more about raspberry pis:
- https://www.raspberrypi.com/documentation/computers/linux_kernel.html
The specs we would have in total on all our machines:

```
            .-/+oossssoo+/-.               aloha@makers
        `:+ssssssssssssssssss+:`           -----------
      -+ssssssssssssssssssyyssss+-         OS: Ubuntu 20.04.4 LTS x86_64/ARM
    .ossssssssssssssssssdMMMNysssso.       Host: SSI Cluster
   /ssssssssssshdmmNNmmyNMMMMhssssss/      Kernel: 5.2.21-popcorn
  +ssssssssshmydMMMMMMMNddddyssssssss+     Uptime: 10 hours
 /sssssssshNMMMyhhyyyyhmNMMMNhssssssss/    Shell: bash 5.0.17
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   Resolution: 1920x1080
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   Terminal: /dev/pts/0
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   CPU: Very powerful/benchmark it (20 cores)
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   Memory: 906MiB / 17GB
.ssssssssdMMMNhsssssssssshNMMMdssssssss.    
 /sssssssshNMMMyhhyyyyhdNMMMNhssssssss/    
  +sssssssssdmydMMMMMMMMddddyssssssss+
   /ssssssssssshdmNNNNmyNMMMMhssssss/
    .ossssssssssssssssssdMMMNysssso.
      -+sssssssssssssssssyyyssss+-
        `:+ssssssssssssssssss+:`
            .-/+oossssoo+/-.




```


<details> 
<summary>Click to expand setup details - this stuff has already been done </summary>
            
Setup the wpa_supplicants.conf file using USB hub

- https://www.raspberrypi-spy.co.uk/2017/04/manually-setting-up-pi-wifi-using-wpa_supplicant-conf/
- https://linuxhint.com/rasperberry_pi_wifi_wpa_supplicant/
- https://www.raspberrypi.com/documentation/computers/configuration.html
For the x86 ("normal" Intel/AMD machine in the cluster):


- Grab a desktop ISO image from https://releases.ubuntu.com/20.04/ 
- Use Balena Etcher or the dd command on Linux/mac to flash the ISO to a USB stick
- Plug the USB Stick on your x86 machine and enter the boot menu on the PC (to find the function key for that just Google "pc model + enter boot menu") and go through the installation process
- Remove the USB Stick and reboot
- Update the system


```
sudo apt-get update && sudo apt-get upgrade && sudo apt install vim

```
- Set a static IP address (use nmcli or your distro's equivalent tool - avoid netplan)

```
sudo nmcli  [list interfaces/network interface cards]
sudo nmcli con  [list the available connections]
sudo nmcli con mod "NAME_OF_DESIRED_CONNECTION (in our case wifi (not infiniband :( ) name - STUDENTS)" ipv4.gateway 10.40.7.1 ipv4.addresses staticipaddress/24 ipv4.dns "8.8.8.8 8.8.4.4"
sudo nmcli con up "NAME_OF_DESIRED_CONNECTION" && sudo systemctl restart NetworkManager

```
- Building popcorn dependencies install (refer https://github.com/ssrg-vt/popcorn-kernel/wiki/Kernel-debugging-tips-settings)

```
sudo apt-get install quilt dkms make gcc coreutils pciutils grep perl procps lsof python-libxml2 libssl-dev libncursesw5-dev bison flex git libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev scurl bc build-essential libelf-dev 

```
Downgrade gcc to gcc v8 in order to get popcorn to build 


```
sudo apt-get install gcc-8
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-8 10
```
Install the popcorn enabled kernel (could take long to compile depending on how performant your machine is) 

```
git clone --depth=1 -b main --single-branch https://github.com/ssrg-vt/popcorn-kernel.git
cd popcorn-kernel
cp /boot/config-...... (closest to 5.2) .config
chmod +x ./update_config.sh
./update_config.sh
```
Edit the .config file using your favourite text editor and make sure that the following options are set as follows:

CONFIG_ARCH_SUPPORTS_POPCORN=yes

```
  make -j12 bindeb-pkg LOCALVERSION=-popcorn (going to ask you some popcorn questions, answer them correctly)
  sudo update-grub
  sudo update-initramfs -u -k all
```
Go up one directory:

```
cd ..
ls (you should see popcorn named .deb files)
sudo apt install ./*.deb (going to create a repo to host the .deb files https://github.com/Krish-sysadmin/kernel_deb)
```
Edit grub like 

```
sudo vim /etc/default/grub

```

Make sure the first 3 lines are like 

```
GRUB_DEFAULT=0
#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=-1
```

Now reboot your system to boot the popcorn kernel. When rebooting you'll see a "GNU Grub" menu. Select advanced options and select the popcorn named kernel and hit ENTER. Now you have a popcorn enabled kernel in your system.


To test whether popcorn works 


```
sudo mkdir /etc/popcorn
sudo touch /etc/popcorn/nodes
sudo vim /etc/popcorn/nodes

```

In the "nodes" file the IPs should be listed as 

x86's IP address
...... others

....... others


Now 

```
sudo modprobe msg_socket 
```

If you get no error, you're good to go!


For each target arm64 machine (in our case the Raspberry Pi 4Bs):

- Get raspberry pi Linux kernel using the rpi-5.2.y branch and applying the popcorn kernel patch there. (the latest popcorn kernel was based on 5.2.21)
- wrt creating the patch: can first find the base Linux 5.2.21 code commit: https://github.com/ssrg-vt/popcorn-kernel/commit/e91ef5bcdeda8956eb9f1972ed90198b698dca0f
- Then, git diff and git apply to create a patch and patch the RPI-Linux code
- Might need to have me fix errors when apply patch since the raspberry kernel might update the same files as popcorn


```
git clone -b rpi-5.2.y --single-branch https://github.com/raspberrypi/linux
git clone https://github.com/ssrg-vt/popcorn-kernel
cd popcorn-kernel
git diff e91ef5bcdeda8956eb9f1972ed90198b698dca0f main  > popcorn-rpi.patch
cp popcorn-rpi.patch ../linux/
cd ../linux
git apply ./popcorn-rpi.patch

sudo apt install -y gcc-aarch64-linux-gnu
```

Now follow the instructions here: https://github.com/Krish-sysadmin/kernel_deb and do that. 

On each of these booted into popcorn enabled pis 

```
sudo mkdir /etc/popcorn
sudo touch /etc/popcorn/nodes
sudo vim /etc/popcorn/nodes

```

In the "nodes" file the IPs should be listed as 

x86's IP address
...... others

....... others

Edit grub like 

```
sudo vim /etc/default/grub

```

Make sure the first 3 lines are like 

```
GRUB_DEFAULT=0
#GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=-1
```
``sudo update-grub2``
``sudo reboot``

If grub file does not exist then just email/message me. We can't afford to mess with the initramfs, bootloader or kernel



Compiler setup:

- Download https://drive.google.com/file/d/1oyFep7igjRaJvl3nx7BRREphcF8Y1dvD/view?usp=drive_open

```
sudo apt-get install build-essential flex bison subversion cmake zip x86_64-linux-gnu-g++
git clone -b main --single-branch https://github.com/ssrg-vt/popcorn-compiler.git
cd popcorn-compiler
ls
sudo mkdir -p /usr/local/popcorn
id 
sudo chown <ID OUTPUTTED BY id command> /usr/local/popcorn
./install_compiler.py --install-all --threads 8
```
The compiler, including supporting libraries and tools, is now installed at <POPCORN PATH>. You can use the the provided Makefile in "popcorn-compiler/util/Makefile.pyalign.template" to build progrms. The makefile template expects that the entire application's source is contained in a single folder.

            
Run applications finally
            - https://popcornlinux-doc.readthedocs.io/en/latest/run_applications.html
            
            
            

Command issues you might face:
- Segfault in the Migration Library Due to Executable Stack
- https://github.com/ssrg-vt/popcorn-kernel/wiki/Procedure-calls-in-migration-point


Benchmarking:
- NPB

```
tar -xzf npb......
cd npb....
ls
make A (If you have problems in the linking stage, you probably need to update the Makefile to use the new x86_64-popcorn-linux-gnu-ld.gold instead of ld.gold.)
make
cd ep (or any other)
scp ep_aarch64 ep_x86-64 popcorn@x86
scp ep_aarch64 ep_aarch64 popcorn@armmachinepis
```
On each node

```
cp ep_aarch64 ep 
cp ep_x86-64 ep

```
Now on the main x86  machine do ./ep after chmod +x ./ep (making it executable)

</details>

Contact me:

- Email me at krish.jain@disroot.org
            
**Things Thomas must do:**

Get ethernet hardware (hub and cables)
            
List of ethernet mac addresses: (ignore first part, that's just the interface name)
            
            
            - enp2s0 - f8:bc:12:8c:fc:7a 
            
            - eth0 - e4:5f:01:4a:11:68
            
            - eth0 - e4:5f:01:4a:11:5c
            
            - eth0 - e4:5f:01:4a:11:17
            
How to connect to Pis from main computer? - SSH = CONNECT
            
            
            - Example: ssh rpi-node1@rpi-node1.local
            
            
            - the nodes are rpi-node1, rpi-node2, rpi-node3
            
            
            RPI means raspberry pi

           
Get a list of IP addresses (fixed, static) for each MAC address!
            
                       
For each computer
            - run the command - ip addr
            
            - whatever MAC address you see for that rpi-node1 or whatever the IP address is the corresponding one
            
Now for each computer including the master to assign the static ip address:
            
            - for the master - unplug the wifi card (says COMFAST) and return it to Mr Tallifer. Easiest is to click Wifi logo and Edit connections and delete the active wifi connection
            
            - for the raspberry pis 
            
            - First, sudo nano /etc/dhcpcd.conf and deal with these lines 

 
interface eth0
            
static ip_address=theipaddress/24
            
static routers=therouteraddress
            
static domain_name_servers=8.8.8.8
            

            
Notes:
            
theipaddress = the ip address you were given
            
            
therouteraddress = the router IP address for the ethernet thing which the Network Manager knows            
   
            - then in the terminal/command interface: do sudo systemctl stop wpa_supplicant && sudo systemctl disable wpa_supplicant

Whenever something says permission denied use sudo.

            - now in /etc/popcorn/nodes for each computer after SSHing do sudo nano /etc/popcorn/nodes
            
            - For each computer in that file put in line seperated lists of all the IP addresses of the computers! IN THE SAME ORDER

            
            
            
GOOD LUCK!
            

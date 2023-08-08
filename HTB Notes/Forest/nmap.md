
Nmap 7.93 scan initiated Tue Jun  6 10:28:17 2023 as: nmap -sV -sS -oN nmap/DEFAULT.nmap 10.10.10.161
Nmap scan report for 10.10.10.161
Host is up (0.22s latency).
Not shown: 989 closed tcp ports (reset)
PORT     STATE SERVICE      VERSION
53/tcp   open  domain       Simple DNS Plus
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2023-06-06 14:37:35Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Nmap done at Tue Jun  6 10:30:57 2023 -- 1 IP address (1 host up) scanned in 160.08 seconds

Nmap 7.93 scan initiated Tue Jun  6 10:41:59 2023 as: nmap -p 389 --script ldap* -oN nmap/LDAPSCRIPTS.nmap 10.10.10.161
Nmap scan report for 10.10.10.161
Host is up (0.30s latency).

PORT    STATE SERVICE
389/tcp open  ldap

|_Result limited to 20 objects (see ldap.maxobjects)
| ldap-brute: 
|   root:<empty> => Valid credentials
|   admin:<empty> => Valid credentials
|   administrator:<empty> => Valid credentials
|   webadmin:<empty> => Valid credentials
|   sysadmin:<empty> => Valid credentials
|   netadmin:<empty> => Valid credentials
|   guest:<empty> => Valid credentials
|   user:<empty> => Valid credentials
|   web:<empty> => Valid credentials
|_  test:<empty> => Valid credentials

| ldap-rootdse: 
| LDAP Results
|   <ROOT>
|       currentTime: 20230606144900.0Z
|       subschemaSubentry: CN=Aggregate,CN=Schema,CN=Configuration,DC=htb,DC=local
|       dsServiceName: CN=NTDS Settings,CN=FOREST,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=htb,DC=local
|       namingContexts: DC=htb,DC=local
|       namingContexts: CN=Configuration,DC=htb,DC=local
|       namingContexts: CN=Schema,CN=Configuration,DC=htb,DC=local
|       namingContexts: DC=DomainDnsZones,DC=htb,DC=local
|       namingContexts: DC=ForestDnsZones,DC=htb,DC=local
|       defaultNamingContext: DC=htb,DC=local
Service Info: Host: FOREST; OS: Windows

Nmap 7.93 scan initiated Tue Jun  6 10:32:01 2023 as: nmap -sV -sS -sC -p 139,445 -oN nmap/SMBSCRIPTS.nmap 10.10.10.161
Nmap scan report for 10.10.10.161
Host is up (0.29s latency).

PORT    STATE SERVICE      VERSION
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-time: 
|   date: 2023-06-06T14:39:06
|_  start_date: 2023-06-06T14:24:18
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
|_clock-skew: mean: 2h26m53s, deviation: 4h02m29s, median: 6m52s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2023-06-06T07:39:03-07:00

SMBCLIENT STATUS:
Password for [WORKGROUP\guest]:
session setup failed: NT_STATUS_ACCOUNT_DISABLED
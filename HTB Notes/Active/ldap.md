
LDAP Is shortened for lightweight directory access protocol. It works on port 329 and it is used to access different information. 

The information is usually different but it is split to different data:
**Descriptive data: Name and location
Static data: Information that doesn't change
Valuable data: Information that is being used daily with core functions.**

An LDAP query typically involves:
- **Session connection.** The user connects to the server via an LDAP port. 
- **Request.** The user submits a query, such as an email lookup, to the server. 
- **Response.** The LDAP protocol queries the directory, finds the information, and delivers it to the user. 
- **Completion.** The user disconnects from the LDAP port.

People can tackle all sorts of operations with LDAP. They can:

    Add. Enter a new file into the database. 
    Delete. Take out a file from the database. 
    Search. Start a query to find something within the database. 
    Compare. Examine two files for similarities or differences. 
    Modify. Make a change to an existing entry.

Using ldapsearch:

The easiest way to search LDAP is to use ldapsearch with the “-x” option for simple authentication and specify the search base with “-b”.

```$ ldapsearch -x -b <search_base> -H <ldap_host>```

For example:
```
$ ldapsearch -x -b "dc=active,dc=htb" -H ldap://10.10.10.100 -D 'SVC_TGS' -w 'GPPstillStandingStrong2k18' -s sub "(objectclass=user)" "!(1.2)"
```

Actually pretty proud of myself for building that command, used this website https://docs.oracle.com/cd/E19623-01/820-6169/ldapsearch-examples.html

The '-s' flag is for enumerating all tree elements. Option base would enumerate only the top level so it wouldn't give many results.

The objectclass is for users (wow smart)
And the "!(1.2)" is for enumerating the base and the 2 is for not giving any non-disabled accounts.

==The value of “2” corresponds to a disabled account status, and so the query above will return active users username in the active.htb domain.==

```
Administrator, Users, active.htb
dn: CN=Administrator,CN=Users,DC=active,DC=htb
Guest, Users, active.htb
dn: CN=Guest,CN=Users,DC=active,DC=htb
krbtgt, Users, active.htb
dn: CN=krbtgt,CN=Users,DC=active,DC=htb
SVC_TGS, Users, active.htb
dn: CN=SVC_TGS,CN=Users,DC=active,DC=htb
```

Alternative to this is using Impacket's GetADUsers.py but that's just too easy lmao.

![[getadusers.py.png]]


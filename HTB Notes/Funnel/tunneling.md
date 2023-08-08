
```bash
christine@funnel:~$ ss -lp

State       Recv-Q      Send-Q           Local Address:Port
LISTEN      0           4096             127.0.0.53%lo:domain
LISTEN      0           128                    0.0.0.0:ssh
LISTEN      0           4096                 127.0.0.1:postgresql                
LISTEN      0           4096                 127.0.0.1:45401                     
LISTEN      0           32                           *:ftp
LISTEN      0           128                       [::]:ssh                   
```

PostGreSQL is running on 5432 (Not shown in the output but just believe me :D)

Local port forwarding is when you tunnel a port from the target machine to another machine. For example you can connect to a victim machine using ssh and connecting a port from the victim machine to your own. Basically you're tunneling the traffic through the victim machine to your own machine.

Remote port forwarding is when the victim machine has a service you want to access remotely. You can forward a port from the victim machine to another. My guess is that now the attacking machine works as a tunnel.

Dynamic port forwarding is the mix of these two. You can use them both if you want to but you have to configure proxychains to tunnel your traffic through localhost:port using socks.

We want to use local forwarding "ssh -L" option to access postgresql.

```bash 
ssh -L 1234:localhost:5432 christine@10.10.10.10
```

Now we have a new tunnel inside the victim machine:

```bash
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.1:1234          0.0.0.0:*               LISTEN      2142/ssh
```

Because we have opened a new socket in out attacking machine, we can access this tunnel from our attacking machine.

The reason this is useful is because we can use local programs to access services.

```sql 
psql -U christine -h localhost -p 1234 
Password for user christine: 
psql (15.3 (Debian 15.3-0+deb12u1), server 15.1 (Debian 15.1-1.pgdg110+1))
Type "help" for help.

christine=# 
```

Some PortGreSQL commands:
```sql 
\l (list databases)
\c (connect to a specific database)
\dt (list tables)
SELECT * FROM col (prints the contents of a column in a table)
```
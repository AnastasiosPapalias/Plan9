# How I get tls certificates for 9front

First of all, I use linux and drawterm for this for now, but I would like to be able to do it all from 9front at some point.

## Generate the certificate

Install certbot on linux and run the following command

```
certbot certonly --manual -d pmikkelsen.com -d vps1.pmikkelsen.com
```

and do the challenges, they should be easy. I use the diff here to make 9fronts dns server understand the needed records.

EDIT: As of Sun Feb 14 2021, this patch is no longer needed since cinap pushed a [proper fix]([url](http://code.9front.org/hg/plan9front/rev/73935ff27172)).

## Importing the cert and private key

Start drawterm and login as the hostowner. After this, the filesystem of the linux system is available at /mnt/term. Run the following:

```
cd /sys/lib/tls/
cp /mnt/term/etc/letsencrypt/live/pmikkelsen.com/privkey.pem ./
cp /mnt/term/etc/letsencrypt/live/pmikkelsen.com/fullchain.pem ./cert
```

Now the private key must be converted to one that can be loaded into factotum

```
auth/pemdecode 'PRIVATE KEY' privkey.pem | auth/asn12rsa -t 'service=tls role=client' > key
rm privkey.pem
chmod 400 key
```

Add the following to /cfg/$sysname/cpurc to load the private key on boot.

```
cat /sys/lib/tls/key >> /mnt/factotum/ctl
```

Done. The key can also be stored in secstore if that is setup, so it doesn't lay unencryped on the disk.

# SMTP over TLS

I have the following in /bin/service.auth/tcp25

```
#!/bin/rc
user=`{cat /dev/user}
exec upas/smtpd -c /sys/lib/tls/cert -n $3
```

Notice I had to put it in the /bin/service.auth folder so that it could find the private key.

# Https with rc-httpd

I have the following in /bin/service.auth/tcp443

```
#!/bin/rc

exec tlssrv -c /sys/lib/tls/cert -l /sys/log/https /bin/service/tcp80 $*
```

Again, in the /bin/service.auth folder. It simply wraps the plain http service in a tls wrapper. The plain tcp80 service looks like this for me

```
#!/bin/rc
PLAN9=/
auth/none /rc/bin/rc-httpd/rc-httpd >>[2]/sys/log/www
```

## Authors

- [@Peter's Website / Random notes](https://pmikkelsen.com/plan9/lets_encrypt)

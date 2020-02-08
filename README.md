Want to play with TLS certificates but self-signed server certificates are
boring? Fancy running your own *Root CA*/*Intermediate CA* setup as a fist
step to conquer the world?

`ekeca`!

## What is this about

There are [a few articles][etoobusy-openssl] in [ETOOBUSY][] about
fiddling with [OpenSSL][].

The bottom line is that sometimes you will have to deal with certificates
in a proper *Root CA*/*Intermediate CA*/*Server* chain, and having the
possibility to play with it a bit can be an invaluable tool to learn.

An [asciinema][] can help you figure out what this is about, really:

<script id="asciicast-299207" src="https://asciinema.org/a/299207.js" async></script>

## Installation

This is just a simple shell script - it should work anywhere you have
a POSIX shell available, no fancy stuff included. Just grab the latest
version, e.g. like this:

```shell
curl -LO https://raw.githubusercontent.com/polettix/ekeca/master/ekeca
chmod +x ekeca
mv ekeca ~/bin
```

and you should be all set.

Of course... you will need to have [OpenSSL][] installed!

## Usage

The main script implements a few *subcommands*, the invocation is as
follows:

```shell
ekeca <subcommand> [<arg> [<arg> [...]]]
```

We will not describe every command here, BUT just remind you that you can
get `help` should you need it:

```shell
$ ekeca  # also: ekeca help
./ekeca <subcommand> [<arg> [<arg> [...]]]

Available subcommands:

- boot
...
```

Run the `boot` command. This will work in the current directory and create
two sub-directories:
   
- `rca` with the *Root CA* stuff
- `ica` with the *Intermediate CA* stuff

After this, every time you are in the directory you will enjoy using these
two CAs.

Want to generate a certificate for a server? Make sure to note down all
the aliases that should go in the certificate, then run `server_create`
passing all of them. Example:

```
# Our Common Name is myserver.example.com. It should also support
# alternative names myserver.local and localhost.
$ ekeca server_create myserver.example.com myserver.local localhost
Generating a RSA private key
.......................................+++++
.+++++
writing new private key to 'myserver.example.com/key.pem'
# ...
```

A new directory was created, i.e. `myserver.example.com` corresponding to
the first positional parameter, which also doubles down as Common Name.

```shell
$ cd myserver.example.com/
$ ls -l
total 20
-rw-r--r-- 1 foo bar 1704 Feb  8 10:28 certificate.pem
-rw-r--r-- 1 foo bar 1115 Feb  8 10:28 certificate-request.pem
-rw-r--r-- 1 foo bar 3383 Feb  8 10:28 certificates-chain.crt
-rw------- 1 foo bar 1704 Feb  8 10:28 key.pem
-rw-r--r-- 1 foo bar  474 Feb  8 10:28 openssl.cnf
-rwxr-xr-x 1 foo bar  223 Feb  8 10:54 renew.sh
```

File `certificate-chain.pem` is what you are probably after for setting
the chain in the server, it contains `certificate.pem` followed by
`../ica/certificate.pem`, as you would expect.

File `openssl.cnf` is the file used to generate the `certificate-request.pem`
you can modify it and then re-generate things using the helper `regen.sh`
script in the sub-directory:

```shell
$ ./renew.sh
```

In this case you just have to pass the name of the sub-directory,
everything will be taken from `myserver.example.com/openssl.cnf`.

Last, you might want to take a look at the generated stuff, which is what
the `print` sub-command is about:

```shell
$ ekeca print certificate.pem
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 4098 (0x1002)
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = IT, ST = RM, O = Everish, OU = Intermediate, CN = Everish Intermediate CA, emailAddress = intermediate.everish@example.com
        Validity
            Not Before: Feb  8 09:58:24 2020 GMT
            Not After : Mar 21 09:58:24 2020 GMT
        Subject: C = IT, ST = RM, O = Everish, OU = Server, CN = myserver.example.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    ...
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                E5:35:3F:00:A9:46:F2:0E:FA:F7:70:87:E2:95:20:11:E5:47:F5:13
            X509v3 Authority Key Identifier: 
                keyid:70:66:80:AF:F6:F6:4F:7D:CD:19:A6:48:43:C6:30:EB:A9:49:C8:75
                DirName:/CN=Everish Root CA/C=IT/ST=RM/L=Roma/O=Everish/OU=Root/emailAddress=root.everish@example.com
                serial:10:00

            X509v3 Basic Constraints: 
                CA:FALSE
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Subject Alternative Name: 
                DNS:myserver.example.com, DNS:myserver.local, DNS:localhost
    Signature Algorithm: sha256WithRSAEncryption
         ...
```

[OpenSSL]: https://www.openssl.org/
[etoobusy-openssl]: https://github.polettix.it/ETOOBUSY/tagged/#openssl
[ETOOBUSY]: https://github.polettix.it/ETOOBUSY
[asciinema]: https://asciinema.org/

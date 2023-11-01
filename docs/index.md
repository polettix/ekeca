---

This is [ekeca][], a bare-bones CA wrapper for [OpenSSL][].

# Installation

[ekeca][] is distributed as a single shell program:

```
curl -LO https://github.com/polettix/ekeca/raw/master/ekeca &&
chmod +x ekeca
mv ekeca ~/bin  # or any other place in PATH
```

[ekeca][] is a wrapper around [OpenSSL][], so the `openssl` command-line
program MUST be available.

# Getting Started

[ekeca][] is a "super"-command that gives access to a host of
sub-commands, invoked like this:

```
$ ekeca sub-command [opt1 [opt2 [...]]
```

The first two commands that can be useful are `commands` and `help`, to
get respectively the list of available commands and help about them. As
an example, this is the `commands` sub-command's output:

```
$ ekeca commands   # as of 2023-11-01
/path/to/ekeca <subcommand> [<arg> [<arg> [...]]]

Available subcommands:
- check_association <key> <certificate>
- boot
- clean
- client_create [<common-name> [<alt-name> [...]]]
- client_renew <dirname>
- commands
- help
- htpasswd_line
- ica_create
- ica_reset
- ica_sign <certificate-request> [<certificate>]
- ica_server_sign <certificate-request> [<certificate>]
- ica_client_sign <certificate-request> [<certificate>]
- print <pem-file>
- rca_create
- rca_reset
- rca_sign <certificate-request> [<certificate>]
- rca_update
- server_create [<common-name> [<alt-name> [...]]]
- server_renew <dirname>
- server_standalone [<common-name> [<alt-name> [...]]]
```

The `help` sub-commands can be invoked without parameters, in which case
the whole help will be printed, or providing command-line parameters
representing sub-commands (e.g. read from the `commands` output above),
so that the output is restricted to those sub-commands only:

```
$ ekeca help commands help
ekeca <subcommand> [<arg> [<arg> [...]]]

Requested subcomands:

- commands
      print list of supported commands

- help
      print this help message
```

# Quick one-off certificates for development/testing

Sometimes the need is to generate a *credible* setup resembling what
would happen with a bought certificate:

- browsers/systems know about Root Certification Authorities.
- certificates are signed by Intermediate Certification Authorities,
  whose certificates are in turn signed by the Root Certification
  Authorities known to browsers/systems.

To make things work, the Root CA certificate must be installed in the
browser/system, which can be a big risk if the associated key gets lost
for any reason.

To address this concern, the idea is to generate a new Root CA and an
Intermediate CA (to “simulate the real world”) just for the specific
certificate and then throw everything away except the strict necessary.
This includes deleting the private keys for the Root CA and the
Intermediate CA, which are the real source of the risk above.

The artifacts that are preserved are:

- `root-for-clients.crt` is the Root CA certificate that should be
  installed into the clients (browser(s), curl, …). This will let them
  accept our server certificate.
- `server.key` is the server’s private key, to be installed in the
  server.
- `server.crt` is the certificates chain including both the server’s
  proper certificate and the Intermediate CA’s certificate (it’s easy to
  separate the two if needed).

There’s no automation for installing the Root CA certificate in the
system/browsers.

Command `server_standalone` generates the files above:

```
- server_standalone [<common-name> [<alt-name> [...]]]
      create a standalone server certificate, complete with the Root CA
      certificate to install into the client. This is one-shot with no
      leftovers or space for renewals. Installing the Root CA
      certificate should be fine because we ditch the corresponding
      private key and no further certificates can be generated for it.
```

Let’s see it in action by generating the files for certificating
`whatever.com` as well as the whole wildcard `*.whatever.com`:

```
$ ekeca server_standalone whatever.com '*.whatever.com'
# ... stuff happens

$ ls whatever.com
root-for-clients.crt  server.crt  server.key
```

# The full game

[ekeca][] supports generating the Root CA, the Intermediate CA, as well
as server and client certificates, also providing control over different
parameters by generating specific configuration files that can be
customized.

<script id="asciicast-299207" src="https://asciinema.org/a/299207.js" async=""></script>



# Copyright & License

> Copyright 2020 Flavio Poletti
>
> Licensed under the Apache License, Version 2.0 (the "License");
> you may not use this file except in compliance with the License.
> You may obtain a copy of the License at
>
>     http://www.apache.org/licenses/LICENSE-2.0
>
> Unless required by applicable law or agreed to in writing, software
> distributed under the License is distributed on an "AS IS" BASIS,
> WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
> See the License for the specific language governing permissions and
> limitations under the License.

[ekeca]: https://github.com/polettix/ekeca
[OpenSSL]: https://www.openssl.org/

Client scripts for
[anarchyirc-link-block-service](https://github.com/binki/anarchyirc-link-block-service).

# Setup

## Obtain

To start, simply clone this repository into a secure location.

```
$ git clone anarchyirc-link-block-service
```

## Generate Credentials

Then generate a certificate. The client uses this for authenticating
itself to anarchyirc-link-block-service.

```
$ ./client-gen-cert.sh
```

The command should output a PEM-encoded public certificate. Send this
and your `me::name` to the person maintaining the link block
service. If this person determines that you should be allowed to link
with the network, that person should register you with the link block
service and let you know the remote includes endpoint, `«endpoint»`,
you should use.

## Configure Post-Certificate-Renewal Hook

The certificate used by your IRCd to identify itself to the world and
used for authentication when linking to AnarchyIRC should be obtained
from a real Certificate Authority and ideally match your `me::name`
setting.

Generally, you should write a script that changes to the directory in
which you ran `client-gen-cert.sh` and then call `client-request.sh`
with the appropriate parameters, including the `«endpoint»` binki gave
you above. For example, such a script might look like this:

```sh
#!/bin/sh
set -e

# This might be a good place to put your logic for rehashing
# your unrealircd instance so that it loads the new cert:
cd ~/unrealircd-bin
./unreal rehash

# Update the link block service with our new cert so that we
# can link with AnarchyIRC.
cd ~/anarchyirc-link-block-client
./client-request.sh '«endpoint»' ~/unrealircd-bin/server.cert.pem
```

### Let’s Encrypt

If you use `certbot`, likely have already pointed your IRCd at the
certificates it outputs using
[`set::ssl::certificate`](https://www.unrealircd.org/docs/Set_block#set::ssl::certificate)
and
[`set::ssl::key`](https://www.unrealircd.org/docs/Set_block#set::ssl::key). You
should modify the invocation of `certbot` in your `crontab(5)` to run
your script after `certbot` exits (this way, the script runs even if
`certbot` does not obtain a new certificate which should ensure that
the link block service eventually gets your updated certificate even
if there is a temporary failure in the first try).

```crontab
43	2	*	*	*	sh -c 'certbot renew; ~/my-post-renew-script.sh'
```

## Configure IRCd

You should ensure that you have a unique
[`me::sid`](https://www.unrealircd.org/docs/Me_block). Probably just
guess or ask the maintainer of the link block service ;-).

With the `«endpoint»` given to you earlier, you can build the URI that
you should
[`include`](https://www.unrealircd.org/docs/Configuration#Include_directive)
from your `unrealircd.conf`:

```
include "«endpoint»/links.conf";
```

If you are running UnrealIRCd-3.x (you shouldn’t be, it’s
unsupported by them now!), you can request a compatible `links.conf`
by settings the HTTP GET parameter `syntax` to `unrealircd3`:

```
include "«endpoint»/links.conf?syntax=unrealircd3";
```

You can visit the built URI in your browser and verify that it looks
right before pasting it into your `unrealircd.conf`.

To avoid downtime, you may add this to your configuration file while
the IRCd is running. Simply `/oper` up and `/rehash` (some clients
require `/raw rehash` or `/quote rehash`). If there are issues, you
should receive server notices. Communicate with other operators or the
link block service sysop to resolve these issues.

# `client.lab.lan`

Wait for `/edc/hosts` and `/edc/krb5.conf` to be ready.

> [!NOTE]
> If your KDC asks you too, set up `/etc/hosts` based on the [tutorial for KDC](KDC.md)
and copy it to the other computers.

Once they are ready, you've reached [checkpoint 1](#checkpoint-1).

## Checkpoint 1

First of all, check if you can see the other computers by the domain names:
`ping` both `server.lab.lan` and `kdc.lab.lan`.

> [!NOTE]
> If there are more people in your group, `ping` them too.

If the communication works, wait for your KDC to tell you that you've reached [checkpoint 2](#checkpoint-2)

## Checkpoint 2

First, try to access the kerberos console using:
```console
client:~# kadmin -p admin/admin
kadmin: addprinc -randkey host/client.lab.lan
kadmin: ktadd host/client.lab.lan
```

> [!WARNING]
> On your machine, `krb5.conf` must be the same as the KDC's `krb5.conf`.
> Make sure the latest version is copied from KDC to your `/etc/`.

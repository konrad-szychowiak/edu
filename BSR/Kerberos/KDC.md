# `kdc.lab.lan`

## Set-up
> [!NOTE]
> The first two steps—preparing kerberos configuration and setting the hostnames—can be done in parallel.
> Ask the student responsible for the **client** to do it using the description from the text below.

Firstly, we will prepare the kerberos configuration in the `/etc/krb5.conf` file.
It should have at least the following options:

```
[libdefaults]
    default_realm = LAB.LAN
    dns_lookup_kdc = false
    dns_lookup_realm = false

[realms]
    LAB.LAN = {
        kdc = kdc.lab.lan
        admin_server = kdc.lab.lan
    }
[domain_realm].
    .lab.lan = LAB.LAN
```

Secondly, in order for the other computers in your group to recognise each other
by the selected hostnames, you need to edit the `/etc/hosts` file.
At the very end of the file, add
```
<IP_KDC> kdc.lab.lan kdc
<IP_SERVER> server.lab.lan
<IP_CLIENT> client.lab.lan
```

> [!TIP]
> `<IP_KDC>`, `<IP_SERVER>` and `<IP_ CLIENT>` denote the ip addresses of the computers respectively.

> [!NOTE]
> If there are more people in your group, set names of your choosing for their IPs too.

Copy both `/etc/krb5.conf` and `/etc/hosts` files to the other computers in the group.
If the **client** prepared `/etc/hosts` copy that file from the client's computer to the other machines. 

Let the others know that you've reached [checkpoint 1](#checkpoint-1).



## Checkpoint 1

First of all, check if you can see the other computers by the domain names: 
`ping` both `server.lab.lan` and `client.lab.lan`.

> [!NOTE]
> If there are more people in your group, `ping` them too. 

If that works, you can continue with KDC's configuration.

Modify the ACLs for the KDC. In the `/var/lib/kerberos/krb5kdc/kadm5.acl` file. Paste:
```
*/admin@LAB.LAN *
* */*@LAB.LAN and
```

Next, we will want to create users.
In order to do this, we need to initialise the KDC's database first:
```console
kdc:~# kdb5_util create -r LAB.LAN -s
```

Now, to access the administration console, we will use:
```console
kdc:~# kadmin.local
```

Once in the console, run the following commands to create new principals and add them to the _key tab_.

```console
kadmin.local: addprinc admin/admin
kadmin.local: addprinc student
kadmin.local: ktadd -k /var/lib/kerberos/krb5kdc/kadm5.keytab kadmin/admin kadmin/changepw
kadmin.local: exit
```

Start the kerberos services and make sure they are running correctly.

```console
kdc:~# service krb5kdc start
kdc:~# service kadmind start
```

> [!TIP]
> For simple verification of the result, you can view the contents of the domain registry:
> ```console
> kdc:~# kdb5_util dump
> ```

Once the services are started, let the others in the group know that you've reached checkpoint 2.

# Checkpoint 2

In checkpoint 2, client and server(s) will add principals for their machine
(starting with `host/` in the name).
To see if they have created the principals successfully, run:
```console
kadmin.local: listprincs
```

If there are any problems, look through the logs of the Kerberos server:
```console
kdc:~# tail -f /var/log/krb5/krb5kdc.log
```

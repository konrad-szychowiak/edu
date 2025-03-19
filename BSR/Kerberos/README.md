# Kerberos

![Static Badge](https://img.shields.io/badge/version-v0-blue?style=for-the-badge)

Kerberos is a distributed cryptographic authentication system with a trusted third party. The role of this party,
The role of this party, which is trusted by all the entities in the system (principals), is performed by the KDC (Key
Distribution Center) server, which performs the
authentication and - potentially (though rarely in practice to the full extent of its meaning) - authorisation services.
The system
uses a central register of credentials. Entities (users, computers
applications) receive a ticket from the KDC containing, among other things, the session key used to access individual
services, e.g. for
The ticket contains, among other things, the session key for accessing specific services, e.g. remote login, printing,
etc. Each client computer and each
user in the authentication domain (realm) must be individually attached to it.

## Preparation

In this exercise you will work in groups of 3 people (at the minimum).
You will divide between yourselves the following roles:

* **KDC** – key distribution centre – Kerberos server at the domain address `kdc.lan.lab`,
* **Server** - the web service server we will want to access, `server.lan.lab`, and
* **Client** - the client machine from which we will attempt to access the service, `client.lan.lab`.

> [!NOTE]
> If more students join your group, every person over the initial number of 3
> should prepare another **service** server.
> Then, whoever takes the role of the **client** will have to check with all the servers
> before continuing to the next checkpoint.

First, once you know which role you are responsible for,
start by setting your hostname to the corresponding domain name:

```shell
hostname <DOMAIN_NAME>
```

> [!CAUTION]
> Remember! Don't copy code snippets without thinking.
> Check and compare with the examples in the files and the configuration already done by you and others in your group.


Now, we will make sure _each_ machine is ready to use Kerberos.
The laboratory will require the following packages: `krb5`, `krb5-client` and `krb5-server`.
You should check if your system has them:

```shell
# check what is already installed
sudo zypper se krb5
``` 

<details>
<summary>If any package is missing, install it.</summary>

To install the packages, use:

```shell
sudo zypper in krb5 krb5-server krb5-client
```

Of course, if only some are missing, you can install only those.
</details>

## Exercises

The next steps are divided into the three roles mentioned earlier.

> [!TIP]
> Some of the steps can be done in parallel; look for hints in the tutorials.
> Also, while waiting for other students to finish, you can help them do their part
> to get more out of the exercises.

The tasks for [KDC](KDC.md) are discussed first,
followed by [server](#server) and [client](#client).
Those links will led you to your sections.
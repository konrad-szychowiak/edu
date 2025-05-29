On the IDS machine:

```shell
sudo zypper addrepo https://download.opensuse.org/repositories/server:monitoring/15.6/server:monitoring.repo
sudo zypper --no-gpg-checks refresh
sudo zypper -n install libdaq1
sudo zypper -n --no-gpg-checks install https://www.cs.put.poznan.pl/mszychowiak/security/tools/lab_IDS/snort-2.9.4-9.6.x86_64.rpm
```

On the 'attacker' machine:

```shell
sudo zypper in nmap
```
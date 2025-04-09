# Restricted Shell

```console
lab-sec-0:~# useradd -m agent007
lab-sec-0:~# ln /bin/bash /bin/rbash
lab-sec-0:~# su --shell /bin/rbash -l agent007
agent007@lab-sec-0:/~$ 
```

Spróbuj coś uruchomić!

```cosnole
agent007@lab-sec-0:/~$ echo $PATH
/usr/lib/restricted/bin
```

```console
lab-sec-0:~# su --shell /bin/rbash -l agent007 
agent007@lab-sec-0:~$ ps
  PID TTY          TIME CMD
 4953 pts/3    00:00:00 rbash
 5006 pts/3    00:00:00 ps
agent007@lab-sec-0:~$ ls -l
total 0
drwxr-xr-x 1 agent007 users 0 Mar 15  2022 bin 
```

```console
lab-sec-0:~# chsh --shell /bin/rbash agent007
lab-sec-0:~# su -l agent007
agent007@lab-sec-0:~$ echo $PATH
/usr/lib/restricted/bin
```

Dodaj `vi` do `rbash`a i zobacz, czy możesz go użyć do ucieczki z ograniczonego środowiska.
Oczekiwany efekt:
```
/bin/rbash: /bin/bash: restricted: cannot specify `/' in command names

shell returned 1

Press ENTER or type command to continue
```

---

# AppArmor


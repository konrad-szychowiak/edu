# AppArmor and Restricted Shells

1. Restricted Bash (`/bin/rbash`)
2. Basic AppArmor Profiling (`aa-genprof`, etc.)
3. Your own custom restricted shell (`~/confined_bash`)

# Restricted Shell

### Setting Up Environment

```console
lab-sec-0:~# useradd -m agent007
lab-sec-0:~# ln /bin/bash /bin/rbash
lab-sec-0:~# su --shell /bin/rbash -l agent007
agent007@lab-sec-0:/~$ 
```

### Inspecting Environment

```cosnole
agent007@lab-sec-0:/~$ echo $PATH
/usr/lib/restricted/bin
```

Let's try running something.

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

Add `vi` to your `restricted/bin/` and see if you can use it to escape the confines of the `rbash`.
Expected result:
```
/bin/rbash: /bin/bash: restricted: cannot specify `/' in command names

shell returned 1

Press ENTER or type command to continue
```

---

# AppArmor

1. Write a script
    ```bash
    #!/bin/bash
        
    file=~/sample.txt
    [ $1 ] && file=$1
    
    touch $file && echo "File $file created"
    rm $file    && echo "File $file deleted"
    ```
1. Start the policy generator but DO NOT profile the script yet! Instead, stop the generator and continue to the next step.
    ```console
    lab-net-X:~# aa-genprof <app>
    ```
1. Use AppArmor GUI to inspect how the policy can be created
1. Use the in-console generator (`aa-genprof`) this time and set up a policy for your script
2. Switch the profile to the enforce mode `aa-enforce` and check if you can run the script.
3. Now modify the script to point you to a different file and folder. Check the same folder but different file (e.g. ~/sample2.txt) and different directory (e.g. /temp/smaple.txt)

# Exercises

> [!WARNING]
> Gdy profiler w środowisku graficznym lub konsolowym zaproponuje użycie regół dla LXC (Linux Conteiners, `.../lxc/...`) _NIE_ wybieraj tej opcji.
> LXC zapewniają pewne obejścia, które mogą popsuć nasze przykłady i uniemożliwić (lub stanowczo utrudnić) wykonanie zadań.

1. Create a new shell about to be confined:
    ```bash
    cp /bin/bash /bin/confined_bash
    ```
1. Use aa-autodep to create an initial profile for this confined shell. Check the profile.
1. Edit the profile so it will allow users to run id and ls commands.


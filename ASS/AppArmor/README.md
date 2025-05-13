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
1. Start the police generator and STOP!
    ```console
    lab-net-X:~# aa-genprof <app>
    ```
1. Use AppArmor GUI to inspect how the policy can be created
1. Use the in-console generator (`aa-genprof`) this time and set up a policy for your script

# Exercises

1. Create a new shell about to be confined:
    ```bash
    cp /bin/bash /bin/confined_bash
    ```
1. Use aa-autodep to create an initial profile for this confined shell. Check the profile.
1. Edit the profile so it will allow users to run id and ls commands.


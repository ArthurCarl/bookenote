# Linux Command Beginner

## whoami

`whoami` display currently logged-in user
 
```shell
$ whoami
alfred

$ id -un # same 
alfred 
```

## How to find out when Debian or Ubuntu package installed or updated
`/var/log/dpkg.log` package install or update log

## check user password expiration 

```
$ chage -l userName

```

## `w`

`w` displays information about currently logged in users and what each user is doing
```
$ w
21:54  up 24 days,  1:19, 3 users, load averages: 1.43 1.82 2.13
USER     TTY      FROM              LOGIN@  IDLE WHAT
alfred   console  -                26Oct19 24days -
alfred   s001     -                Sat22   2days -bash
alfred   s000     -                Sat22       - w


```

## `du` `ncdu`
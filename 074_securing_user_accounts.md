# Securing User Accounts

`sudo` allows users to perform administrative tasks without the risks of having them logged in a the `root` user. `sudo` also allows fine-grained control what the individual users are allowed to do, e.g., what commands they are allowed to execute and with what arguments (i.e., the users get only the privileges they need to perform their respective tasks).

Moreover, `sudo` allows users to perform the tasks with elevated privileges by entering their own password (no risk that a threat actor learns the `root` password). Finally, `sudo` improves auditing capabilities because you'll be able to see what users are doing with their admin privileges. This gives network users enough privileges to get their job done, but no privileges beyond that.

## `sudo` logs
According to this post, on RHEL-based Linux systems sudo logs are in `/var/log/secure` and on Debian-based Linux systems it's in `/var/log/auth.log`.

## Adding users to a group
* `groups` shows the groups a user is in
* Under centOS, the admin users are in the `wheel` group, under Debian they're in `sudo` group.
* `sudo usermod -a -G wheel maggie` to add user `maggie` to the group `wheel`
* create user by `sudo useradd lionel`
* set user password by `sudo passwd lionel`

## `sudo` policy file
* always open by doing `sudo visudo`
* the entry `%wheel ALL=(ALL) ALL` in the `sudoers` file means that:
  * the users in the group `wheel`
  * on ALL hosts (the 1st `ALL`)
  * as any user (the 2nd `(ALL)`)
  * can execute any command (the 3rd `ALL`)
* User alias in `sudoers` file: `User_Alias ADMINS = jsmith, mikem`

## Disabling `root` account
* `sudo passwd -l root` locks the password of the given account by adding a `!` in front of the password which turns it into a value that match no possible hash value

## `sudo` timer
* by default, once a user call `sudo` and enters their password, they can perform another `sudo` command within five minutes without having to re-enter their password
* set `sudo` timer to 0 by `sudo -k`
* to set `sudo` timer to 0 by default, `sudo visudo` and add the line `Defaults timestamp_timeout = 0`
* you can also set `sudo` timer for a specific user by adding the line `Defaults:lionel timestamp_timeout = 0`

## Misc
* `su - lionel` login to user `lionel` account

## `sudo` privileges
* to find out your `sudo` privileges: `sudo -l`

## Preventing users from having root shell access
* check `sudoers` file

## Preventing users from using shell escapes
* e.g., prevent `vim` shell escape `:!<cmd>`
* problem: even if a user is only allowed to open a single file with `sudo` privilege, that user can use shell escape feature to execute other commands with root rights
* this can be fixed using `sudoedit` since `sudoedit` has no shell escape feature: `frank ALL=(ALL) sudoedit /etc/ssh/sshd_config`

## Preventing user from executing other dangerous commands
* it is also advisable to prevent using commands like `cat`,`cut`, `awk` or `sed` with `sudo` privileges.
* the user's actions with commands should also be limited: `sylvester ALL=(ALL) /usr/bin/systemctl status sshd, /usr/bin/systemctl restart sshd` instead of the insecure `ylvester ALL=(ALL) /usr/bin/systemctl sshd`

## Letting users run as other users
Say you want user `frank`, a database admin, to be able to run script `db_script.sh` as user `database`. Then, add the following line to the `/etc/sudo.conf` file:

`frank ALL=(database) /usr/local/sbin/db_script.sh`

With the above line, `frank` can run the script by executing: `sudo -u database /usr/local/sbin/db_script.sh`

In addition, never allow users to run with `sudo` privileges scripts that are in their home directory. This is because they can change the scripts, e.g., add `sudo -i`. To prevent this, move such a script to e.g., `/usr/local/sbin` and add an appropriate line to `/etc/sudo.conf` (e.g.,  `frank ALL=(database) /usr/local/sbin/script.sh`)

## Detecting and deleting default user accounts
* To detect default user accounts, check the `/etc/password` file (seems to be `/etc/passwd` under CentOS)
* Then check in `/etc/sudoers` what commands the users may execute
* Be especially aware if you see something like `raspex ALL=(ALL) NOPASSWD: ALL`

## Locking down user's home directories on RHEL and CentOS
* `/etc/login.defs` contains the value `UMASK`
* if `UMASK` is set to e.g., `077`, the home directories created by calling the `useradd` command will be created with permissions `700`. That is, only the user whom the home directory belongs to is able to read and write the files.

## `useradd` on Debian/Ubuntu
* `useradd` command on a Debian/Ubuntu machine does not have the preconfigured defaults like with RHEL/CentOS. Thus, if you call `sudo useradd frank` no home directory is created and the users gets the wrong default shell assigned.
* to fix this, you would need to do something like `sudo useradd -m -d /home/frank -s /bin/bash frank`

## `adduser` on Debian/Ubuntu
* `adduser` is an interactive way to create user accounts
* however, it is not useful for use in shell scripting
* on the other hand, `adduser` will automatically encrypt user's home directory if you have `ecryptfs-utils` installed (`sudo apt install ecryptfs-utils`)
* do this by executing:

```shell
$ sudo adduser --encrypt-home cleopatra
```

When `cleopatra` logs in for the first time, she'll want to write her passphrase down and store it in a safe place by executing

```shell
$ ecryptfs-unwrap-passphrase
```

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

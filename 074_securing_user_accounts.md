# Securing User Accounts

`sudo` allows users to perform administrative tasks without the risks of having them logged in a the `root` user. `sudo` also allows fine-grained control what the individual users are allowed to do, e.g., what commands they are allowed to execute and with what arguments (i.e., the users get only the privileges they need to perform their respective tasks).

Moreover, `sudo` allows users to perform the tasks with elevated privileges by entering their own password (no risk that a threat actor learns the `root` password). Finally, `sudo` improves auditing capabilities because you'll be able to see what users are doing with their admin privileges.

## `sudo` logs
TBD
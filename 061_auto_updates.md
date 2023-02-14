# Updates

## `yum`

It is possible to install the `yum-cron` package to automatically run `yum`. The configuration is stored in `/etc/yum/yum-cron.conf`. The utility can be installed like this:

```shell
~ sudo yum install yum-cron
~ sudo systemctl enable --now yum-cron
```

In the `/etc/yum/yum-cron.conf` you can configure the type of automatic update, e.g., `default`, `security`, `security-severity:Critical`, `minimal`, etc.

With the `needs-restarting` command (part of the `yum-utils` package) you can query the system if services or the system itself need to be restarted. E.g., `sudo needs-restarting` shows the entire information while `sudo needs-restarting -s` only shows services that need to be restarted.

## `dnf`

You can install `dnf-automatic` with `sudo dnf install dnf-automatic` and configure it in the `/etc/dnf/automatic.conf` file. You then need to set a timer for `dnf-automatic` using:

```shell
~ sudo systemctl enable --now dnf-automatic.timer
```

You can verify that the timer is running using:

```shell
~ sudo systemctl status dnf-automatic.timer
```

Check `man dnf-automatic` for more infos.

## Best Practices in Enterprise Setting
* Restrict what packages end users are allowed to install
* Maintain own repositories with approved packages and approved updates
* Test updates on a separate test network before allowing them to be installed on a production network

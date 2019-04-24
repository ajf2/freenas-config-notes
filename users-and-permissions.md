# Users and Permissions
_Using FreeNAS 11.2 U3_

Users in FreeNAS do not propogate into jails. Each jail will have its own set of users and groups.

Sometimes it may be desirable to share user accounts between the host system and its jails, for example if multiple jails need to access the same data but the services that run in the jails each use different user accounts.

### Changing Service User/Group Accounts
First, create your user and group using the `adduser` command. The important bit is that the user id in the jails matches the user id in the host FreeNAS system. See https://www.ixsystems.com/documentation/freenas/11.2/jails.html#accessing-a-jail-using-ssh for an example. For a simple service account, you probably want to configure it with its own login group, `nologin` shell, its own home directory and no password authentication.

If you make a mistake you can always delete the problematic account with `rmuser`.

Then stop the service.
```
service <service_name> onestop
```

Next, add these to the `/etc/rc.conf` file in the respective jails.
```
<service_name>_user="<user>"
<service_name>_group="<group>"
```

Then make sure the new user account can read the service's files, likely in `/usr/local/`.
```
chown -R <user>:<group> <path/to/service>
```

And finally, start the service again.
```
service <service_name> onestart
```
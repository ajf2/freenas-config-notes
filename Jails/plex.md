# Setting Up Plex Media Server
_Using FreeNAS 11.2 U3_

These notes assume the use of iocage jails, new in FreeNAS 11.2, not the deprecated warden jails.

These notes area adapted from https://support.plex.tv/articles/201370363-move-an-install-to-another-system/

New with FreeNAS 11.2 is the Plex Media Server (PlexPass) plugin. This allows you to use your Plex account credentials to unlock the features only available to PlexPass subscribers.

## 1) Set up the new plugin.
Install the new plugin and start it up. Don't sign in to its web interface yet, and sign out if you have.

## 2) Mount storage to the new jail.
**IMPORTANT: There seems to be a bug in FreeNAS 11.2U3 that causes a jail to fail to start if you try to add a mount point with a space in. Using a backslash before the space makes it worse, so avoid spaces altogether.**

Using the new FreeNAS UI, stop the Plex jail and add mount points for your media files to the new jail. Make sure to set them as read-only.

Try to match up the structure of the old jail's mount points so that reconfiguring isn't necessary. For consistency, a suggestion is to mount everything to the new jail's `/Plex/` directory.

You may want mount points for movies, TV shows, music etc. as well as for some free space to use for video optimisation when streaming to mobile.

## 3) Migrating data from the deprecated warden jail version.
The new plugins use the iocage jail system and as such, will require data to be migrated from the old warden jail, if you have one.

#### 3.1) Disable trash auto-empty.
In the old Plex server UI, go to `Settings -> Library` and disable `Empty trash automatically after every scan`. This is recommended by the Plex guide, but it doesn't say why.

#### 3.2) Start the new plugin, but stop the services.
Using the FreeNAS UI, start up the new plugin, then go to the Jails page and open a shell window to the plugin's jail. Run the following to stop Plex from running (use `service -e` to list the running services).
```
service plexmediaserver_plexpass stop
```

The old Plex plugin jail can be stopped through the FreeNAS legacy UI.

#### 3.3) Backup the existing data
From the old plugin's shell, run the following to create a backup of Plex's data.
```
cd /var/db
tar -czf plexdata.tar.gz plexdata/
```
This will take a while. Feel free to copy the backup off to another system if you like.

#### 3.4) Copying the data to the new jail
Once the backup is finished, open a shell from the FreeNAS server's new UI and copy the existing data over to the new jail. Assuming `/mnt/storage/` is your ZFS pool:
```
cp -Rf /mnt/storage/jails/plexmediaserver_1/var/db/plexdata/Plex\ Media\ Server/ /mnt/storage/iocage/jails/plex/root/Plex\ Media\ Server/
```

#### 3.5) Fix permissions in new jail
Once the data has finished copying to the new jail, log in to the jail's shell and set the file owners to `plex:plex`.
```
cd /
chown -R plex:plex /Plex\ Media\ Server/
```

#### 3.6) Restart the new jail
With the mount points configured, start the new jail up again with the new UI.

## 4) Reconfigure Plex
Use the management link on the overflow menu on the FreeNAS UI's Jails page to open the Plex web interface.

#### 4.1) Log in to Plex.
Log in with your Plex account to open the server's web interface.

#### 4.2) Update login.
Go to `Settings -> General` and click `Sign Out`, wait and click `Claim Server`, which should pop up. This will apparently reset connection information and certificates.

#### 4.3) Update libraries.
Go to `Manage -> Libraries` and edit each library to point to the new file locations.

Wait for the library scans to finish.

#### 4.4) Final maintenance.
Still in the new Plex server's UI, go to `Settings -> Library` and turn on `Empty trash automatically after every scan` again.

Now go to `Manage -> Troubleshooting` and click `Clean Bundles`.

Wait for this to finish, then also click `Optimise Database`.

## 5) Stop and delete the old jail
Once the new Plex plugin is working fine, go back to the FreeNAS legacy UI and delete the old Plex server from the Plugins page.
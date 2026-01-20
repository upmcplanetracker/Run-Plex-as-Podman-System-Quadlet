Plex Podman Quadlet Deployment
==============================

**Disclaimer:** I am not affiliated with, associated, authorized, endorsed by, or in any way officially connected with the Plex project. This should be a drop-in replacement, but back up any config files just in case.

This guide demonstrates how to turn a Plex Docker container into a **System Podman Quadlet**. Unlike standard containers, a system quadlet is managed by `systemd` and will start automatically every time your Ubuntu system boots.

**Note:** I have not been successful getting this to run rootless while maintaining QSV/GPU access. If you are running this without a GPU, you can likely convert this to rootless with little effort.

System Information
------------------

*   **Tested Environment:** Ubuntu 25.10
*   **Podman Version:** 5.4.x
*   **Configuration Type:** System-wide Quadlet (Root)

1\. The Quadlet File (plex.container)
-------------------------------------

Download or create the `plex.container` quadlet file. Ensure that once moved to its final location, it is owned by `root`.

2\. Installation & Setup
------------------------

### Step A: Clean up existing containers

If you are currently running Plex in Docker or a manual Podman container, stop and remove it first to prevent name conflicts.

### Step B: Set Permissions

Add your user to the `video` and `render` groups to allow hardware access for Intel QuickSync:

`sudo usermod -aG video,render $USER`

### Step C: Identify User IDs

Check your current User ID (UID) and Group ID (GID). Update the `PLEX_UID` and `PLEX_GID` values in the `.container` file if they are not 1000.

`id`

### Step D: Move and Edit

Move the file to the systemd directory and edit your paths, timezone, and QSV settings:

`sudo mkdir -p /etc/containers/systemd`
`sudo mv plex.container /etc/containers/systemd/`
`sudo nano /etc/containers/systemd/plex.container` to edit it

3\. Activation
--------------

Run these commands to trigger the Quadlet generator and start your server:

`sudo systemctl daemon-reload`
`sudo systemctl start plex`

4\. Post-Install Verification
-----------------------------

### Check GPU Access

To ensure Plex can see your Intel Graphics for QuickSync, run:

`podman exec -it plex ls -l /dev/dri`

You should see `card0` and `renderD128` in the output. Because we are using `UserNS=host`, you should see your actual username/group (or numeric IDs) rather than "nobody".

### Check Logs

If the container isn't starting, use journalctl or podman logs to investigate:

`sudo journalctl -u plex -f`
# OR
`sudo podman logs plex`

5\. Updating the Container
--------------------------

Because `AutoUpdate=registry` is set, you can update the container (and all other System Quadlets) with a single command:

`sudo podman auto-update`

_Note: Because this is a system quadlet, it will now start automatically on every system reboot._

# SmokeScreen
Host your Plex media with rclone on cloud storage on Ubuntu Linux.

# Thank You
Reddit users /u/gesis and /u/ryanm91

# Pre-requisites
This project relies on:
* [rclone](https://rclone.org/downloads/) 1.41+
* [Plex Media Server](https://plex.tv)
* Sonarr, Radarr, Usenet Client, Torrent Client (optional)

IMPORTANT: Plex must run as the SAME USER as the scripts. On Ubuntu, this is done by:

At the command line:  
`sudo service plexmediaserver stop`  
`sudo chown -R myuser:myuser /var/lib/plexmediaserver`  
`sudo systemctl edit plexmediaserver`  

This is the content to be placed in the editor  
`[Service]`  
`User=myuser`  
`Group=myuser`  

Then:  
`sudo systemctl daemon-reload`  
`sudo service plexmediaserver start`  

# Installation
clone the repo:  
`cd ~/`  
`git clone https://github.com/FourFingerLifeHug/SmokeScreen.git`  

Move the config file in to place  
`mkdir ~/.config/SmokeScreen`  
`mv ~/SmokeScreen/smokescreen.conf ~/.config/SmokeScreen/smokescreen.conf`  

Move the scripts to ~/bin/:  
`mkdir ~/bin`  
`mv ~/SmokeScreen/* ~/bin/`  

Make sure all the scripts are executable:  
`sudo chmod a+x ~/bin/*`  

# Default Configuration Variables
The default configuration creates folders and mount points in your user's home directory. This may not be acceptable for your configuration, so change them to a more suitable location. All steps in this README refer to the `$variable` name and not the `~/path` to avoid confusion.

# Required rclone Remotes
Create a remote in rclone that points at the top level of your cloud storage, and make sure you create it so that it is writable. This remote will be the `$primaryremote` in the configuration. Then create another remote of type 'cache' that points at the first remote and the subfolder where your media is located, and configure the Plex settings for your server. This will be the `$cacheremote` configuration option.

If you need assistance creating remotes in rclone, visit [their site](https://rclone.org/).

The default configuration options are `$primaryremote` to be called `GSUITE` and `$cacheremote` to be called `GSUITE-CACHE`. 

# Cloud Storage Setup
There is a checking script included that looks for a specific file on cloud storage. Set in the configuration as `$checkfilename`, when Cloud Storage is mounted you should see this file at `$mediadir/$checkfilename`. Use rclone to upload a file of this name to your cloud storage. Example:

    touch ~/google-check
    rclone move ~/google-check GSUITE:PlexMediaFolder

Now mount the system by running the `check.mount` script. You should see your Plex Media Folder from cloud storage mounted at `$mediadir`. If you don't, stop here and resolve it before continuing.

# Plex Media Server Configuration
Plex can be configured however you choose for scanning, thumbnail generation, etc and will work the same as though all your media is on the local hard drive. Simply create your media libraries in Plex so that they point at the folders where your cloud storage is mounted.

# Sonarr and Radarr Configuration
Sonarr and Radarr should be configured to use TV and Movie folders in `$mediadir` as the root for all series/movies. Nothing special is required.

Media that these tools download will follow the path:

* Download client (SAB, Deluge, etc) downloads to temp directory
* Sonarr/Radarr "import" the file to the appropriate subfolders in `$mediadir`
* rclone handles moving the file to cloud storage automatically

If you are NOT using Sonarr or Radarr, manually add episodes and movies to your media folders. Be sure that media manually placed here follows Plex's media naming expectations.

# Automatic Processing
CRON is used to automatically mount the drives.

Add the following to your user's crontab:

    */5  *  * * * /home/USER/bin/check.mount  >> /home/USER/logs/check.mount.log 2>&1

# A Note About Music
Cloud storage-hosted music doesn't work well with Plex. It takes forever to scan, and a long time to start streaming to devices. It works, but is outside the scope of this project.

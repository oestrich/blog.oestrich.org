---
layout: post
categories:
- linux
- backup
title: Backup with Duply and Duplicity
description: Steps on how to backup with duply and duplicity
date: 2016-06-17 11:30 AM
---

I recently stopped using Crashplan as my backup service of choice. I still wanted backups so I started looking for alternatives. Duply and duplicity came up as a nice choice. This is how I set it up.

This assumes you already have `duplicity` and `duply` installed. My linux of choice is [Arch][arch] and [duplicity][duplicity-arch] was in the repo. [duply][duply-arch] is an AUR that is easily installed.

## GPG

We need to start by generating a GPG key. This will be used to encrypt backups.

`gpg --gen-key`

After generating the key, make sure to save the key ID.

### Issues

I had troubles with pinentry on a GUI-less system. You may need to symlink the `pinentry-curses` version of `pinentry`. Found from [this forum post][pinentry-curses].

```bash
sudo ln -s /usr/bin/pinentry-curses /usr/bin/pinentry
```

I also needed to set up GPG to [allow gpg password from an ENV variable][env-variable]. This is required with GPG 2.1.

### ~/.gnupg/gpg.conf

```
use-agent
pinentry-mode loopback
# ...
```

### ~/.gnupg/gpg-agent.conf

```
allow-loopback-pinentry
# ...
```

## Duply

Start by creating a profile, then edit the configuration.

```bash
$ duply eric create
```

### ~/.duply/eric/conf

This file sets up basic configuration for duply. The file contains a lot of commented out options. These are the only options I have set right now. I commented what they do inline.

```
GPG_KEY='...' # your gpg key id, get from `gpg --list-keys`
GPG_PW='...' # your gpg password
GPG_OPTS='--pinentry-mode loopback' # required to use GPG_PW

TARGET='sftp://eric@hostname/backups/hostname' # backing up to another linux machine via SSH
SOURCE='/home/eric/' # grab my home folder

PYTHON="python2" # arch has python 3 default, set for python 2 instead

# run a full backup once a week
MAX_FULLBKP_AGE=1W
DUPL_PARAMS="$DUPL_PARAMS --full-if-older-than $MAX_FULLBKP_AGE "
```

### ~/.duply/eric/exclude

I wanted to only include certain folders in my home directory. Duplicity has a nice exclude file format you can use to deal with this. Duply has it baked in. The exclude file is created when creating your profile.

```
# keep out junk folders
- /home/eric/prog/*/*/log/
- /home/eric/prog/*/*/tmp/

# Include Folders
+ /home/eric/bin
+ /home/eric/Desktop
+ /home/eric/Documents
+ /home/eric/dotfiles
+ /home/eric/Music
+ /home/eric/ownCloud
+ /home/eric/Pictures
+ /home/eric/prog
+ /home/eric/.ssh

# Exclude everything else
- /home/eric/
```

The important thing to remember is duplicity goes down the list when determining if something is excluded or included. That's why the `log` and `tmp` folders are excluded first. Otherwise they would be included by the `/home/eric/prog` line if it was first.

## Automatic backup

Set cron to run the backup script nightly. I use [keychain][keychain] and needed to source the correct file to get SSH working.

### crontab -e

```
@daily . ~/.keychain/$(hostname)-sh && /usr/bin/duply eric backup
```

## Improvements

Sometime in the future I want to switch to [Google Cloud Storage][gcs] instead of SSH. I'm currently only backing up between local machines while I'm testing this out. I would like to have an onsite and a cloud backup.

[arch]: https://www.archlinux.org/
[duplicity]: http://duplicity.nongnu.org/
[duply]: http://duply.net/
[duplicity-arch]: https://www.archlinux.org/packages/community/x86_64/duplicity/
[pinentry-curses]: https://archlinuxarm.org/forum/viewtopic.php?f=59&t=8096 
[duply-arch]: https://aur.archlinux.org/packages/duply
[env-variable]: https://lists.launchpad.net/duplicity-team/msg02653.html
[keychain]: http://www.funtoo.org/Keychain
[gcs]: https://cloud.google.com/storage/

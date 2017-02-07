### Btrfs sync subvolumes
This is a pretty naive implementation of replication exact set of subvolumes from one location to another, plays nicely with Just backup btrfs.

### Why?
My primary use case is that I'm doing regular snapshots on SSD for my system drive (root, home, etc.) as wel as copying them to external HDD, all done by [Just backup btrfs](https://github.com/nazar-pc/just-backup-btrfs) natively.

However, I also wanted to make offline backup from time to time and this project does just that, copying all snapshots from online HDD to offline HDD that I'm connecting from time to time.

### Requirements
* php-cli (version 5.6+)

### Installation
The whole thing is a single file, so first way is just to copy file `btrfs-sync-subvolumes` somewhere.

Alternatively you can install it globally using Composer like this:
```bash
sudo COMPOSER_BIN_DIR=/usr/local/bin composer global require nazar-pc/btrfs-sync-subvolumes
```
`COMPOSER_BIN_DIR=/usr/local/bin` will instruct Composer to install binary to `/usr/local/bin` so that you'll be able to call `btrfs-sync-subvolumes` right after installation.
Alternatively you can skip it and add `~/.composer/vendor/bin/` to your PATH.

### Removal
If installed manually - just remove file, if installed with Composer - remove it with:
```bash
sudo COMPOSER_BIN_DIR=/usr/local/bin composer global remove nazar-pc/btrfs-sync-subvolumes
```
Or without `COMPOSER_BIN_DIR=/usr/local/bin` if you didn't use it during installation.

### Usage
```
Usage:
  php btrfs-sync-subvolumes /source/directory /target/directory

It will scan /source/directory for directories and will copy each subvolume to /target/directory.
So, /source/directory/sub1 will be copied as /target/directory/sub1 and so on for each subvolume (binary diffs are used if possible in order to optimize transfer).
As the result it will replicate all subvolumes from /source/directory in /target/directory so that both directories will have the same list of identical subvolumes.
Subvolumes from /target/directory that do not exist in /source/directory will be removed.

NOTE: Important assumptions:
  - all directories found in /source/directory are subvolumes (no real check exists)
  - each next subvolume (as listed by filesystem) is based on previous, otherwise instead of binary diff the whole subvolume will be copied
```

There are few ways to run script.
```bash
sudo php btrfs-sync-subvolumes
```

or mark file as executable and just
```bash
sudo ./btrfs-sync-subvolumes
```

Also you can call it with cron or in some other way:)

### What it actually does?
* treats each directory inside `/source/directory` as subvolume
* checks whether same subvolume is present in `/target/directory`, if it exists - skips subvolume
* for each next subvolume tries to send it as binary diff from previous subvolume
* if it is the first subvolume or copying using diff fails - copies full subvolume
* removes each subvolume from `/target/directory` that is not present in `/source/directory` (only if `/source/directory` is not empty, just in case something went wrong)

### Configuration
There is no configuration or options, feel free to customize script for your needs.

### License
MIT, feel free to hack it and share!

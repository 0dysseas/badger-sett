# Badger Sett

> A *sett* or set is a badger's den which usually consists of a network of tunnels
  and numerous entrances. Setts incorporate larger chambers used for sleeping or
  rearing young.

This script is designed to raise young Privacy Badgers by teaching them a little
about the trackers on popular sites, thus preparing them to fight trackers out
in the wild. `crawler.py` visits the top 2,000 sites of the Majestic Million
with a fresh version of Privacy Badger installed and saves the `action_map` and
`snitch_map` it learns in `results.json`.

## Setup

0. Prerequisites: have [docker](https://docs.docker.com/install/) installed.
   Make sure your user is part of the `docker` group so that you can build and
run docker images without `sudo`. You can add yourself to the group with

```
$ sudo usermod -aG docker $USER
```

1. Clone the repository

```
$ git clone https://github.com/efforg/badger-sett
```

2. Run a scan 

```
$ ./runscan.sh
```

This will run a scan with the latest version of Privacy Badger's master branch and won't commit the results.

To run the script with a different branch of privacy badger, set the `PB_BRANCH` variable. e.g.

```
$ PB_BRANCH=my-feature-branch ./runscan.sh
```

3. Monitor the scan

After the script outputs "Running scan in Docker...", you can monitor the
scan's progress with:

```
$ tail -f docker-out/log.txt
```

### Automatic crawling

To set up the script to run periodically and automatically update the
repository with its results:

1. Create a new ssh key with `ssh-keygen`. Give it a name unique to the
   repository.

```
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/USER/.ssh/id_rsa): /home/USER/.ssh/id_rsa_badger_sett
```

2. Add the new key as a deploy key with R/W access to the repo on Github.
   https://developer.github.com/v3/guides/managing-deploy-keys/

3. Add a SSH host alias for Github that uses the new key pair. Create or open
   `~/.ssh/config` and add the following:

```
Host github-badger-sett
  HostName github.com
  User git
  IdentityFile /home/bennett/.ssh/id_rsa_badger_sett
```

4. Configure git to connect to the remote over SSH. Edit `.git/config`:

```
[remote "origin"]
  url = ssh://git@github-badger-sett:/efforg/badger-sett
```

This will have `git` connect to the remote using the new SSH keys by default.

5. Create a cron job to call `runscan.sh` once a day. Set the environment
   variable `GIT_PUSH=1` to have the script automatically commit and push
   `results.json` when a scan finishes. Here's an example `crontab` entry:

```
0 0 * * *  GIT_PUSH=1 /home/user/badger-sett/runscan.sh
```

6. If everything has been set up correctly, the script should push a new version
   of `results.json` after each crawl. Soon, whenever you `make` a new version of
   Privacy Badger, it will pull the latest version of the crawler's data and
   ship it with the new version of the extension.

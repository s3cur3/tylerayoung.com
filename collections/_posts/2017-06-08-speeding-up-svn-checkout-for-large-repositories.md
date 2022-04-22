---
title: Speeding Up SVN Checkout for Large Repositories
categories: ['Version Control']
authors: ['tyler']
layout: post
---

After [moving our gigantic SVN repo to a new server](/2017/06/07/migrating-a-large-svn-repository-to-a-new-server/), we wanted to speed it up. Note that some of these recommendations are peculiar to using the `svn+ssh://` protocol. If you’re serving SVN via Apache or something, you might need very different advice.

Here are all the things we changed on the server to speed up SVN. Note that these are in no particular order… it’s hard to say what will give you the biggest bang for your buck. That said, all of these are pretty cheap gags, so if checkout time is a priority, you might as well try them all!

1.  Ensure you’re running the latest version of Subversion. At the time of this writing, that means v1.9, which offers loads of performance improvements over v1.6 that we were running!
2.  Ensure you’re only checking out the files you really need. If you can get by checking out only some directories in the repo, rather than the whole thing, you might consider doing so. (For our usage—we only store art assets in SVN—this is just fine, because there are no real dependencies between subdirectories.)
3.  Set up a cron job to periodically run `svnadmin pack` on your repository as a way of reducing fragmentation of your repository’s shards.
4.  Upgrade to hosting the repo on an SSD. We found we were I/O bound by our spinning rust hard drives.
5.  Ensure your uplink speed is reasonable. Doing all of the above, we were saturating our old-as-hell 10 Mbit uplink (pushing 1.25 MB/sec isn’t hard off an SSD!).
6.  Try disabling compression (we do so in our `svnserve` wrapper script, seen below). By default, SVN uses compression level 5 (on a scale from 0 to 9). If you primarily store binary files that don’t benefit from compression, or you have a fast connection, this might be a win. In our case, our server’s CPU was pegged at 100% during a checkout; dropping compression removed the CPU as a bottleneck.
7.  If you’re using the `svn+ssh://` protocol:
    1.  Ensure you’re running the latest versions of OpenSSH and OpenSSL. Aside from being security holes, old versions have serious performance issues. (I’m ashamed to say our SVN server was running a 6 year old version of both tools! This is what we get for not having anyone “own” the maintenance on this server.)
    2.  In your sshd configuration file (for our Ubuntu installation, this was located at `/etc/ssh/sshd_config`), disallow “old and busted” ciphers. If your server defaults to 3DES, it’s another security risk plus performance disaster. (It’s quite a bit slower than AES, which typically benefits from hardware acceleration.) You should have a line in the file that looks like this:  
        `Ciphers aes128-gcm@openssh.com,aes128-ctr,aes256-gcm@openssh.com,aes256-ctr,aes192-ctr`  
        Don’t forget to restart your SSH server after making the change!
8.  If you’re using the `svn://` protocol: Bump the size of SVN’s in-memory cache. It defaults to something like 16 MB, but if you’re running a dedicated server on halfway reasonable hardware, you can allocate way more than that. To do this, you’ll have to be using an `svnserve` wrapper. That is, you’ll have to have a custom shell script—something like `/usr/local/bin/svnserve`—that modifies the arguments that `ssh+svn`\-connected users pass to `svnserve`. Something like this:
    
    ```sh
    #!/bin/sh
    # Set the umask so files are group-writable
    umask 002
    # Call the 'real' svnserve, also passing in the default repo location
    # Use a 4096 MB in-memory cache size, allowing both deltas and full texts of commits to be cached
    exec /usr/bin/svnserve "$@" --compression 0 --memory-cache-size=4096 --cache-txdeltas yes --cache-fulltexts yes
    ```

9.  If all of the above isn’t enough, it may be because you have a large number of files in your directories. (Modern OSes get less efficient as you get to an extremely large number of files.) You might consider [re-sharding your repo](https://stackoverflow.com/questions/4034888/reshard-existing-large-svn-repository), but note that this will take a _long_ time (I’ve heard to expect 1 hour per GB to dump, then another hour per GB to reload).

After making the above changes (_except_ re-sharding) on our `svn+ssh://` server, we were able to go from an average download speed of about 100 kilobytes/second to about 6 megabytes/secon—not bad at all!

---
title: Default profile
legacyUrl: https://docs.ipfs.io/guides/examples/default-profile/
description: Your profile defines which file-system and data-store your IPFS node will use, along with other configuration options. Learn how to set, change, and reset your profile.
---

# Configure a default profile and switching between profiles

The profile of your IPFS node allows you to specify which file-system or data-store you want to use. Changing these options will affect the performance of your node. For example, if you want to have a faster datastore for your node, you can use the `badgerds` profile. If you're running your node on a low-power device like a Raspberry Pi, you can use the `lowpower` profile. Profiles have been developed to customize your IPFS node to perform best under given conditions.

## Find your current profile

Your IPFS profile is found within your node's `config` file. The default location for the `config` file is `~/.ipfs/config`. If you have set an `$IPFS_PATH` variable, you can find your `config` file at `$IPFS_PATH/config`. Use `grep` to find the currently set profile:

```bash
cat ~/.ipfs/config | grep "prefix"

> "prefix": "flatfs.datastore",
> "prefix": "leveldb.datastore",
```

IPFS uses the `flatfs` profile by default, which in turn uses LevelDB internally. That's why you see `leveldb.datastore` in the command output, even though both prefixes refer to the `flatfs` datastore in this case.

If you previously configured your IPFS node to use another profile, let's say `badgerds`, the above command output would be slightly different:

```bash
"prefix": "badger.datastore",
```

## Available profiles

Here's a list of all the profiles available for your IPFS node:

### Available only when initializing the node:
- `flatfs` - the most tested datastore. Stores each block as a separate file. Use when you want a simple and reliable datastore, need garbage collection on a small datastore (<= 10GiB), or you're concerned about memory. The default datastore.
- `badgerds`- the fastest datastore. Use when performance is critical. Will not properly reclaim space when your datastore is smaller than several gigabytes. Uses up to several gigabytes of memory.
- `default-datastore` - configures the node to use `flatfs`. <!-- Since this is available only upon initialization, why don't they just stick with the default?  When would they need this?-->

### Available at any time: <!-- I tried to group these in a logical order. Did I get it right? -->
- `lowpower` - Reduces daemon overhead on the system. May affect node functionality. Performance of content discovery and data fetching may be degraded.
- `server`- disables local host discovery. Use when running IPFS on machines with public IPv4 addresses.
- `local-discovery` - sets default values to fields affected by the server profile; enables discovery in local networks.
- `test` - reduces external interference of the IPFS daemon. Use to run the daemon in test environments.
- `default-networking` - restores default network settings after using the test profile. <!-- correct interpretation of "inverse of test profile"? -->
- `randomports` - provides a random port number for a Docker swarm (cluster of nodes). <!-- Docker? -->

See [Configure a Node](./configure-node.md) for more context. <!-- What is the proper format to a page within our doc? I looked all over for an example, and haven't found one so far. The format I have goes to the github page and I see many links within the ipfs doc doing the same. It's within how-to, so only one dot, yes? Include .md or not?-->

## Reset your profile

To reset the profile of your node to default, run the following command:

```diff
ipfs config profile apply default-datastore

...
"Datastore": {
    "BloomFilterSize": 0,
    "GCPeriod": "1h",
    "HashOnRead": false,
    "Spec": {
-       >> "child": {"path":"badgerds","syncWrites":false,"truncate":true,"type":"badgerds"},
+       << "mounts": [{"child":{"path":"blocks","shardFunc":"/repo/flatfs/shard/v1/next-to-last/2","sync":true,"type":"flatfs"},"mountpoint":"/blocks","prefix":"flatfs.datastore","type":"measure"},{"child":{"compression":"none","path":"datastore","type":"levelds"},"mountpoint":"/","prefix":"leveldb.datastore","type":"measure"}],
-       >> "prefix": "badger.datastore",
        <> "type": "measure"
        ** "type": "mount"
    },
    "StorageGCWatermark": 90,
    "StorageMax": "10GB"
},
...
```

The above command shows the difference between your existing IPFS configuration and the new configuration. It will overwrite your IPFS `config` file, so be sure to create a backup of your existing IPFS configuration before running this command.

## Converting profiles

Not all profiles are compatible with each other, because they may use different technologies for storing the data inside the datastores. For instance, if you want to convert `badgerds` to `default-datastore`, you have to use another helper tool called [ipfs-ds-convert](https://dist.ipfs.io/#ipfs-ds-convert) to convert the datastore to the required format. Please follow the instructions given below to install `ipfs-ds-convert` for your operating system.

### MacOS

Download the tarball for MacOS, extract the contents, and move the binary file to your path:

```bash
wget -O /tmp/ipfs-ds-convert.tar.gz https://dist.ipfs.io/ipfs-ds-convert/v0.5.0/ipfs-ds-convert_v0.5.0_darwin-amd64.tar.gz
sudo tar -xzvf /tmp/ipfs-ds-convert.tar.gz -C /usr/local/bin/ --strip-components=1
sudo chmod +x /usr/local/bin/ipfs-ds-convert
rm /tmp/ipfs-ds-convert.tar.gz
```

### Linux

Download the tarball for Linux, extract the contents, and move the binary file to your path:

```bash
wget -O /tmp/ipfs-ds-convert.tar.gz https://dist.ipfs.io/ipfs-ds-convert/v0.5.0/ipfs-ds-convert_v0.5.0_linux-amd64.tar.gz
sudo tar -xzvf /tmp/ipfs-ds-convert.tar.gz -C /usr/local/bin/ --strip-components=1
sudo chmod +x /usr/local/bin/ipfs-ds-convert
rm /tmp/ipfs-ds-convert.tar.gz
```

### Windows

Download the zip file, extract it and then add the path to `ipfs-ds-convert.exe` to your environment path:

- Download the zip package from here: [ipfs-ds-convert](https://dist.ipfs.io/ipfs-ds-convert/v0.5.0/ipfs-ds-convert_v0.5.0_windows-amd64.zip) and extract it.
- Add the full path to `ipfs-ds-convert.exe` to your environment variables path.

To find more about `ipfs-ds-convert` please visit here: [ipfs-ds-convert](https://dist.ipfs.io/#ipfs-ds-convert).
Once you are done with the installation process, verify that `ipfs-ds-convert` has been installed successfully by executing the following command:

```bash
ipfs-ds-convert --version

> ipfs-ds-convert version 0.5.0
```

If the above command does not display a similar output, that means there is some issue with the installation. The most common issue is that the path to the executable binary is not in your environment path.

On the other hand, if the command executes successfully, then proceed to convert your IPFS profile. Run the following command to begin the process of converting your existing datastore to the required format:

```bash
ipfs-ds-convert convert

> convert 2020/12/06 21:27:26 Checks OK
> convert 2020/12/06 21:27:27 Copying keys, this can take a long time
> copied 2002 keys
> convert 2020/12/06 21:29:01 All data copied, swapping repo
> convert 2020/12/06 21:29:02 Verifying key integrity
> convert 2020/12/06 21:29:02 2002 keys OK
> convert 2020/12/06 21:29:02 Saving new spec
> convert 2020/12/06 21:29:02 All tasks finished
```

This can take a very long time to complete. If you are running this on a headless server we recommend you use something like `screen` or `tmux` to run this command in a persistent shell.

After the above command finishes, your IPFS node should be reset to the default profile.

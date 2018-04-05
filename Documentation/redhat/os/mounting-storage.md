# Mounting storage

Container Linux Configs can be used to format and attach additional filesystems to Container Linux nodes, whether such storage is provided by an underlying cloud platform, physical disk, SAN, or NAS system. This is done by specifying how partitions should be mounted in the config, and then using a _systemd mount unit_ to mount the partition. By [systemd convention](http://www.freedesktop.org/software/systemd/man/systemd.mount.html), mount unit names derive from the target mount point, with interior slashes replaced by dashes, and the `.mount` extension appended. A unit mounting onto `/var/www` is thus named `var-www.mount`.


Mount units name the source filesystem and target mount point, and optionally the filesystem type. *Systemd* mounts filesystems defined in such units at boot time. The following example formats an [EC2 ephemeral disk](booting-on-ec2.md#instance-storage) and then mounts it at the node's `/media/ephemeral` directory. The mount unit is therefore named `media-ephemeral.mount`.

```yaml container-linux-config
storage:
  filesystems:
    - name: ephemeral1
      mount:
        device: /dev/xvdb
        format: ext4
        wipe_filesystem: true
systemd:
  units:
    - name: media-ephemeral.mount
      enable: true
      contents: |
        [Unit]
        Before=local-fs.target
        [Mount]
        What=/dev/xvdb
        Where=/media/ephemeral
        Type=ext4
        [Install]
        WantedBy=local-fs.target
```

## Use attached storage for Docker

Docker containers can be very large and debugging a build process makes it easy to accumulate hundreds of containers. It's advantageous to use attached storage to expand your capacity for container images. Be aware that some cloud providers treat certain disks as ephemeral and you will lose all Docker images contained on that disk.

We're going to format a device as ext4 and then mount it to `/var/lib/docker`, where Docker stores images. Be sure to hardcode the correct device or look for a device by label:

```yaml container-linux-config
storage:
  filesystems:
    - name: ephemeral1
      mount:
        device: /dev/xvdb
        format: ext4
        wipe_filesystem: true
systemd:
  units:
    - name: var-lib-docker.mount
      enable: true
      contents: |
        [Unit]
        Description=Mount ephemeral to /var/lib/docker
        Before=local-fs.target
        [Mount]
        What=/dev/xvdb
        Where=/var/lib/docker
        Type=ext4
        [Install]
        WantedBy=local-fs.target
    - name: docker.service
      dropins:
        - name: 10-wait-docker.conf
          contents: |
            [Unit]
            After=var-lib-docker.mount
            Requires=var-lib-docker.mount
```

## Creating and mounting a btrfs volume file

Container Linux uses ext4 + overlayfs to provide a layered filesystem for the root partition. If you'd like to use btrfs for your Docker containers, you can do so with two systemd units: one that creates and formats a btrfs volume file and another that mounts it.

In this example, we are going to mount a new 25GB btrfs volume file to `/var/lib/docker`. One can verify that Docker is using the btrfs storage driver once the Docker service has started by executing `sudo docker info`. We recommend allocating **no more than 85%** of the available disk space for a btrfs filesystem as journald will also require space on the host filesystem.

```yaml container-linux-config
systemd:
  units:
    - name: format-var-lib-docker.service
      contents: |
        [Unit]
        Before=docker.service var-lib-docker.mount
        RequiresMountsFor=/var/lib
        ConditionPathExists=!/var/lib/docker.btrfs
        [Service]
        Type=oneshot
        ExecStart=/usr/bin/truncate --size=25G /var/lib/docker.btrfs
        ExecStart=/usr/sbin/mkfs.btrfs /var/lib/docker.btrfs
    - name: var-lib-docker.mount
      enable: true
      contents: |
        [Unit]
        Before=docker.service
        After=format-var-lib-docker.service
        Requires=format-var-lib-docker.service
        [Mount]
        What=/var/lib/docker.btrfs
        Where=/var/lib/docker
        Type=btrfs
        Options=loop,discard
        [Install]
        RequiredBy=docker.service
```

Note the declaration of `ConditionPathExists=!/var/lib/docker.btrfs`. Without this line, systemd would reformat the btrfs filesystem every time the machine starts.

## Mounting NFS exports

This Container Linux Config excerpt mounts an NFS export onto the Container Linux node's `/var/www`.

```yaml container-linux-config
systemd:
  units:
    - name: var-www.mount
      enable: true
      contents: |
        [Unit]
        Before=remote-fs.target
        [Mount]
        What=nfs.example.com:/var/www
        Where=/var/www
        Type=nfs
        [Install]
        WantedBy=remote-fs.target
```

To declare that another service depends on this mount, name the mount unit in the dependent unit's `After` and `Requires` properties:

```yaml
[Unit]
After=var-www.mount
Requires=var-www.mount
```

If the mount fails, dependent units will not start.

## Further reading

Check the [`systemd mount` docs](http://www.freedesktop.org/software/systemd/man/systemd.mount.html) to learn about the available options. Examples specific to [EC2](booting-on-ec2.md#instance-storage), [Google Compute Engine](booting-on-google-compute-engine.md#additional-storage) and [Rackspace Cloud](booting-on-rackspace.md#mount-data-disk) can be used as a starting point.

# Customizing the SSH daemon

Container Linux defaults to running an OpenSSH daemon using `systemd` socket activation -- when a client connects to the port configured for SSH, `sshd` is started on the fly for that client using a `systemd` unit derived automatically from a template. In some cases you may want to customize this daemon's authentication methods or other configuration. This guide will show you how to do that at boot time using a [Container Linux Config][cl-configs], and after building by modifying the `systemd` unit file.

As a practical example, when a client fails to connect by not completing the TCP connection (e.g. because the "client" is actually a TCP port scanner), the MOTD may report failures of `systemd` units (which will be named by the source IP that failed to connect) next time you log in to the Container Linux host. These failures are not themselves harmful, but it is a good general practice to change how SSH listens, either by changing the IP address `sshd` listens to from the default setting (which listens on all configured interfaces), changing the default port, or both.

## Customizing sshd with a Container Linux Config

In this example we will disable logins for the `root` user, only allow login for the `core` user and disable password based authentication. For more details on what sections can be added to `/etc/ssh/sshd_config` see the [OpenSSH manual][openssh-manual].
If you're interested in additional security options, Mozilla provides a well-commented example of a [hardened configuration][mozilla-ssh-rec].

```yaml container-linux-config
storage:
  files:
    - path: /etc/ssh/sshd_config
      filesystem: root
      mode: 0600
      contents:
        inline: |
          # Use most defaults for sshd configuration.
          UsePrivilegeSeparation sandbox
          Subsystem sftp internal-sftp
          UseDNS no

          PermitRootLogin no
          AllowUsers core
          AuthenticationMethods publickey
```

### Changing the sshd port

Container Linux ships with socket-activated SSH daemon by default. The configuration for this can be found at `/usr/lib/systemd/system/sshd.socket`. We're going to override some of the default settings for this in the Container Linux Config provided at boot:

```yaml container-linux-config
systemd:
  units:
    - name: sshd.socket
      dropins:
      - name: 10-sshd-port.conf
        contents: |
          [Socket]
          ListenStream=
          ListenStream=222
```

`sshd` will now listen only on port 222 on all interfaces when the system is built.

### Disabling socket activation for sshd

It may be desirable to disable socket-activation for sshd to ensure it will reliably accept connections even when systemd or dbus aren't operating correctly.

To configure sshd on Container Linux without socket activation, a Container Linux Config file similar to the following may be used:

```yaml container-linux-config
systemd:
  units:
  - name: sshd.service
    enable: true
  - name: sshd.socket
    mask: true
```

Note that in this configuration the port will be configured by updating the `/etc/ssh/sshd_config` file with the `Port` directive rather than via `sshd.socket`.

### Further reading

Read the [full Container Linux Config][cl-configs] guide for more details on working with Container Linux Configs, including setting user's ssh keys.

## Customizing sshd after first boot

Since [Container Linux Configs][cl-configs] are only applied on first boot, existing machines will have to be configured in a different way.

The following sections walk through applying the same changes documented above on a running machine.

*Note*: To avoid incidentally locking yourself out of the machine, it's a good idea to double-check you're able to directly login to the machine's console, if applicable.

### Changing the sshd port

The sshd.socket unit may be configured via systemd [dropins](using-systemd-drop-in-units.md).

To change how sshd listens, update the list of `ListenStream`s in the `[Socket]` section of the dropin.

*Note*: `ListenStream` is a list of values with each line adding to the list. An empty value clears the list, which is why `ListenStream=` is necessary to prevent it from *also* listening on the default port `22`.

To change just the listened-to port (in this example, port 222), create a dropin at `/etc/systemd/system/sshd.socket.d/10-sshd-listen-ports.conf`

```
# /etc/systemd/system/sshd.socket.d/10-sshd-listen-ports.conf
[Socket]
ListenStream=
ListenStream=222
```

To change the listened-to IP address (in this example, 10.20.30.40):

```
# /etc/systemd/system/sshd.socket.d/10-sshd-listen-ports.conf
[Socket]
ListenStream=
ListenStream=10.20.30.40:22
FreeBind=true
```

You can specify both an IP and an alternate port in a single `ListenStream` line. IPv6 address bindings would be specified using the format `[2001:db8::7]:22`.

*Note*: While specifying an IP address is optional, you must always specify the port, even if it is the default SSH port. The `FreeBind` option is used to allow the socket to be bound on addresses that are not yet configured on an interface, to avoid issues caused by delays in IP configuration at boot. (This option is required only if you are specifying an address.)

Multiple ListenStream lines can be specified, in which case `sshd` will listen on all the specified sockets:

```
# /etc/systemd/system/sshd.socket.d/10-sshd-listen-ports.conf
[Socket]
ListenStream=
ListenStream=222
ListenStream=10.20.30.40:223
FreeBind=true
```

### Activating changes

After creating the dropin file, the changes can be activated by doing a daemon-reload and restarting `sshd.socket`

```
$ sudo systemctl daemon-reload
$ sudo systemctl restart sshd.socket
```

We now see that systemd is listening on the new sockets:

```
$ systemctl status sshd.socket
● sshd.socket - OpenSSH Server Socket
   Loaded: loaded (/etc/systemd/system/sshd.socket; disabled; vendor preset: disabled)
   Active: active (listening) since Wed 2015-10-14 21:04:31 UTC; 2min 45s ago
   Listen: [::]:222 (Stream)
           10.20.30.40:223 (Stream)
 Accepted: 1; Connected: 0
...
```

And if we attempt to connect to port 22 on our public IP, the connection is rejected, but port 222 works:

```
$ ssh core@[public IP]
ssh: connect to host [public IP] port 22: Connection refused
$ ssh -p 222 core@[public IP]
Container Linux by CoreOS stable (1353.8.0)
core@machine $
```

### Disabling socket-activation for sshd

Simply mask the systemd.socket unit:

```
# systemctl mask --now sshd.socket
```

Finally, restart the sshd.service unit:

```
# systemctl restart sshd.service
```

### Further reading on systemd units

For more information about configuring Container Linux hosts with `systemd`, see [Getting Started with systemd](getting-started-with-systemd.md).


[openssh-manual]: http://www.openssh.com/cgi-bin/man.cgi?query=sshd_config
[mozilla-ssh-rec]: https://wiki.mozilla.org/Security/Guidelines/OpenSSH#Modern_.28OpenSSH_6.7.2B.29
[cl-configs]: provisioning.md
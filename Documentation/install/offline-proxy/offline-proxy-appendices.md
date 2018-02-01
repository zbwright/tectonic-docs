# Tectonic Offline Proxy Appendices

## Appendix A: Anchoring a certificate authority manually

To manually anchor a certificate authority within Container Linux we will perform four steps:
1. Copy the certificate to the host
2. Derive the hash of the certificate
3. Create a symbolic link from the certificate to its hash
4. Copy the certificate into the host certificate bundle

Copying of the certificate to the host can occur through the use of an SSH client (scp, sftp, etc) or through the combination of copy/paste with a text editor. For our example, we will use use copy/paste with a text editor as it minimizes the complexity of transferring the data.

To begin, copy the text form of the certificate from its source location. The Tectonic Offline Proxy makes this data available both in its startup logs as well as on the filesystem (this requires creation of a "volume" which can contain the saved cert or a bind mount back to the host filesystem outside of the container.)

```
$ docker logs tectonic-offline-proxy
Performing directory setup
Creating cache directories
2018/01/09 22:14:50| Set Current Directory to /data/cache/top/
2018/01/09 22:14:50| Creating missing swap directories
2018/01/09 22:14:50| /data/cache/top exists
2018/01/09 22:14:50| Making directories in /data/cache/top/00
2018/01/09 22:14:50| Making directories in /data/cache/top/01
2018/01/09 22:14:50| Making directories in /data/cache/top/02
2018/01/09 22:14:50| Making directories in /data/cache/top/03
2018/01/09 22:14:51| Making directories in /data/cache/top/04
2018/01/09 22:14:51| Making directories in /data/cache/top/05
2018/01/09 22:14:51| Making directories in /data/cache/top/06
2018/01/09 22:14:51| Making directories in /data/cache/top/07
2018/01/09 22:14:51| Making directories in /data/cache/top/08
2018/01/09 22:14:51| Making directories in /data/cache/top/09
2018/01/09 22:14:51| Making directories in /data/cache/top/0A
2018/01/09 22:14:51| Making directories in /data/cache/top/0B
2018/01/09 22:14:51| Making directories in /data/cache/top/0C
2018/01/09 22:14:51| Making directories in /data/cache/top/0D
2018/01/09 22:14:51| Making directories in /data/cache/top/0E
2018/01/09 22:14:51| Making directories in /data/cache/top/0F
Starting squid process with the following CA
-----BEGIN CERTIFICATE-----
MIIFNjCCAx6gAwIBAgIEeDrWpTANBgkqhkiG9w0BAQsFADA0MTIwMAYDVQQDDClD
b3JlT1MgUHJveHkgU2VydmVyIENlcnRpZmljYXRlIEF1dGhvcml0eTAeFw0xNzEy
MTkyMjM1NThaFw0xOTEyMTkyMjM1NThaMDQxMjAwBgNVBAMMKUNvcmVPUyBQcm94
eSBTZXJ2ZXIgQ2VydGlmaWNhdGUgQXV0aG9yaXR5MIICIjANBgkqhkiG9w0BAQEF
AAOCAg8AMIICCgKCAgEAvcSghpzJlI6RNGKJnFPt0MAgzKtTWp+xItvQaZMMiDSZ
OSRl+RqdBEXARQVW7Y2ydPEOUDCkMUSbgqPWm1TwJ1JsjnOK1Bnow3ezlgp9cAh1
Ov4CC/4uDiSuTSg5QBy3DFbIYai7wMMXMAT4AnVLKJr57u9Z2R7H2GRxXVqX5UC/
4/fAvOf2DgJUUqpVjZThj9uueEYfAhe30SgXWQfgvhx61GAaRijX4BTfOkUj3zQ5
ZozyRx3HO0ZwitBjcEmqNfHrUTUXKAyARmFjbowjbRYHfaBAZjFzi0I4j+sb19lJ
c/4tPZbWHRjucu2rBowZn4WlAz21QquhkJHJqt/y522WT0nonltHHQGwJ6PJzZIn
uTnlYNGlO/eognNnvXhpbJQst37cxEjx5rwfzDRjAzP82B1/N7xfAnJOk3s8W0rA
Rx6IsDh0njUsZOD/stul1hFQXVB+GCS77m9qS4FnpxDjzePIcz2rqR57/FfWYZRr
dSP7AYXp7IPcohTXw1G9RgWW4HPul+lNxy4+FnjDRdhCeIRxdSdfZ27WcA3GjBW8
4OPisL0kiwuDSdCjJgXSEhrfu8hJe2Y/ATQ7Nv91Q9qd63YeUQnBhoMcnWhjPtho
NtlXCbhqydj13IrTSjWYk7NsC99nNTmQkkyAJ2Lknmg/Y7/tupFuwh22WjRGJcsC
AwEAAaNQME4wHQYDVR0OBBYEFLOBvT98kNAweaC+1b2VhqHw2PI3MB8GA1UdIwQY
MBaAFLOBvT98kNAweaC+1b2VhqHw2PI3MAwGA1UdEwQFMAMBAf8wDQYJKoZIhvcN
AQELBQADggIBABHutaXamhEWI9WEALTd5uLL1IRj3pmckiubuFLfclJocRL4zKdH
BYad8vf7E0lRH7OSRU87e4qjw8S4Mp0kBXDeQW295rnPl4xqTn9cfD66yPhJ2qMd
oT7LmX+eVCeXELDeuw+lzkTjeWgbvAUBb0BVk/jF4qgxuAet7f4ayY87BABr/LVe
DB+EJzn0KUuaCauriL7O+3DZxYi+T+4kyOwDvePdfxureG9dz31uPye0kc4P6VAT
Au9O9Gz3yUOKR0mMFcCyCxcBpIkCLrVVT+nfdftpJX56dWbkrJsCTEu48HGaQZuv
f+xgqTBJY6SS0pgJMnnngNAtHKXXOOMmzWhjhRj/br5T9Y8E33dMkirD9uaalqPL
4TBufQSTze1trG8efPSRV0QQoFF2JD/4JqJrxc3PSBInDA4rKb+az/GR4674vx3O
xz3M8OtNcPy3Tn833HL/w3BXuCi83vBxAczK7N5shMuD55uQR0lEMr9deftzvY8S
2EuYIHBtnEifsw21+r682T9kOiqanx8NVt8mtvXgm28W28jtiqTlqZDKmkYwMvsp
SSA3pTsZeKDY62sXSLrSTADXa+UWN324qsNO9rgmKZu/TqqR9CpSBv/7we77hzop
v9VnekZLcfxG7j6qqTu5XBr5rsTI7Mo4d8+8mAQKmRYOSibaCkdzYVSo
-----END CERTIFICATE-----
...
```

Here, we see the public cert which is being used by Tectonic Offline Proxy, copy it to your system clipboard.

On the host where you wish to anchor the cert, edit the file `/etc/ssl/certs/tectonic-offline-proxy.pem`:

```
$  sudo vim /etc/ssl/certs/tectonic-offline-proxy.pem
```

Paste in the certificate and then press `esc`, then `:wq` to exit the editor, saving the file.

Next, verify that the certificate is readable and that we can derive the hash:

```
$  openssl x509 -noout -hash -in /etc/ssl/certs/tectonic-offline-proxy.pem
296964d6
```

Using the derived hash (296964d6 in our example), create a symbolic link from the certificate to the hash (ending in ".0") as the target file:

```
$ sudo ln -s /etc/ssl/certs/tectonic-offline-proxy.pem /etc/ssl/certs/296964d6.0
```

This process will anchor the certificate for use by any applications on the host which utilize OpenSSL. For applications which do not use OpenSSL (e.g. applications written in Go like Docker) certs are read in from the system "bundle" located at `/etc/ssl/certs/ca-certificates.crt`

Repackaging this certificate file requires breaking the symbolic link which is in place by default and then appending our certificate authority to this list. This can be done as follows:

```
$ sudo rm /etc/ssl/certs/ca-certificates.crt
$ cat /usr/share/ca-certificates/ca-certificates.crt \
/etc/ssl/certs/tectonic-offline-proxy.pem | sudo tee /etc/ssl/certs/ca-certificates.crt
```

This command catenates the contents of both `/usr/share/ca-certificates/ca-certificates.crt` as well as `/etc/ssl/certs/tectonic-offline-proxy.pem` and saves the resulting output to `/etc/ssl/certs/ca-certificates.crt`.

Finally, any Go based utilities which operate as long running services will need to be restarted as they only read the system certificate bundle on startup of the application.

## Appendix B: Manual configuration of clients

After provisioning the Tectonic Offline Proxy server, clients will need to be configured to leverage this service. The configuration requires two steps (both of which are trivial using Container Linux Configs or Ignition Sequences):
* configuration of the environment definitions which will be leveraged by individual services on the client system and
* installation of the Tectonic Offline Proxy certificate authority.

### Configuration of applications to utilize Tectonic Offline Proxy

Configuration of services on the host to direct traffic to the Tectonic Offline Proxy server consists of supplying the following variables to the application environment(s):

* http_proxy
* https_proxy
* no_proxy
* HTTP_PROXY
* HTTPS_PROXY
* NO_PROXY

It should be noted that, unfortunately, there is no standard used within applications for the capitalization of these variables. This is the reason for supplying both lowercase and uppercase versions each variable. For end users on the command line this can be achieved simply with the "export" Bourne-Again Shell built-in command.

With the intention of making this as easy as possible to deploy users may optionally supply these variables system wide to all units running on the host. Making this change can be done in a single place: `/etc/systemd/system.conf`. For the sake of simplicity, we will use the "drop-in" capability to have these changes tracked as one file which can easily be copied to the host. The following contents would be placed into the file `/etc/systemd/system.conf.d/tectonic-offline-proxy.conf`.

```
[Manager]
DefaultEnvironment="http_proxy={{.proxy_endpoint}}"
DefaultEnvironment="HTTP_PROXY={{.proxy_endpoint}}"
DefaultEnvironment="https_proxy={{.proxy_endpoint}}"
DefaultEnvironment="HTTPS_PROXY={{.proxy_endpoint}}"
DefaultEnvironment="no_proxy={{.proxy_endpoint}}"
DefaultEnvironment="NO_PROXY={{.proxy_endpoint}}"
```

In this case the value "{{.proxy_endpoint}}" should be changed to the DNS hostname or IP address of the server running Tectonic Offline Proxy.

### Installation of the Tectonic Offline Proxy certificate authority

Within Container Linux X.509 certificate authorities (CAs) which should be considered trusted for all applications on a host need to be "anchored." The process of anchoring a CA comprises deriving the hash of the certificate and then placing that cert into a well known location so that it can be discovered and referenced by the system SSL/TLS libraries. A full description of a manual process explaining how this works can be found in [Appendix A][appendix-a].

Identify the hash of our Tectonic Offline Proxy certificate authority. We will need this information in later steps:

```
$  openssl x509 -noout -hash -in /opt/top/cert/tectonic-offline-proxy.pem
296964d6
```

Next we need to copy the file to the host which will use Tectonic Offline Proxy to retrieve the assets to install Tectonic. To achieve this, the file can be manually supplied (as in [Appendix A][appendix-a]) or can be supplied through automated means, ensuring that all hosts have a consistent configuration.

We include the contents of the Tectonic Offline Proxy certificate authority and the hash in the following Container Linux Config snippet. In this example, replace all instances of "{{.proxy_endpoint}}" with the hostname or IP address of the system running Tectonic Offline Proxy and port (e.g. `top.example.com:8443`).

```
---
systemd:
  units:
    - name: ca-firstboot.service
      enable: true
      contents: |
        [Unit]
        DefaultDependencies=no
        Wants=systemd-tmpfiles-setup.service clean-ca-certificates.service
        After=systemd-tmpfiles-setup.service clean-ca-certificates.service
        Before=sysinit.target
        ConditionPathIsReadWrite=/etc/ssl/certs
        ConditionFirstBoot=True

        [Service]
        Type=oneshot
        ExecStart=/usr/sbin/update-ca-certificates
        [Install]
        WantedBy=multi-user.target
storage:
  files:
    - path: /etc/ssl/certs/ca-certificates.crt
      filesystem: root
      mode: 0755
      contents:

    - path: /etc/systemd/system.conf.d/tectonic-offline-proxy.conf
      filesystem: root
      mode: 0644
      contents:
        inline: |
          [Manager]
          DefaultEnvironment="http_proxy={{.proxy_endpoint}}" "HTTP_PROXY={{.proxy_endpoint}}" "https_proxy={{.proxy_endpoint}}" "HTTPS_PROXY={{.proxy_endpoint}}" "no_proxy={{.proxy_endpoint}}"

    - path: /etc/ssl/certs/tectonic-offline-proxy.pem
      filesystem: root
      mode: 0644
      contents:
        inline: |
          -----BEGIN CERTIFICATE-----
          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
          -----END CERTIFICATE-----

  links:
    - path: /etc/ssl/certs/296964d6.0
      filesystem: root
      target: /etc/ssl/certs/tectonic-offline-proxy.pem

```

Add this Container Linux Configuration snippet to the configuration which will be used to bring up the client machines. The use of the Container Linux Configuration snippet will allow for the automated configuration of use of Tectonic Offline Proxy during the process of machine provisioning.

It should be noted that, unfortunately, there is no standard used within applications for the capitalization of these variables. This is the reason for supplying both lowercase and uppercase versions each variable. For end users on the command line this can be achieved simply with the "export" Bourne-Again Shell built-in command.

With the intention of making this as easy as possible to deploy users may optionally supply these variables system wide to all units running on the host. Making this change can be done in a single place: `/etc/systemd/system.conf`. For the sake of simplicity, we will use the "drop-in" capability to have these changes tracked as one file which can easily be copied to the host. The following contents would be placed into the file `/etc/systemd/system.conf.d/tectonic-offline-proxy.conf`.

```
[Manager]
DefaultEnvironment="http_proxy={{.proxy_endpoint}}"
DefaultEnvironment="HTTP_PROXY={{.proxy_endpoint}}"
DefaultEnvironment="https_proxy={{.proxy_endpoint}}"
DefaultEnvironment="HTTPS_PROXY={{.proxy_endpoint}}"
DefaultEnvironment="no_proxy={{.proxy_endpoint}}"
DefaultEnvironment="NO_PROXY={{.proxy_endpoint}}"
```

In this case the value "{{.proxy_endpoint}}" should be changed to the DNS hostname or IP address of the server running Tectonic Offline Proxy.


[quay-offline]: https://quay.io/repository/coreosinc/tectonic-offline-proxy
[appendix-b]: offline-proxy-appendices.md#appendix-b-manual-configuration-ofclients
[appendix-a]: offline-proxy-appendices.md#appendix-a-anchoring-a-certificate-authority-manually

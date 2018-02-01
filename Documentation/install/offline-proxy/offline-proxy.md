# Tectonic Offline Proxy

The Tectonic Offline Proxy (TOP) is an administrative tool allowing for an administrator to speed up the deployment of the Tectonic Kubernetes distribution while minimizing the number of hosts which need to egress to the internet.

The installation of Tectonic is achieved through the retrieval of a number of container images from Quay.io and Google Container Registry (GCR). As production scale container registries employ heavy use of object storage technologies (Amazon Simple Storage Service / Google Storage) and content distribution networks (Amazon Cloudfront / Akamai / etc), it becomes near impossible to effectively filter based on IP address ranges. It is for this reason that having a system which can filter traffic based on URIs is paramount. Tectonic Offline Proxy meets this need while providing additional functionality.

The Tectonic Offline Proxy provides a combination of URL filtering, intelligent caching, and preemptive loading of the resources needed to successfully install a cluster of hosts. With these capabilities in place administrators and security engineers have a network location which can be monitored for aberrant activity while providing users of Tectonic a seamless experience.

## Requirements

To complete the installation users will need to have a dedicated node. This node will require at minimum 1GB of RAM and 40GB of disk space.

## Example Topologies

The Tectonic Offline Proxy (TOP) can operate in two modes: "DMZ" and "SCIF" (pronounced "skiff"). By default, the software will operate in "DMZ" mode. In both cases the client configuration is identical, requiring only changes to TOP in the event that the mode of operation needs to change.

### DMZ Mode

In DMZ mode it is accepted that the Tectonic Offline Proxy will be the only node in the environment which is allowed to make egress connections to the internet. The endpoints which will be allowed contact are restricted through a series of whitelisting filters which operate on the URIs requested on behalf of clients. Through this mechanism administrators will have a high degree of assurance that no communications are happening with unintended endpoints, regardless of the administrative capabilities of a user on the cluster.

### SCIF Mode

In computing a SCIF refers to a "Sensitive Compartmented Information Facility," a facility which is operated under the directives of United States Department of Defense governing the handling of classified information. When the Tectonic Offline Proxy (TOP) is operating in SCIF mode it is assumed that there is no ability for any compute resource to make any network connection out of it's self contained environment. This drives the need for TOP to be able to be installed and operated while bringing all software which will be needed by the cluster at runtime.

In this configuration, because there is no concept of another network segment, all communications are directed to the Tectonic Offline Proxy.

## Tectonic Offline Proxy Installation

The installation of Tectonic Offline Proxy is separated into two distinct stages:
1. The acquisition of the software.
2. The instantiation and configuration of the software.

It is broken down into these two stages because step 1 only needs explicit steps when operating in SCIF mode. For users operating in DMZ mode the software can be acquired directly without the intermediate step of retrieval.

### Acquisition of the software

When operating in SCIF mode users will need to download the Tectonic Offline Proxy from a host with a connection to the internet. Once the software has been retrieved it can be copied onto removable media and then manually transferred to the host. For our directions we will utilize a USB disk (which can be run through virus scanning software as mandated by many organizations) before copying the files to the installation host and continuing with step 2.

To begin, contact your sales representative and request a license for the Tectonic Offline Proxy. This license will entitle you to support and updates. The Tectonic license does not include Tectonic Offline Proxy.

Once your organization has been entitled to use Tectonic Offline Proxy you will receive credentials which allow for the download of the software from Quay.io.

To download the software go to: [https://quay.io/repository/coreosinc/tectonic-offline-proxy][quay-offline]

(image 1)

Once there, click the download Icon beside the tag you wish to retrieve and then follow the directions:

(image 2)

At this point, the image can be exported to a TAR archive, allowing the image to be easily moved.

```
$ docker save -o tectonic-offline-proxy-v0.1.tar \
 quay.io/coreosinc/tectonic-offline-proxy:v0.1
```

Copy the created file "tectonic-offline-proxy-v0.1.tar" to your USB media and undergo any security scanning which may need to occur.

When operating in a DMZ, the software can be retrieved with:

```
$ docker login
$ docker pull quay.io/coreosinc/tectonic-offline-proxy:v0.1
```

### Instantiation and configuration of the software

The instantiation of the software can be done anywhere an administrator can run a Docker container. While it is assumed that the administrator will achieve this task using CoreOS Container Linux, these steps will work for organizations which must run alternative flavors of Linux in DMZ environments.

The container should be instantiated as a persistent service on the host. This will ensure that after any reboots (which may occur due to software patching) the service will come back online in proper working order. To achieve this goal we will utilize a systemd service unit.

On the host which will run the Tectonic Offline Proxy, create the following file:

Filename: /etc/systemd/system/tectonic-offline-proxy.service

Contents:

```
[Unit]
Description=Tectonic Offline Proxy

[Service]
Type=simple
ExecStartPre=/usr/bin/mkdir -p /opt/top/cert /opt/top/cache
ExecStart=/usr/bin/docker run --rm \
	--volume /opt/top/cert:/data/cert \
	--volume /opt/top/cache:/data/cache/top \
	--publish 8443:8443 \
	--name tectonic-offline-proxy \
	quay.io/coreosinc/tectonic-offline-proxy:v0.1.2

ExecStop=/usr/bin/docker stop tectonic-offline-proxy

[Install]
WantedBy=multi-user.target
```

With this unit in place, start the unit with the following command:

```
$ sudo systemctl start tectonic-offline-proxy.service
```

In order to properly cache HTTPS content, the Tectonic Offline Proxy will generate a certificate authority for use by the nodes in the cluster. This is is a publicly shareable cert which is displayed in the startup logs of the service. Retrieve a copy of this file from the logs with via the following command:

```
$ docker logs tectonic-offline-proxy
```

The file can also be displayed using the command:

```
$ cat /opt/top/cert/tectonic-offline-proxy.pem
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
```

Verify that the service is listening on port 8443 (NOTE: If there is a requirement to use a TCP port other than the default of 8443 this can be done by changing the port binding performed by Docker in the systemd unit above. If this is done the administrator will need to note this change and make it in the relevant client configuration.)

```
$ ss -ntlp | grep 8443
LISTEN     0      128          *:8443                     *:*
$ docker ps -f "name=tectonic-offline-proxy"
CONTAINER ID        IMAGE                                           COMMAND                  CREATED             STATUS              PORTS               NAMES
5f2509b2eb0b        quay.io/coreosinc/tectonic-offline-proxy:v0.1   "/usr/sbin/startup.sh"   About an hour ago   Up About an hour                        tectonic-offline-proxy
```

After completing this step, note the hostname or IP address of the system running Tectonic Offline Proxy as we will need it in the subsequent client configuration.

## Client Installation

There are two primary ways to use Tectonic Offline Proxy:
* Through the Tectonic Installer
* Manual configuration of the clients

Manual configuration of the clients is discouraged due to the large number of moving parts and the potential for user error. It is suggested that the administrator make use of the Tectonic installer with the "bare metal" platform which handles many of the intricacies of this configuration on behalf of the user.

### Configuration of the Tectonic Installer

Once the Tectonic Offline Proxy server has been started we may configure the Tectonic installer with the relevant configuration options. These options include:

* tectonic_http_proxy_address
* tectonic_https_proxy_address
* tectonic_no_proxy
* tectonic_proxy_exclusive_units
* tectonic_custom_ca_pem_list

These values within the Tectonic configuration file are implemented as general purpose knobs for users to utilize a third party proxy with the deployment of Tectonic hosts. Tectonic Offline Proxy has been designed to work as a drop in tool for the use with these options.

#### tectonic_http_proxy_address / tectonic_https_proxy_address

These options specify the IP/DNS endpoint of Tectonic Offine Proxy. If Tectonic Offline Proxy was deployed at the hostname "top.example.com" on port 8443 (the default) then these options would be configured as follows:

```
tectonic_http_proxy_address = "top.example.com:8443"
tectonic_https_proxy_address = "top.example.com:8443"
```

The endpoint for Tectonic Offline Proxy should be configured for **both values** as this directs the host to use this endpoint for both HTTP and HTTPS traffic.

#### tectonic_no_proxy

This option specifies the list of endpoints that we do not want to direct through Tectonic Offline Proxy. These

#### tectonic_proxy_exclusive_units

#### tectonic_custom_ca_pem_list

### Manual Configuration of clients

While manual configuration of clients is possible, much work has been put into the Tectonic installer to minimize potential pain for the user in the deployment process. These manual configuration steps are covered in depth in [Appendix B][appendix-b], primarily for troubleshooting purposes. Users should only rely on manual configuration in the event that they have complete inability to use the Tectonic installer.

## Upgrading Tectonic Offline Proxy

The container image for Tectonic Offline Proxy is intended to be used as a standalone appliance. As its deployment model is that of an appliance, it should require little outside of the following steps:

1. retrieve the new version to the host which is running Tectonic Offline Proxy
2. stop the container running the old version
3. start the container running the new version on the same port

As Tectonic Offline Proxy brings it's content with it, the refreshed data should be available as soon as the container completes it's startup process.


[quay-offline]: https://quay.io/repository/coreosinc/tectonic-offline-proxy
[appendix-b]: offline-proxy-appendices.md#appendix-b-manual-configuration-ofclients
[appendix-a]: offline-proxy-appendices.md#appendix-a-anchoring-a-certificate-authority-manually

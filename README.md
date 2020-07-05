# Unbound DNS Docker Image - tailored for unRAID use

## What is Unbound?

Unbound is a validating, recursive, and caching DNS resolver.
> [unbound.net](https://unbound.net/)

## Standard usage

This repository has two configurations.

1.  Standalone  
2.  Use with Stubby (DNS-over-TLS)  ([Stubby container](https://hub.docker.com/r/fdm80/stubby/))  

If you want to use Unbound in standalone fashion then you can proceed with installing the contrainer.  

If you want to use Unbound with Stubby, this setup uses two containers.  One running Stubby and another running Unbound. Unbound forwards requests that are not in its cache to the Stubby container. Stubby then performs DNS resolution over TLS.  The Stubby container should be fully setup first since the Unbound installation will search for Stubby, obtain the IP address of that container, and automatically setup Unbound to forward requests to it.

By default, both containers are configured to use Cloudflare DNS.  

## How to use - unRAID template setup
(assuming you use other DNS containers with "Network Type = Custom : br0" such as pihole or stubby)

1.  Network Type = Custom : br0  
  a.  Set your own IP address
  
2.  Port Mapping  
  a.  Name:  Host Port 1  
  b.  Host Port:  53  
  c.  Connection Type:  TCP

3.  Port Mapping  
  a.  Name:  Host Port 2  
  b.  Host Port:  53  
  c.  Connection Type:  UDP  

4.  Path/Volume Mapping  
  a.  Name:  Appdata  
  b.  Container Path:  /opt/unbound/etc/unbound/  
  c.  Host Path:  /mnt/user/appdata/unbound/  
  d.  Access Mode:  Read/Write  

Start the container to allow it to create the "/appdata/unbound/" folder.  
Stop the containter.  
Download/copy the ".conf" files to the appdata folder.  
> [Unbound .conf files (standalone - 3 files)](https://github.com/fdm1980/unbound/tree/master/unbound/)  
> [Unbound .conf files (use with stubby - 2 files)](https://github.com/fdm1980/unbound/tree/master/unbound%20(use%20with%20stubby)/)  

Restart the container.

### Serve Custom DNS Records for Local Network

While Unbound is not a full authoritative name server, it supports resolving custom entries on a small, private LAN.
In other words, you can use Unbound to resolve fake names such as your-computer.local within your LAN.

To support such custom entries using this image, you need to provide an `a-records.conf` or `srv-records.conf` file.
This conf file is where you will define your custom entries for forward and reverse resolution.

#### A records

The `a-records.conf` file should use the following format:

```
# A Record
  #local-data: "somecomputer.local. A 192.168.1.1"
  local-data: "laptop.local. A 192.168.1.2"

# PTR Record
  #local-data-ptr: "192.168.1.1 somecomputer.local."
  local-data-ptr: "192.168.1.2 laptop.local."
```

#### SRV records

The `srv-records.conf` file should use the following format:

```
# SRV records
# _service._proto.name. | TTL | class | SRV | priority | weight | port | target.
_etcd-server-ssl._tcp.domain.local.  86400 IN    SRV 0        10     2380 etcd-0.domain.local.
_etcd-server-ssl._tcp.domain.local.  86400 IN    SRV 0        10     2380 etcd-1.domain.local.
_etcd-server-ssl._tcp.domain.local.  86400 IN    SRV 0        10     2380 etcd-2.domain.local.
```

### Override default forward

You can view the configuration of upstream DNS servers in the [forward-records.conf](https://github.com/fdm1980/unbound/blob/master/unbound/forward-records.conf) file.  You can create your own configuration file and overwrite the one placed in `/appdata/unbound/forward-records.conf`.

If you are using Unbound and forwarding to Stubby, then look to that container's configuration file for the upstream DNS servers.

### Use a customized Unbound configuration

Instead of using this image's default configuration for Unbound, you may supply your own configuration (unbound.conf).

***Note:** Care has been taken in the image's default configuration to enable security options so it is recommended to use it as a guide.*

# User feedback

## Documentation

Documentation for this image is stored in the [`README.md`](https://github.com/fdm1980/unbound/blob/master/README.md).

Documentation for Unbound is available on the [project's website](https://unbound.net/).

## Acknowledgments

Many thanks to the original creator.  See original acknowledgements.
- Matthew Vance (https://github.com/MatthewVance/unbound-docker)

## Licenses
### License

Unless otherwise specified, all code is released under the MIT License (MIT). See the [repository's `LICENSE` file](https://github.com/fdm1980/unbound/blob/master/LICENSE) for details.  
(originally from https://github.com/MatthewVance/unbound-docker)

### Licenses for other components

- Docker: [Apache 2.0](https://github.com/docker/docker/blob/master/LICENSE)
- DNSCrypt server Docker image: [ISC License](https://github.com/jedisct1/dnscrypt-server-docker/blob/master/LICENSE)
- LibreSSL: [Various](http://cvsweb.openbsd.org/cgi-bin/cvsweb/src/lib/libssl/src/LICENSE?rev=1.12&content-type=text/x-cvsweb-markup)
- OpenSSL: [Apache-style license](https://www.openssl.org/source/license.html)
- Unbound: [BSD License](https://unbound.nlnetlabs.nl/svn/trunk/LICENSE)

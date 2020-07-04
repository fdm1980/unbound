# Unbound DNS Docker Image - tailored for unRAID use

## What is Unbound?

Unbound is a validating, recursive, and caching DNS resolver.
> [unbound.net](https://unbound.net/)

## Standard usage

This repository has two configurations.

1.  Standalone  
2.  Use with Stubby  ([Stubby container](https://hub.docker.com/r/fdm80/stubby/))  

If you want to use Unbound in standalone fashion then you can proceed with installing the contrainer.  

If you want to use Unbound with Stubby, this setup uses two containers.  One running Stubby and another running Unbound. Unbound forwards requests that are not in its cache to the Stubby container. Stubby then performs DNS resolution over TLS.  The Stubby container should be setup first since the Unbound installation will search for an existing container called "stubby" and obtain the IP address of that container and automatically setup Unbound to forward requests to it.

By default, both containers are configured to use Cloudflare DNS.  

### How to use - unRAID template setup
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
> [Unbound config files (standalone - 3 files)](https://github.com/fdm1980/unbound/tree/master/unbound/)  
> [Unbound config files (use with stubby - 2 files)](https://github.com/fdm1980/unbound/tree/master/unbound%20(use%20with%20stubby)/)  

Restart the container.

### Serve Custom DNS Records for Local Network

While Unbound is not a full authoritative name server, it supports resolving
custom entries on a small, private LAN. In other words, you can use Unbound to
resolve fake names such as your-computer.local within your LAN.

To support such custom entries using this image, you need to provide an
`a-records.conf` or `srv-records.conf` file.
This conf file is where you will define your custom
entries for forward and reverse resolution.

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

Once the file has your entries in it, mount your version of the file as a volume
when starting the container:

```console
docker run --name my-unbound -d \
-p 53:53/udp -p 53:53/tcp \
-v $(pwd)/a-records.conf:/opt/unbound/etc/unbound/a-records.conf:ro \
--restart=always mvance/unbound:latest
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

Run a container taht use this SRV config file:
```console
docker run --name my-unbound -d \
-p 53:53/udp -p 53:53/tcp \
-v $(pwd)/srv-records.conf:/opt/unbound/etc/unbound/srv-records.conf:ro \
--restart=always mvance/unbound:latest
```

### Override default forward

By default, forwarders are configured to use Cloudflare and CleanBrowsing DNS. You can retrieve the configuration in the [1.10.0/forward-records.conf](1.10.0/forward-records.conf) file.

You can create your own configuration file and override the one placed in `/opt/unbound/etc/unbound/forward-records.conf` in the container.

Example `forward-records.conf`:
```
forward-zone:
  # Forward all queries (except those in cache and local zone) to
  # upstream recursive servers
  name: "."

  # my DNS
  forward-addr: 192.168.0.1@53#home.local
```

Once the file has your entries in it, mount your version of the file as a volume
when starting the container:

```console
docker run --name my-unbound -d -p 53:53/udp -p 53:53/tcp -v \
$(pwd)/forward-records.conf:/opt/unbound/etc/unbound/forward-records.conf:ro \
--restart=always mvance/unbound:latest
```


### Use a customized Unbound configuration

Instead of using this image's default configuration for Unbound, you may supply your own configuration. If your customized configuration is located at `/my-directory/unbound/unbound.conf`, pass `/my-directory/unbound` as a volume when creating your container:

```console
docker run --name=my-unbound \
--volume=/my-directory/unbound:/opt/unbound/etc/unbound/ \
--publish=53:53/tcp \
--publish=53:53/udp \
--restart=unless-stopped \
--detach=true \
mvance/unbound:latest
```

This will expose all files in `/my-directory/unbound/` to the container. As an alternate way to serve custom DNS records for any local zones, either place them directly in your `unbound.conf`, or place the local zones in a separate file and use Unbound's include directive within your `unbound.conf`:

```
include: /opt/unbound/etc/unbound/local-zone-unbound.conf
```

Your volume's contents might eventually look something like this:

```
/my-directory/unbound/
-- unbound.conf
-- local-zone-unbound.conf
-- secret-zone.conf
-- some-other.conf
```

Overall, this approach is very similar to the `a-records.conf` approach described above. However, by passing your unbound directory rather than a single file, you have more options for customizing and segmenting your Unbound configuration.

***Note:** Care has been taken in the image's default configuration to enable
security options so it is recommended to use it as a guide.*

# User feedback

## Documentation

Documentation for this image is stored right here in the [`README.md`](https://github.com/MatthewVance/unbound-docker/blob/master/README.md).

Documentation for Unbound is available on the [project's website](https://unbound.net/).

## Issues

If you have any problems with or questions about this image, please contact me
through a [GitHub issue](https://github.com/MatthewVance/unbound-docker/issues).

## Contributing

You are invited to contribute new features, fixes, or updates, large or small. I
imagine the upstream projects would be equally pleased to receive your
contributions.

Please familiarize yourself with the [repository's `README.md`
file](https://github.com/MatthewVance/unbound-docker/blob/master/README.md)
before attempting a pull request.

Before you start to code, I recommend discussing your plans through a [GitHub
issue](https://github.com/MatthewVance/unbound-docker/issues), especially for
more ambitious contributions. This gives other contributors a chance to point
you in the right direction, give you feedback on your design, and help you find
out if someone else is working on the same thing.

## Acknowledgments

The code in this image is heavily influenced by DNSCrypt server Docker image,
though the upstream projects most certainly also deserve credit for making this
all possible.
- [Docker](https://www.docker.com/)
- [DNSCrypt server Docker image](https://github.com/jedisct1/dnscrypt-server-docker)
- [OpenSSL](https://www.openssl.org/)
- [Unbound](https://unbound.nlnetlabs.nl/)

## Licenses

### License

Unless otherwise specified, all code is released under the MIT License (MIT).
See the [repository's `LICENSE`
file](https://github.com/MatthewVance/unbound-docker/blob/master/LICENSE) for
details.

### Licenses for other components

- Docker: [Apache 2.0](https://github.com/docker/docker/blob/master/LICENSE)
- DNSCrypt server Docker image: [ISC License](https://github.com/jedisct1/dnscrypt-server-docker/blob/master/LICENSE)
- LibreSSL: [Various](http://cvsweb.openbsd.org/cgi-bin/cvsweb/src/lib/libssl/src/LICENSE?rev=1.12&content-type=text/x-cvsweb-markup)
- OpenSSL: [Apache-style license](https://www.openssl.org/source/license.html)
- Unbound: [BSD License](https://unbound.nlnetlabs.nl/svn/trunk/LICENSE)

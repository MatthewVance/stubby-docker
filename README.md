# Unbound and Stubby Docker Images

## What does this do?

This allows you to run a Stubby for better DNS over TLS support than Unbound provides without losing the performance benefits of having a local caching DNS resolver.

According to the [Stubby documentation](https://dnsprivacy.org/wiki/display/DP/About+Stubby):
> Unbound can be configured as a local forwarder using DNS-over-TLS to forward queries. However at the moment Unbound does not have all the TCP/TLC features that Stubby has for example, it cannot support 'Strict' mode, it cannot pad queries to hide query size and it opens a separate connection for every DNS query (Stubby will re-use connections)
>
>However, Unbound is a more mature and stable daemon and may be more reliable today. 

To achieve this, this setup uses two containers, one running Stubby and another running Unbound. Unbound exposes DNS over port 53 and forwards requests not in its cache to the Stubby container on port 8053 (not publically exposed). Stubby then performs DNS resolution over TLS. By default, this is configured to use Cloudflare DNS. 

## How to use

### Building

`sudo docker build -t mvance/stubby:latest .`

`sudo docker build -t mvance/unbound:1.8.3-stubby .`

### Standard usage

Run these containers with the following command:

```console
docker-compose up -d
```

Next, point your DNS to the IP of your Docker host running the Unbound container.

### Serve Custom DNS Records for Local Network

While Unbound is not a full authoritative name server, it supports resolving
custom entries on a small, private LAN. In other words, you can use Unbound to
resolve fake names such as your-computer.local within your LAN.

To support such custom entries using this image, you need to update the provided
[a-records.conf](https://github.com/MatthewVance/stubby-docker/blob/master/unbound/a-records.conf) file. This conf file is where you will define your custom entries for forward and reverse resolution.

The `a-records.conf` file should use the following format:

```
# A Record
  #local-data: "somecomputer.local. A 192.168.1.1"
  local-data: “laptop.local. A 192.168.1.2”

# PTR Record
  #local-data-ptr: "192.168.1.1 somecomputer.local."
  local-data-ptr: "192.168.1.2 laptop.local."
```

### Use a customized Unbound configuration
Instead of using this image's default Unbound configuration, you may supply your own unbound.conf. See my [unbound-docker README](https://github.com/MatthewVance/unbound-docker/blob/master/README.md#use-a-customized-unbound-configuration) for further details. Note, you will likely want to apply the concepts from those directions via [docker-compose.yml](https://github.com/MatthewVance/stubby-docker/blob/master/stubby/stubby.yml).

## Issues

If you have any problems with or questions about this image, please contact me through a [GitHub issue](https://github.com/MatthewVance/stubby-docker/issues).

## Contributing

You are invited to contribute new features, fixes, or updates, large or small. I imagine the upstream projects would be equally pleased to receive your contributions.

Please familiarize yourself with the [repository's `README.md` file](https://github.com/MatthewVance/stubby-docker/blob/master/README.md) before attempting a pull request.

Before you start to code, I recommend discussing your plans through a [GitHub issue](https://github.com/MatthewVance/stubby-docker/issues), especially for more ambitious contributions. This gives other contributors a chance to point you in the right direction, give you feedback on your design, and help you find out if someone else is working on the same thing.

## Acknowledgments

These deserve credit for making this all possible.

- [Docker](https://www.docker.com/)
- [DNSCrypt server Docker image](https://github.com/jedisct1/dnscrypt-server-docker)
- [Docker](https://www.docker.com/)
- [Filippo Valsorda's Gist](https://gist.github.com/FiloSottile/2b171d359232114839358a74f7df33cb)
- [Franksn's Reddit post](https://www.reddit.com/r/pihole/comments/7oyh9m/guide_how_to_use_pihole_with_stubby/)
- [OpenSSL](http://www.libressl.org/)
- [Unbound](https://www.unbound.net/)

## Licenses
### License

Unless otherwise specified, all code is released under the MIT License (MIT). See the [repository's `LICENSE` file](https://github.com/MatthewVance/stubby-docker/blob/master/LICENSE) for details.

### Licenses for other components

- DNSCrypt server Docker image: [ISC License](https://github.com/jedisct1/dnscrypt-server-docker/blob/master/LICENSE)
- Docker: [Apache 2.0](https://github.com/docker/docker/blob/master/LICENSE)
- OpenSSL: [Apache-style license](https://www.openssl.org/source/license.html)
- Stubby: [BSD-3-Clause](https://github.com/getdnsapi/getdns/blob/develop/LICENSE) 
- Unbound: [BSD License](https://unbound.nlnetlabs.nl/svn/trunk/LICENSE)

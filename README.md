# js-libp2p-i2p

[![](https://img.shields.io/badge/project-libp2p-yellow.svg?style=flat-square)](http://libp2p.io/)
[![](https://img.shields.io/badge/freenode-%23libp2p-yellow.svg?style=flat-square)](http://webchat.freenode.net/?channels=%23libp2p)
[![](https://img.shields.io/codecov/c/github/Jorropo/js-libp2p-i2p.svg?style=flat-square)](https://codecov.io/gh/Jorropo/js-libp2p-i2p)
[![](https://img.shields.io/travis/Jorropo/js-libp2p-i2p.svg?style=flat-square)](https://travis-ci.com/Jorropo/js-libp2p-i2p)
[![Dependency Status](https://david-dm.org/Jorropo/js-libp2p-i2p.svg?style=flat-square)](https://david-dm.org/Jorropo/js-libp2p-i2p)
[![js-standard-style](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/feross/standard)

[![](https://raw.githubusercontent.com/libp2p/interface-transport/master/img/badge.png)](https://github.com/libp2p/interface-transport)


> JavaScript implementation of the I2P module for libp2p. It exposes the [interface-transport](https://github.com/libp2p/interface-connection) for dial/listen. `libp2p-i2p` is a very thin shim that adds support for dialing to a `multiaddr`. This small shim will enable libp2p to use other different transports.

This module is in WIP and not even working, code is just from js-libp2p-tcp and need to be changed.

It was templated on [js-libp2p-tcp](https://github.com/libp2p/js-libp2p-tcp).

## Lead Maintainer

[Jorropo](https://github.com/Jorropo)

## Table of Contents

- [Install](#install)
  - [npm](#npm)
- [Usage](#usage)
- [Contribute](#contribute)
- [License](#license)

## Install

### npm

```sh
> npm install libp2p-i2p
```

## Usage

Currently you need to manualy setup your I2P nodes (this will be automatic when [js-i2cp][https://github.com/Jorropo/js-i2cp/] is ready).

First install an [i2p node](https://geti2p.net/).

Then when your node works (you may have to configure upnp if you are behind nat, ...)

So what we will do know ?

We will create 2 things, a service redirecting connection from I2P to your IPFS node.
And a socks client to allow your node to access i2p.

So to do that go to your i2p admin panel in section i2ptunnelmgr [localhost:7657/i2ptunnelmgr](http://localhost:7657/i2ptunnelmgr).

### I2P to IPFS
And now add a new Standard I2P service, this will be i2p to ipfs.<br/>
Standard is to mean TCP. Looks [geti2p.net/en/docs/api/i2ptunnel](https://geti2p.net/en/docs/api/i2ptunnel) for complete info.<br/>
Then fill these field :
- Name, currently name doesn't matter, but in future they will so you should choose: `IPFS[_<id for multiple ipfs in one i2p>]_i2p-ipfs`
- Description, as your choice, this is good idea to change it.
- Starting of the tunnel at start, check it, tunnel have 3 state:
  - shutdown (here nothings happend, this is just to store configuration)
  - waiting (here the tunnel is listening for new connection and start if asked, takes a bit of ram)
  - running (the tunnel is at his cruising speed running all)
   When the server start tunnel will be at shutdown and this option set it to waiting or running (depending of configuration)
- Target
  - Host, here put the ip of the server hosting ipfs (probably 127.0.0.1)
  - Port, this is the port that will be used to connect to your ipfs node, this is a special port for libp2p-i2p, 4002 is a good choice, just be sure that not all ready used
  - SSL, off, that for using SSL instead of TCP, but that use more CPU and totaly useless for a loopback connection (also libp2p-i2p) doesn't support SSL
- private key file, this file is the identity of your service/client in i2p, we can use this to whitelist some access, ... even if currently that not implented, in the future you will need to have the same indentity to send data and receive some.
  So Write it like that : `ipfs[-<id for multiple ipfs in one i2p>]-privKeys.dat`

This part is important, tunnel size and count.<br/>
**You need to choose what kind of node you would be**:
- Public node, this node connects to i2p but also on non anonymized network (network, cjdns, ...), here i2p connection is to share ipfs to other i2p users.
- Public unreachable node, this node is a public node but it can't punch his nat so other nodes can't contact it, i2p can be used as a way to dig under a nat but that slow, for that prefer [cjdns](https://github.com/cjdelisle/cjdns) or [yggdrasil](https://github.com/yggdrasil-network) to fly over your nat.
- Anonymized node, this node try to hide the best as possible, protection isn't perfect but we can try to maximize it. So this node will not have any listen except i2p (and maybe tor if you trust them).

So how to configue that ?
- For public node put 0 jumps, that means people will directly came to yours i2prouter, also 1 tunnel and 0 fallback because more tunnel is when connection is limited but connection to your self is near infinite (if bandwith is too slow this is to other user to add tunnels to reach you)
- For public unreachable node you should put 1 jumps because that will move rendez vous to some other nodes (and they will be reachable). Also now you should deal with multiple tunnel, simply more tunnel = more bandwith but more CPU and RAM usage, I personaly use 3 but try with more and if that works good for you, then fallback, this is a prebuilt tunnel to be used when a main tunnel crash, they don't cost a lot but they arn't very usefull (they are but rarely), I personaly use 1 and if 2 tunnel crash at the same time I will be running at 2 tunnel instead of 3 for a minutes (not a real problem).
- For anonymized node, same as public unreachable but with at least 3 jumps and add some change in tunnel length, I personaly use +0/2 that the best for anonymity (if you don't go in excessive level), +-0/1 is also good but use it with 4 jumps, 3 is quite short for that, never use +-0/2 except if you have 5 jumps (more than 3 start to be very heavy), also more jumps is more CPU and RAM use so less tunnel count and finaly bandwith.

**Then limit**:<br/>
Set them all to 0 (unlimited).<br/>
Maybe except maximum simultaneous connection, set it to somethings you RAM can handle (try to reduce if i2prouter use too many ram).

And finaly sig, there is only 2 good choice :
- Ed25519-SHA-512, a very spread out, very sure not perfect, doesn't deal very well versus quantic computer (may need to change in 10 years (I'm currently in 2019 while writing that)))
- Red25519-SHA-512, An I2P algorithm, should be more random than Ed25519 while doing quite the same, still experimental

Reduce activity if unused: activate if ipfs doesn't run all the time, 1 tunnel is good for a reduced activity maybe set inactivity time to 1 or 2 minutes because ipfs will normaly not stop querying nodes.

All other value should stay in their default state.

After that your service should work like that (here I have made an giant anonymized node):
![service finish screenshot](https://ipfs.io/ipfs/QmVRuVBijgR5uKSoa9jpXz6E3AR1C2Vefe1hU93XuBuL5q)

### IPFS to I2P
Now do new a socks 4/4a/5 client. (Why not a Standard like service ? Because Standard is one connection per client/service while socks 4/4a/5 allow the user to chose where to connect.)
Fill it like service except:
- Name, invert i2p and ipfs, here this ipfs to i2p
- Port, is here port of the proxy (entry point to i2p), I personaly use 4003, just take one not used.
- Accessible by, chose where network card can use this proxy, If i2p and ipfs are in the same machine (like me) select 127.0.0.1
- Persistant private key, here this is by default un checked, check it and add the same key as in the service.
- Shared tunnel, this option allows you to use the same tunnel for browsing i2p, irc chatting and more, but that not good for anonymity (allow to link your web with your node) leave it unchecked
- Representative plugin, uncheck it, that a way to browse internet if connection fail, but here we are to access i2p
- For public unreachable node you can set jumps to 0 because that will be you contacting other people.

### Building the multiaddr
We are close to the end, now we need to know what is your i2p address !<br/>
To know that simply go to tunnel mgr and find your IPFS_i2p-ipfs service, you should see an hostname ot address or destination followed by a value like that : `7caermkfqayuov6gnmr735ikdf7hdxsf43zqhmxkpyl4wvhgf3tq.b32.i2p`, copy this value, that the i2p address of your ipfs node.<br/>
You may have a `base32:` prefix, if this is the case, remove it.

Now build your listening multiaddr :
```
/i2p/manual/<listener bind>/<listener port>/<proxy address>/<proxy port>/<i2p addr>/
```
Example (coresponding with the port in my tutorial):
```
/i2p/manual/127.0.0.1/4002/127.0.0.1/4003/7caermkfqayuov6gnmr735ikdf7hdxsf43zqhmxkpyl4wvhgf3tq.b32.i2p/
```
Why `/i2p/manual/` ?<br/>
Because in the future some automatical configuration will be disponible (and maybe even some integrated i2p).

### Transport

[![](https://raw.githubusercontent.com/libp2p/interface-transport/master/img/badge.png)](https://github.com/libp2p/interface-transport)

## Contribute

Contributions are welcome! The libp2p implementation in JavaScript is a work in progress. As such, there's a few things you can do right now to help out:

- [Check out the existing issues](//github.com/libp2p/js-libp2p-i2p/issues).
- **Perform code reviews**.
- **Add tests**. There can never be enough tests.

Please be aware that all interactions related to libp2p are subject to the IPFS [Code of Conduct](https://github.com/ipfs/community/blob/master/code-of-conduct.md).

Small note: If editing the README, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.

## License

[MIT](LICENSE) © 2015-2016 David Dias

---
title: Discovery & Bootstrap Nodes
date: 2022-02-17T00:00:00+01:00
weight: 2
---

# Discovery & Bootstrap Nodes

This guide explains the discovery and bootstrap mechanisms currently available in js-waku,
their benefits and caveats and how to use them.

Node discovery is the mechanism that enables a Waku node to find other nodes.
Waku is a modular protocol, several discovery mechanisms are and will be included in Waku
so that developers can select the best mechanism(s) based for their use case and the user's environment
(e.g. mobile phone, desktop browser, server, etc).

When starting a Waku node,
it needs to connect to other nodes to be able to send, receive and retrieve messages.
Which means there needs to be a discovery mechanism that enable finding other nodes when not connected to any node.
This is called _bootstrapping_.

Once connected, the node needs to find additional peers to have:

- Enough peers in the Waku Relay mesh (target is 6),
- Enough peers in reserve, in case current peers are overloaded or go offline,
- Peers with specific Waku capabilities (e.g. Store, Light Push, Filter).

For now, we are focusing in making bootstrap discovery protocols available,
research of other discovery protocols is in progress.

## Default Bootstrap Mechanism

**The default bootstrap mechanism is [to connect to the Status' nim-waku prod fleet](#nwaku-prod-fleet)**.

To use:

```ts
const waku = await Waku.create({ bootstrap: { default: true } });
```

{{< hint info >}}
When creating a Waku node without passing the `bootstrap` option,
the node does **not** connect to any remote peer or bootstrap node.

As the current strategy is to connect to nodes operated by Status,
we want to ensure that developers consciously opt-in
while providing a friendly developer experience.

We intend to change this in the future and enable bootstrap by default
once we have implemented more decentralized strategies.
{{< /hint >}}

## Predefined Bootstrap Nodes

Addresses of nodes hosted by Status are predefined in the codebase:

https://github.com/status-im/js-waku/blob/e4024d5c7246535ddab28a4262006915d2db58be/src/lib/discovery/predefined.ts#L48

They can be accessed via the `getPredefinedBootstrapNodes` function.

**Pros:**

- Low latency,
- Low resource requirements.

**Cons:**

- Prone to censorship: node ips can be blocked,
- Limited: Static number of nodes,
- Poor maintainability: Code needs to be changed to update the list.

### Nwaku Prod Fleet

The nwaku prod fleet run the latest [nwaku](https://github.com/status-im/nim-waku/) release.
The fleet aims to provide a stable, yet not warranted, service.

```ts
import { Waku } from "js-waku";

const waku = await Waku.create({
  bootstrap: {
    peers: getPredefinedBootstrapNodes(),
  },
});
```

### Nwaku Test Fleet

The nwaku test fleet run the latest commit from [nwaku](https://github.com/status-im/nim-waku/)'s master branch.
The fleet is subject to frequent database reset,
hence messages are generally kept in store nodes for a few days at a time.

```ts
import { Waku, discovery } from "js-waku";

const waku = await Waku.create({
  bootstrap: {
    peers: getPredefinedBootstrapNodes(discovery.predefined.Fleet.Test),
  },
});
```

## Use your own nodes

Developers have the choice to run their own [nwaku](https://github.com/status-im/nim-waku/) nodes
and use them to bootstrap js-waku nodes.

There are two ways to set bootstrap nodes:

1. Using an array of multiaddrs (as `string` or `Multiaddr`):

```ts
import { Waku } from "js-waku";

const waku = await Waku.create({
  bootstrap: {
    peers: [
      "/dns4/node-01.ac-cn-hongkong-c.wakuv2.test.statusim.net/tcp/443/wss/p2p/16Uiu2HAkvWiyFsgRhuJEb9JfjYxEkoHLgnUQmr1N5mKWnYjxYRVm",
    ],
  },
});
```

2. Passing an async function that returns an array of multiaddr (as `string` or `Multiaddr`):

```ts
import { Waku } from "js-waku";

const waku = await Waku.create({
  bootstrap: {
    getPeers: async () => {
      const addresses = [];
      // Fetch the multiaddrs from somewhere...
      return addresses;
    },
  },
});
```

{{< hint info >}}
Read [Nwaku Service Node](/docs/guides/nwaku/) to learn how to run your own node.
{{< /hint >}}

**Pros & Cons**: Same than [Predefined Bootstrap Nodes](#predefined-bootstrap-nodes)

## DNS Discovery

[EIP-1459: Node Discovery via DNS](https://eips.ethereum.org/EIPS/eip-1459) has been implemented in js-waku, nwaku and go-waku
with some modification on the [ENR format](https://rfc.vac.dev/spec/31/).

DNS Discovery enables anyone to register an ENR tree in the `TXT` field of a domain name.

_ENR_ is the format used to store node connection details (ip, port, multiaddr, etc).

This enables a separation of software development and operations
as dApp developers can include one or several domain names to use for DNS discovery,
while operators can handle the update of the dns record.

It also enables more decentralized bootstrapping as anyone can register a domain name and publish it for others to use.

{{< hint warning >}}
While this method is implemented in js-waku,
it is currently not recommended to use due to a [bug](https://github.com/status-im/nim-waku/issues/845) in the websocket implementation of nwaku.

The nwaku [prod fleet](#nwaku-prod-fleet) and [test fleet](#nwaku-test-fleet) have a [websockify](https://github.com/novnc/websockify)
instance deployed alongside nwaku, acting as a work around the nwaku websocket [bug](https://github.com/status-im/nim-waku/issues/845).
{{< /hint >}}

**Pros:**

- Low latency, low resource requirements,
- Bootstrap list can be updated by editing a domain name: no code change is needed,
- Can reference to a greater list of nodes by pointing to other domain names in the code or in the ENR tree.

**Cons:**

- Prone to censorship: domain names can be blocked,
- Limited: Static number of nodes, operators must provide their ENR to the domain owner to get their node listed.

## Other Discovery Mechanisms

Other discovery mechanisms such as gossipsub peer exchange, discv5, etc are currently being research and developed.

They aim to improve the current _status quo_ in the following aspects:

- More decentralized mechanisms: Less reliance on specific entities,
- Less setup for node operators: Enabling their nodes to be discovered by connecting to bootstrap nodes,
  without having to update a domain name.

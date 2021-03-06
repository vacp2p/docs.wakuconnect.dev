---
title: FAQ
date: 2022-02-16T00:00:00+10:00
weight: 21
---

# FAQ

Frequently Asked Questions for developers using js-waku:

### 1. Why should I build a frontend only webapp (no NodeJS backend)?

Waku enables dApp to add communication, e.g. interaction between users, in a fully decentralized manner.
A webapp that uses NodeJS as a backend implies that a party runs said NodeJS software in a centralized infrastructure.

Despite using Waku & Ethereum, such webapp cannot become decentralized.

By building a frontend only webapp, that entirely runs in the browser, one can distribute the frontend code in many manners:
host it, mirror it, have it on GitHub, deploy it on IPFS, etc.
Enabling anyone to download this code and run in the browser,
making the webapp a truly decentralized dApp.

### 2. I am getting a `Module parse failed: Unexpected token` error

When using an older version of babel (used by `react-scripts`), the following error may appear when running the webapp:

```
./node_modules/multistream-select/src/ls.js 55:2
Module parse failed: Unexpected token (55:2)
File was processed with these loaders:
 * ./node_modules/babel-loader/lib/index.js
You may need an additional loader to handle the result of these loaders.
|   await pipe(protocolsReader, lp.decode(), async
|   /** @type {AsyncIterable<BufferList>} */
>   source => {
|     for await (const protocol of source) {
|       // Remove the newline
```

```
./node_modules/js-waku/build/module/lib/waku_relay/index.js 228:16
Module parse failed: Unexpected token (228:16)
File was processed with these loaders:
 * ./node_modules/react-scripts/node_modules/babel-loader/lib/index.js
You may need an additional loader to handle the result of these loaders.
|       }
|
>       meshPeers?.forEach(peer => {
|         toSend.add(peer);
|       });
```

As documented in issue [#165](https://github.com/status-im/js-waku/issues/165),
this error comes from an older babel version.
You need babel version **7.13.2** or above.

You can check your version using `npm ls`:

```
▶ npm ls @babel/preset-env
waku-pres@0.1.0
└─┬ react-scripts@4.0.3
  ├─┬ @svgr/webpack@5.5.0
  │ └── @babel/preset-env@7.12.17
  ├─┬ babel-preset-react-app@10.0.0
  │ └── @babel/preset-env@7.12.1
  └─┬ workbox-webpack-plugin@5.1.4
    └─┬ workbox-build@5.1.4
      └── @babel/preset-env@7.12.17 deduped
```

The best way to fix this is by using a more recent ReactJS stack.
This might not always possible, in this case force the installation of babel **7.14**:

```
npm i --save-dev @babel/preset-env@7.14
rm -rf node_modules package-lock.json
npm install
```

### 3. Store nodes only keep 30 days of messages by default, what if I need to keep messages permanently?

Waku store protocol at this point does not provide a scalable solution to serve a large size archival data
(as store nodes need to persist the entire history on their own local storage space),
due to this, this protocol is more suitable for serving recent historical messages.

Nevertheless, research on the scalability aspect has been done and documented in [here](https://github.com/vacp2p/research/milestone/8).

Note that it is possible for operators to run their own store node and set a higher value than the default ones:

`nwaku`:

```
--store-capacity    Maximum number of messages to keep in store. [=50000].
```

`go-waku`:

```
--store-days value      maximum number of days before a message is removed from the store (default: 30)
--store-capacity value  maximum number of messages to store (default: 50000)
```





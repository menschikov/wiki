# Listing and reading hotel details

- The address of Winding Tree index used in this example is for demo purposes only. There is only example data.

## Javascript library

### Dependencies
```
  "@windingtree/off-chain-adapter-http": "2.0.0",
  "@windingtree/off-chain-adapter-swarm": "3.1.0",
  "@windingtree/wt-js-libs": "0.2.4"
```

### Example code

```js
// 1. Load the libs
const WTLibs = require('@windingtree/wt-js-libs');
const SwarmAdapter = require('@windingtree/off-chain-adapter-swarm');
const HttpAdapter = require('@windingtree/off-chain-adapter-http');

// 2. Configure the wt-js-libs
const libs = WTLibs.createInstance({
  dataModelOptions: {
    // We are using Ropsten as testnet, our demo index is on 0x407f550023eb6ad8a4797844489e17c5ced17e06
    provider: 'https://ropsten.infura.io/',
  },
  offChainDataOptions: {
    adapters: {
      // WT is using bzz-raw protocol to access data on swarm
      'bzz-raw': {
        options: {
          swarmProviderUrl: 'https://swarm-gateways.net/',
        },
        create: (options) => {
          return new SwarmAdapter(options);
        },
      },
      // WT is not using http at the moment, we only resolve https
      https: {
        create: () => {
          return new HttpAdapter();
        },
      },
    },
  },
});

// 3. Collect the data
// This has to be in an async block due to the nature of distributed data
(async () => {
  // Get an instance of WTIndex wrapper
  const index = await libs.getWTIndex('0x407f550023eb6ad8a4797844489e17c5ced17e06');

  // We can get all hotels available in the WTIndex
  // - We will get only hotels on valid and accessible addresses
  // - Every hotel is an abstraction over on-chain and off-chain data
  // - This should be reasonably fast, because only a single method call on WTIndex is performed
  const hotels = await index.getAllHotels();

  // Notice how the approach is totally protocol agnostic
  for (let hotel of hotels) {
    // You can benefit from a recursive shorthand method that downloads all of hotel data for you
    const serializedHotel = await hotel.toPlainObject();
    // And you can access all of the data in a simple, synchronous way
    const hotelLocation = serializedHotel.dataUri.contents.descriptionUri.contents.location;
    
    
    // OR with much finer control and a lot of await calls, you can do this:
    
    // This actually fetches data from a hotel smart contract
    const hotelDataUri = await hotel.dataUri;
    // But it gets cached, so the second call is way faster
    const hotelDataUri2 = await hotel.dataUri;

    // Let's initialize the off-chain data index - but no data is downloaded yet
    const offChainData = await hotel.dataIndex;
    // Only the next command actually initiates the download of the data index document
    const hotelDescriptionUri = await offChainData.contents.descriptionUri;
    // And only now the actual contents of descriptionUri gets downloaded
    const hotelName = await hotelDescriptionUri.contents.name;
    // But this is fast, because the document is already cached in memory.
    // The properties are of course interchangeable, so the data is downloaded
    // only when any of the data properties is accessed for the first time.
    const hotelDescription = await hotelDescriptionUri.contents.description;
  }
})();
```

## REST API

- The following code expects [wt-read-api](https://github.com/windingtree/wt-read-api) at least in version 1.0.0



---
Contacts:
[Google Group](https://groups.google.com/forum/#!forum/windingtree) |
[Twitter](https://twitter.com/windingtree) |
[Medium](http://blog.windingtree.com/) |
[YouTube](https://www.youtube.com/channel/UCFuemEOhCfenYMoNdjD0Aew) |
[Telegram](https://t.me/windingtree) |
[Reddit](https://reddit.com/r/windingtree) |
[BitcoinTalk](https://bitcointalk.org/index.php?topic=1946065)

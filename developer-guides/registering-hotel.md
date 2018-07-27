# Registering a hotel

- The address of Winding Tree index used in this example is for demo purposes only. There is only example data
and anyone can delete them, because the originating wallet with password is publicly available.

## Javascript library

- We don't provide an easy way in wt-js-libs how to store your off-chain data.
- Dependencies
```
  "@windingtree/off-chain-adapter-http": "2.0.0",
  "@windingtree/off-chain-adapter-swarm": "3.2.0",
  "@windingtree/wt-js-libs": "0.2.8"
```


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

// !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
// !!!!!!!! NEVER USE THIS WALLET !!!!!!!!
// !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
const WALLET_FILE = {"version":3,"id":"7fe84016-4686-4622-97c9-dc7b47f5f5c6","address":"d037ab9025d43f60a31b32a82e10936f07484246","crypto":{"ciphertext":"ef9dcce915eeb0c4f7aa2bb16b9ae6ce5a4444b4ed8be45d94e6b7fe7f4f9b47","cipherparams":{"iv":"31b12ef1d308ea1edacc4ab00de80d55"},"cipher":"aes-128-ctr","kdf":"scrypt","kdfparams":{"dklen":32,"salt":"d06ccd5d9c5d75e1a66a81d2076628f5716a3161ca204d92d04a42c057562541","n":8192,"r":8,"p":1},"mac":"2c30bc373c19c5b41385b85ffde14b9ea9f0f609c7812a10fdcb0a565034d9db"}};
const PASSWORD = 'windingtree';
// This is for demo purposes only, it's the same
// wallet used in https://github.com/windingtree/wt-demo-app
// Never use this wallet for any production data,
// everyone would have access to it.
// However, this Wallet has some Ether on Ropsten,
// so this example should work.

// 3. Get your off-chain data ready
// This is not provided by wt-js-libs out of the box
const offChainDataUri = 'https://jirkachadima.cz/wt/hotel-data-index.json';

// 4. Register your hotel to Winding Tree platform
(async () => {
  // Get an instance of WTIndex wrapper
  const index = await libs.getWTIndex('0x407f550023eb6ad8a4797844489e17c5ced17e06');

  // Create a Wallet abstraction and unlock it.
  const wallet = await libs.createWallet(WALLET_FILE);
  wallet.unlock(PASSWORD);

  try {
    // Register the hotel itself
    // a. Get ready transaction data
    const { hotel, transactionData, eventCallbacks } = await index.addHotel({
      manager: wallet.getAddress(),
      dataUri: offChainDataUri,
    });
    // b. Sign and send the transaction. You probably don't have to use our wallet abstraction.
    // This signs a transaction and waits for it to be mined. You can get finer control
    // of this by using your own eventCallbacks, not waiting for the promise to be resolved etc.
    const receipt = await wallet.signAndSendTransaction(transactionData, eventCallbacks);
    // After the transaction is mined, one of the eventCallbacks
    // sets the address of the freshly created hotel.
    const newHotelAddress = hotel.address;
    console.log('hotel address: ', newHotelAddress);
  } finally {
    // Don't forget to lock your wallet after you are done, you
    // don't want to leave your private keys lying around.
    wallet.lock();
  }
})();
```

## REST API

TBA

---
Contacts:
[Google Group](https://groups.google.com/forum/#!forum/windingtree) |
[Twitter](https://twitter.com/windingtree) |
[Medium](http://blog.windingtree.com/) |
[YouTube](https://www.youtube.com/channel/UCFuemEOhCfenYMoNdjD0Aew) |
[Telegram](https://t.me/windingtree) |
[Reddit](https://reddit.com/r/windingtree) |
[BitcoinTalk](https://bitcointalk.org/index.php?topic=1946065)

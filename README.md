# WavesPhpAPI

The wrapper and pre-processor for conventional node REST API and cryptographic primitives.
In spite of including only basic functional, it's easily extensible for all operations you need (such as leasing or batch transfers).

See [https://github.com/wavesplatform/Waves/wiki](https://github.com/wavesplatform/Waves/wiki) to learn more about REST API

## Requirements 
  * CURL utility (console access)
  * PHP 5.4+
  * Base58 encoder/decoder
  * Curve-25519 extension
  * Blake2b library or extension (optional)
  * Keccak256 library or extension (optional)

Hashing is applied only in manipulations with account (wallet) and node's API key.
All crypto-methods are combined in trait, so one may use custom implementation of any of these algorithms.

## Usage

### Account

Create new account

```$account = new \Wavesapi\Account();```

or import existing one

```$account = new \Wavesapi\Account($base58EncodedSeed);```

Get base58-encoded seed

```$account->getSeed();```

Get base58-encoded public key

```$account->getPublicKey();```

Get base58-encoded private key

```$account->getPrivateKey();```

Use the following solution instead of '/utils/hash' API method
```
// returns H6nsiifwYKYEx6YzYD7woP1XCn72RVvx6tC1zjjLXqsu
\Wavesapi\Account::chainedHash('ridethewaves!')
```

### Address
All path-related classes are subclasses of \Wavesapi\Request. You have to configure the superclass by passing host name and api_key in the constructor of the child object.
Port number should be specified in host param if necessary.
```
$address = new \Wavesapi\Address([
  'host'    => $host,
  'api_key' => $api_key
]);
// list of addresses in the node's wallet
$address->getList();
// create new address from for the current seed
$address->create();
// balance in WAVES
$address->getBalance();
```
### Transaction
```
$txn = new \Wavesapi\Transaction();
// transaction's data by its id 
$txn->getById($txn_id);
// latest transactions by the given account address
$txn->getByAccount($addr, $limit);
```
### Assets
```
$assets = new \Wavesapi\Assets();

// balances for all assets that the account with given address ever had (besides WAVES)
$assets->getBalance($addr);

// account's balance for the given asset
$assets->getBalanceForId($addr, $assetId)

// distribution of asset balance over accounts
$assets->getDistribution($assetId);
```

Publish signed asset transfer transaction to the blockchain:
```
// timestamp should be passed in milliseconds, e.g. time() * 1000
// private key is also expected to be base58-encoded string
$assets->setCredentials($senderPublicKey, $senderPrivateKey);
$assets->broadcastTransfer([
  'assetId'       =>  $assetId,
  'recipient'     =>  $recipient,
  'amount'        =>  $amount,
  'fee'           =>  $fee,
  'feeAssetId'    =>  $feeAssetId,
  'timestamp'     =>  $timestamp,
  'attachment'    =>  $attachment
]);
```

### Matcher

DEX access object is designed to be configured separately because hostname and/or port for matcher are usually not the same as for the rest-api module.

```
$dex = new \Wavesapi\Matcher([
  'host' => $host
]);

// matcher's public key
$dex->getPublicKey();
```

Price and amount are always specified as integers with merged decimal positions.
Price (price per unit) should be normalized on WAVES asset precision accordingly to the formula $price = $yourPrice * (10^8 / $amountAssetPrecision).

```
// price-amount orderbook (depth of market) for a given asset pair
$dex->getDOM($amountAsset, $priceAsset);

// order history for a given public key
$dex->setCredentials($senderPublicKey, $senderPrivateKey);

$dex->getHistoryByPublicKey();

// create limit order
// minimum fee is 0.001 WAV ($matcherFee = 100000)
$dex->createOrder([
  'matcherPublicKey'  =>  $matcherPublicKey,
  'amountAsset'       =>  $amountAsset,
  'priceAsset'        =>  $priceAsset,
  'orderType'         =>  'buy', // or 'sell'
  'price'             =>  $price,
  'amount'            =>  $amount,
  'timestamp'         =>  $timestamp,
  'expiration'        =>  $expiration,
  'matcherFee'        =>  $matcherFee 
]);

// cancel previously created order
$dex->cancelOrder([
  'amountAsset'  =>  $amountAsset,
  'priceAsset'   =>  $priceAsset,
  'orderId'      =>  $orderId,
  'timestamp'    =>  $timestamp
]);
```

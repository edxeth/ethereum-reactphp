# Ethereum-ReactPHP (updated with listener/indexer + bugfixes)

is a typed PHP-7.1+ interface to [Ethereum JSON-RPC API](https://github.com/ethereum/wiki/wiki/JSON-RPC).

Check out the latest [API documentation](http://ethereum-php.org/dev/).

### Add library in a [composer.json](https://getcomposer.org/doc/01-basic-usage.md#composer-json-project-setup) file

```yaml
{
  "minimum-stability":"dev",
  "autoload": {
    "psr-4": {
      "Ethereum\\": "src/"
    }
  },
  "repositories": [
    {
      "type": "git",
      "url": "https://github.com/digitaldonkey/ethereum-php.git"
    }
  ],
  "require": {
    "digitaldonkey/ethereum-php": "dev-master"
  }
}
```

### Usage


```sh
composer require digitaldonkey/ethereum-php
```

This is the important part of [composer.json](https://github.com/digitaldonkey/ethereum/blob/8.x-1.x/composer.json) in [Drupal Ethereum Module](https://drupal.org/project/ethereum).


```php
require __DIR__ . '/vendor/autoload.php';
use Ethereum\Ethereum;

try {
	// Connect to Ganache
    $eth = new Ethereum('http://127.0.0.1:7545');
    // Should return Int 63
    echo $eth->eth_protocolVersion()->val();
}
catch (\Exception $exception) {
    die ("Unable to connect.");
}

```

**Calling Contracts**

You can call (unpayed) functions in smart contracts easily. 

 The json file "$fileName" used is what you get when you compile a contract with [Truffle](truffleframework.com). 

```php
$ContractMeta = json_decode(file_get_contents($fileName));
$contract = new SmartContract(
  $ContractMeta->abi,
  $ContractMeta->networks->{NETWORK_ID}->address,
  new Ethereum(SERVER_URL)
);
$someBytes = new EthBytes('34537ce3a455db6b')
$x = $contract->myContractMethod();
echo $x->val()
```

You can also run tests at smart contracts, check out EthTestClient.

### Event listening and handling

You can use Ethereum-PHP to watch changed on your smart contracts or index a Blockchain block by block. gs

See [UsingFilters](https://github.com/digitaldonkey/ethereum-php/blob/master/UsingFilters.md) and [ethereum-php-eventlistener](https://github.com/digitaldonkey/ethereum-php-eventlistener).


### Limitations

Currently not all datatypes are supported.

This library is read-only for now. This means you can retrieve information stored in Ethereum Blockchain.

To *write* to the blockchain you need a to sign transactions with a private key which is not supported yet.


![architecture diagram](https://raw.githubusercontent.com/digitaldonkey/ethereum-php/dev/doxygen-assets/ArchitectureDiagrammCS6.png "Drupal Ethereum architecture")

### Documentation

The API documentation is available at [ethereum-php.org](http://ethereum-php.org/).

For reference see the [Ethereum RPC documentation](https://github.com/ethereum/wiki/wiki/JSON-RPC) and for data encoding [RLP dcumentation](https://github.com/ethereum/wiki/wiki/RLP) in [Ethereum Wiki](https://github.com/ethereum/wiki).

There is also a more readable [Ethereum Frontier Guide](http://ethereum.gitbooks.io/frontier-guide/content/rpc.html) version.

# Listener and Indexer

**TL;DR**

When developing dapps you might need some Backend process to react data of the latest block or on on-chain Events (Solidity Events). You might fill up some database by indexing all blocks or set up a daemon process to do something every time a new Block is created. 

--------------------------------

As much as we want real decentralization, the reality is actually different. 
Mots Dapp's require a Backend process. Sometimes for additional data which is too expensive to store (semi decentralized apps) or at least for monitoring what is happening at on chain.


You can do this in PHP very easily. E.g Indexing a chain from block 0 to the latest at script start time:

```php
$web3 = new Ethereum('http://192.168.99.100:8545');
// Block 0 -> last block.
new BlockProcessor($web3, function (Block $block) {

    // This will be run on every Block.
    print "\n\n#### BLOCK NUMBER " . $block->number->val() . " ####\n";

    // Add to database... 
    print_r($block->toArray());
  }
);

``` 

## Integration with Truffle and Contract Events

If you are using [Truffle](http://truffleframework.com/) to develop you Dapp you can easily set up a monitoring system for your smart contracts:

```php 
// Extend a \Ethereum\SmartContract with EventHandlers
class CallableEvents extends SmartContract {
  public function onCalledTrigger1 (EthEvent $event) {
    echo '### ' . substr(__FUNCTION__, 2) . "(\Ethereum\EmittedEvent)\n";
    var_dump($event);
  }
  public function onCalledTrigger2 (EthEvent $event) {
    echo '### ' . substr(__FUNCTION__, 2) . "(\Ethereum\EmittedEvent)\n";
    var_dump($event);
  }
}

$web3 = new Ethereum('http://192.168.99.100:8545');
$networkId = '5777';

// Contract Classes must have same name as the solidity classes for this to work.
$contracts = SmartContract::createFromTruffleBuildDirectory(
  'YOUR/truffle/build/contracts',
   $web3,
   $networkId
);

// process any Transaction from current Block to the future.
new ContractEventProcessor(
  $web3,
  $contracts,
  'latest',
  'latest'
);

```

## Background 

The Loop script is based on [reactphp](https://github.com/reactphp/react) which is actually older that Javascript React. 

The Ethereum part is based on [Ethereum-php](https://github.com/digitaldonkey/ethereum-php) library.

You might use [ganache-cli-docker-compose](https://github.com/digitaldonkey/ganache-cli-docker-compose) for testing.

You may use the indexer with [Infura](https://infura.io) Ethereum as a Service, as it doesn't rely on filters, which Infura does not support.

I used a very simple [DAPP](https://github.com/digitaldonkey/react-box-event-handling) to interact with the CallableEvents smart contract used as a example here. 

monkeytips pool
====================
Formerly known as cryptonote-forknote-pool, forked from TurtleCoin then forked from Worktips coin.

High performance Node.js (with native C addons) mining pool for Cryptonote based coin.

Comes with lightweight example front-end script which uses the pool's AJAX API.


#### Table of Contents
* [Features](#basic-features)
* [Usage](#usage)
  * [Requirements](#requirements)
  * [Pool setup](#1-pool-setup)
  * [Configuration](#2-configuration)
  * [Troubleshooting](#3-troubleshooting)
  * [Host the front-end](#4-host-the-front-end)
  * [Customizing your website](#5-customize-your-website)
  * [Using walletd instead of simplewallet](#6-using-walletd-instead-of-simplewallet-for-payments)
* [Credits](#credits)
* [License](#license)


#### Basic features

* TCP (stratum-like) protocol for server-push based jobs
  * Compared to old HTTP protocol, this has a higher hash rate, lower network/CPU server load, lower orphan
    block percent, and less error prone
* IP banning to prevent low-diff share attacks
* Socket flooding detection
* Payment processing
  * Splintered transactions to deal with max transaction size
  * Minimum payment threshold before balance will be paid out
  * Minimum denomination for truncating payment amount precision to reduce size/complexity of block transactions
* Detailed logging
* Ability to configure multiple ports - each with their own difficulty
* Variable difficulty / share limiter
* Share trust algorithm to reduce share validation hashing CPU load
* Clustering for vertical scaling
* Modular components for horizontal scaling (pool server, database, stats/API, payment processing, front-end)
* Live stats API (using AJAX long polling with CORS)
  * Currency network/block difficulty
  * Current block height
  * Network hashrate
  * Pool hashrate
  * Each miners' individual stats (hashrate, shares submitted, pending balance, total paid, etc)
  * Blocks found (pending, confirmed, and orphaned)
* An easily extendable, responsive, light-weight front-end using API to display data

#### Extra features

* Admin panel
  * Aggregated pool statistics
  * Coin daemon & wallet RPC services stability monitoring
  * Log files data access
  * Users list with detailed statistics
* Historic charts of pool's hashrate and miners count, coin difficulty, rates and coin profitability
* Historic charts of users's hashrate and payments
* Miner login(wallet address) validation
* Five configurable CSS themes
* Universal blocks and transactions explorer based on [chainradar.com](http://chainradar.com)
* FantomCoin & MonetaVerde support
* Set fixed difficulty on miner client by passing "address" param with ".[difficulty]" postfix
* Prevent "transaction is too big" error with "payments.maxTransactionAmount" option


### Community / Support

* [CryptoNote Technology](https://cryptonote.org)
* [CryptoNote Forum](https://forum.cryptonote.org/)
* [CryptoNote Universal Pool Forum](https://bitcointalk.org/index.php?topic=705509)
* [Forknote](https://forknote.net)


Usage
===

#### Requirements

* [Node.js](http://nodejs.org/) v0.10+ ([follow these installation instructions](https://github.com/nodejs/node#download))
* [Redis](http://redis.io/) key-value store v2.6+ ([follow these instructions](http://redis.io/topics/quickstart))
* libssl required for the node-multi-hashing module
* For Ubuntu: `sudo apt-get install libssl-dev`


#### 1) POOL SETUP

Rebuild the coin from the following GitHub repository: https://github.com/monkeytips/monkeytips using the instructions in the repository Readme.md file.

-  Start the daemon in the RPC mode

```
./monkeytipsd --rpc-bind-port=13002
```

-  Generate the pool wallet using walletd

```
./walletd --container-file=YOURWALLETNAME --container-password=YOURWALLETPASSWORD --daemon-port=13002 --bind-port=8420 --rpc-legacy-security --generate-container
```

-  Start the wallet using walletd

```
./walletd --container-file=YOURWALLETNAME --container-password=YOURWALLETPASSWORD --daemon-port=13002 --bind-port=8420 --rpc-legacy-security
```

Note: if you want to use the RPC password please set it in the pool wallet config section in the file config.js


-  Setup pool environment

```
sudo apt-get install nodejs nodejs-legacy npm tk8.5 tcl8.5 libssl-dev
```

```
npm config set strict-ssl false
```

-  Install Redis database

```
wget http://download.redis.io/redis-stable.tar.gz
```

```
tar xvzf redis-stable.tar.gz
```

```
cd redis-stable
```

```
make
```

```
make test
```

```
sudo apt-get install redis-server
```

- Start Redis server

```
redis-server
```

-  Clone the pool repository

```
git clone https://github.com/monkeytips/monkeytips-pool.git pool
```

-  Run the pool NPM update

```
cd pool
```

```
npm update
```

-  Start the pool (ONLY AFTER YOU HAVE CONFIGURED IT - see below for configuration)

```
sudo node init.js
```


#### 2) Configuration


Explanation for each field:
```javascript
/* Used for storage in redis so multiple coins can share the same redis instance. */
"coin": "monkeytips",

/* Used for front-end display */
"symbol": "WTIP",

/* Minimum units in a single coin, see COIN constant in DAEMON_CODE/src/cryptonote_config.h */
"coinUnits": 100,

/* Coin network time to mine one block, see DIFFICULTY_TARGET constant in DAEMON_CODE/src/cryptonote_config.h */
"coinDifficultyTarget": 40,

"logging": {

    "files": {

        /* Specifies the level of log output verbosity. This level and anything
           more severe will be logged. Options are: info, warn, or error. */
        "level": "info",

        /* Directory where to write log files. */
        "directory": "logs",

        /* How often (in seconds) to append/flush data to the log files. */
        "flushInterval": 5
    },

    "console": {
        "level": "info",
        /* Gives console output useful colors. If you direct that output to a log file
           then disable this feature to avoid nasty characters in the file. */
        "colors": true
    }
},

/* Modular Pool Server */
"poolServer": {
    "enabled": true,

    /* Set to "auto" by default which will spawn one process/fork/worker for each CPU
       core in your system. Each of these workers will run a separate instance of your
       pool(s), and the kernel will load balance miners using these forks. Optionally,
       the 'forks' field can be a number for how many forks will be spawned. */
    "clusterForks": "auto",

    /* Address where block rewards go, and miner payments come from. */
    "poolAddress": "YOUR_POOL_WALLET_ADDRESS_HERE"

    /* Poll RPC daemons for new blocks every this many milliseconds. */
    "blockRefreshInterval": 1000,

    /* How many seconds until we consider a miner disconnected. */
    "minerTimeout": 900,

    "ports": [
        {
            "port": 3333, //Port for mining apps to connect to
            "difficulty": 5000, //Initial difficulty miners are set to
            "desc": "Low end hardware" //Description of port
        },
        {
            "port": 5555,
            "difficulty": 25000,
            "desc": "Mid range hardware"
        },
        {
            "port": 7777,
            "difficulty": 100000,
            "desc": "High end hardware"
        }
    ],

    /* Variable difficulty is a feature that will automatically adjust difficulty for
       individual miners based on their hashrate in order to lower networking and CPU
       overhead. */
    "varDiff": {
        "minDiff": 5000, //Minimum difficulty
        "maxDiff": 2000000,
        "targetTime": 100, //Try to get 1 share per this many seconds
        "retargetTime": 30, //Check to see if we should retarget every this many seconds
        "variancePercent": 30, //Allow time to very this % from target without retargeting
        "maxJump": 100 //Limit diff percent increase/decrease in a single retargeting
    },

    /* Set difficulty on miner client side by passing <address> param with .<difficulty> postfix
       minerd -u D3z2DDWygoZU4NniCNa4oMjjKi45dC2KHUWUyD1RZ1pfgnRgcHdfLVQgh5gmRv4jwEjCX5LoLERAf5PbjLS43Rkd8vFUM1m.5000 */
    "fixedDiff": {
        "enabled": true,
        "separator": ".", // character separator between <address> and <difficulty>
    },

    /* Feature to trust share difficulties from miners which can
       significantly reduce CPU load. */
    "shareTrust": {
        "enabled": true,
        "min": 10, //Minimum percent probability for share hashing
        "stepDown": 3, //Increase trust probability % this much with each valid share
        "threshold": 10, //Amount of valid shares required before trusting begins
        "penalty": 30 //Upon breaking trust require this many valid share before trusting
    },

    /* If under low-diff share attack we can ban their IP to reduce system/network load. */
    "banning": {
        "enabled": true,
        "time": 600, //How many seconds to ban worker for
        "invalidPercent": 25, //What percent of invalid shares triggers ban
        "checkThreshold": 30 //Perform check when this many shares have been submitted
    },
    /* Slush Mining is a reward calculation technique which disincentivizes pool hopping and rewards users to mine with the pool steadily: Values of each share decrease in time – younger shares are valued higher than older shares.
    More about it here: https://mining.bitcoin.cz/help/#!/manual/rewards */
    /* There is some bugs with enabled slushMining. Use with '"enabled": false' only. */

    "slushMining": {
        "enabled": false, // 'true' enables slush mining. Recommended for pools catering to professional miners
        "weight": 120, //defines how fast value assigned to a share declines in time
        "lastBlockCheckRate": 1 //How often the pool checks for the timestamp of the last block. Lower numbers increase load for the Redis db, but make the share value more precise.
    }
},

/* Module that sends payments to miners according to their submitted shares. */
"payments": {
    "enabled": true,
    "interval": 1800, //how often to run in seconds
    "maxAddresses": 50, //split up payments if sending to more than this many addresses
    "mixin": 1, //number of transactions yours is indistinguishable from
    "transferFee": 50000000, //fee to pay for each transaction
    "minPayment": 100000000000, //miner balance required before sending payment
    "maxTransactionAmount": 2000000000000000, //split transactions by this amount(to prevent "too big transaction" error)
    "denomination": 100000000 //truncate to this precision and store remainder
},

/* Module that monitors the submitted block maturities and manages rounds. Confirmed
   blocks mark the end of a round where workers' balances are increased in proportion
   to their shares. */
"blockUnlocker": {
    "enabled": true,
    "interval": 30, //how often to check block statuses in seconds

    /* Block depth required for a block to unlocked/mature. Found in daemon source as
       the variable CRYPTONOTE_MINED_MONEY_UNLOCK_WINDOW */
    "depth": 10,
    "poolFee": 0.0, //1.8% pool fee (2% total fee total including donations)
    "devDonation": 0.0, //0.1% donation to send to pool dev - only works with Monero
    "coreDevDonation": 0.0 //0.1% donation to send to core devs - works with Bytecoin, Monero, Dashcoin, QuarazCoin, Fantoncoin, AEON and OneEvilCoin
},

/* AJAX API used for front-end website. */
"api": {
    "enabled": true,
    "hashrateWindow": 600, //how many second worth of shares used to estimate hash rate
    "updateInterval": 3, //gather stats and broadcast every this many seconds
    "port": 8117,
    "blocks": 30, //amount of blocks to send at a time
    "payments": 30, //amount of payments to send at a time
    "password": "test" //password required for admin stats
},

/* Coin daemon connection details. */
"daemon": {
    "host": "127.0.0.1",
    "port": 13002
},

/* Wallet daemon connection details. */
"wallet": {
    "host": "127.0.0.1",
    "port": 8420,
    "password": "<replace with rpc password>"
},

/* Redis connection into. */
"redis": {
    "host": "127.0.0.1",
    "port": 6379
}

/* Monitoring RPC services. Statistics will be displayed in Admin panel */
"monitoring": {
    "daemon": {
        "checkInterval": 60, //interval of sending rpcMethod request
        "rpcMethod": "getblockcount" //RPC method name
    },
    "wallet": {
        "checkInterval": 60,
        "rpcMethod": "get_address_count"
    }

/* Collect pool statistics to display in frontend charts  */
"charts": {
    "pool": {
        "hashrate": {
            "enabled": true, //enable data collection and chart displaying in frontend
            "updateInterval": 60, //how often to get current value
            "stepInterval": 1800, //chart step interval calculated as average of all updated values
            "maximumPeriod": 86400 //chart maximum periods (chart points number = maximumPeriod / stepInterval = 48)
        },
        "workers": {
            "enabled": true,
            "updateInterval": 60,
            "stepInterval": 1800, //chart step interval calculated as maximum of all updated values
            "maximumPeriod": 86400
        },
        "difficulty": {
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        },
        "price": { //USD price of one currency coin received from cryptonator.com/api
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        },
        "profit": { //Reward * Rate / Difficulty
            "enabled": true,
            "updateInterval": 1800,
            "stepInterval": 10800,
            "maximumPeriod": 604800
        }
    },
    "user": { //chart data displayed in user stats block
        "hashrate": {
            "enabled": true,
            "updateInterval": 180,
            "stepInterval": 1800,
            "maximumPeriod": 86400
        },
        "payments": { //payment chart uses all user payments data stored in DB
            "enabled": true
        }
    }
```

#### 3) Troubleshooting


- NPM issues resolving

```
sudo apt-get remove npm
```

```
sudo apt-get remove nodejs-legacy
```

```
sudo apt-get remove nodejs
```

```
sudo rm /usr/bin/node
```

```
sudo apt-get install nodejs
```

```
sudo apt-get install nodejs-legacy
```

```
sudo apt-get install npm
```

- NODEJS alternative installations

```
wget https://nodejs.org/dist/v0.10.25/node-v0.10.25.tar.gz && tar xvzf node-v0.10.25.tar.gz
```

```
cd node-v0.10.25
```

```
make
```

```
sudo make install
```

OR:

```
curl -sL https://deb.nodesource.com/setup_0.10 | sudo -E bash -
```

```
sudo apt-get install -y nodejs
```

THEN RUN:

```
cd pool
```

```
npm update
```

IF IT STILL FAILS:

```
cd pool
```

```
sudo rm -rf node_modules
```

```
sudo rm -rf ~/.node-gyp
```

```
npm update
```


#### 4) Host the front-end

Simply host the contents of the `website_example` directory on file server capable of serving simple static files.


Edit the variables in the `website_example/config.js` file to use your pool's specific configuration.
Variable explanations:

```javascript

/* Must point to the API setup in your config.json file. */
var api = "http://poolhost:8117";

/* Pool server host to instruct your miners to point to.  */
var poolHost = "poolhost.com";

/* IRC Server and room used for embedded KiwiIRC chat. */
var irc = "irc.freenode.net/#forknote";

/* Contact email address. */
var email = "support@poolhost.com";

/* Market stat display params from https://www.cryptonator.com/widget */
var cryptonatorWidget = ["DSH-BTC", "DSH-USD", "DSH-EUR"];

/* Download link to cryptonote-easy-miner for Windows users. */
var easyminerDownload = "https://github.com/zone117x/cryptonote-easy-miner/releases/";

/* Used for front-end block links. */
var blockchainExplorer = "http://chainradar.com/{symbol}/block/{id}";

/* Used by front-end transaction links. */
var transactionExplorer = "http://chainradar.com/{symbol}/transaction/{id}";

/* Any custom CSS theme for pool frontend */
var themeCss = "themes/default-theme.css";

```

#### 5) Customize your website

The following files are included so that you can customize your pool website without having to make significant changes
to `index.html` or other front-end files thus reducing the difficulty of merging updates with your own changes:
* `custom.css` for creating your own pool style
* `custom.js` for changing the functionality of your pool website


Then simply serve the files via nginx, Apache, Google Drive, or anything that can host static content.


#### 6) Using walletd instead of simplewallet for payments

This pool uses walletd for RPC queries with added RPC security. More information on how to upgrade from simplewallet to walletd please see the following post: [Use walletd instead of simplewallet for payments](https://github.com/turtlecoin/turtle-pool/pull/5)


Credits
===

* [LucasJones](//github.com/LucasJones) - Co-dev on this project; did tons of debugging for binary structures and fixing them. Pool couldn't have been made without him.
* [surfer43](//github.com/iamasupernova) - Did lots of testing during development to help figure out bugs and get them fixed
* [wallet42](http://moneropool.com) - Funded development of payment denominating and min threshold feature
* [Wolf0](https://bitcointalk.org/index.php?action=profile;u=80740) - Helped try to deobfuscate some of the daemon code for getting a bug fixed
* [Tacotime](https://bitcointalk.org/index.php?action=profile;u=19270) - helping with figuring out certain problems and lead the bounty for this project's creation
* [fancoder](https://github.com/fancoder/) - See his repo for the changes
* [TurtleCoin developers](https://github.com/turtlecoin/turtlecoin)

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html

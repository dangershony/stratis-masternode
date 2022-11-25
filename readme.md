## Running a Stratis sidechain (Cirrus) masternode in docker-compose

This guide will show how to run Cirrus masternodes in docker compose

**Disclaimer**  
**Use this guide at your own risk**  

**Note**  
This guide assumes the user has some prior knowledge of the blockchain api calls and how utxos and wallets work  
If you havn't already, please get familiar with running a masternode using the [offical stratis guide](https://academy.stratisplatform.com/Operation%20Guides/InterFlux%20Masternodes/Creating%20Masternode/creating-a-masternode.html)  

This guide was made on a windows OS with docker desktop wsl for linux, however it should be exactly the same if deployed on a linux OS.  
(if there are any differences please open an issue or make a PR)

### Step 1 - Build
We will build the images from source    

#### Building the Fullnode image

First find out which release of the code you want to run go to releases,  
https://github.com/stratisproject/StratisFullNode/releases  

To create the image run this command, set the APPVERSION or keep blank to build master

`docker build -f "Dockerfile-cirrusd" --build-arg APPVERSION=1.3.2.4 -t masternode/cirrus-miner:1.3.2.4 .`

#### Building the Dashboard image

Dashboard is always master

`docker build -f "Dockerfile-dashboard" -t masternode/cirrus-dashboard:latest .`

#### Upgradeing an existing setup  

If you already have a setup and just upgrading the docker images, run the code above to generate new images (with the new versions).  
Change the images in the `docker-compose.yml` file.  
Then just call `docker-compose up -d` to update the new images.  

### Step 2 - Deploy and sync the nodes

The docker script is built for running two sidechain masternodes and one mainchain staking node.  
If you want to run only one master node edit the file `docker-compose.yml` and just comment out (or delete) the second masternode and it's dashboard.  
However if you want to run more then 2 masternodes (well done) just add a third masternode and dashboard following the same pattern as the other two (don't forget to use different ports for each masternode).  

After amending the `docker-compose.yml` you are ready to deploy.  

#### Running the docker compose containers

First deploy the chains to start the sync process (this can take about 12 hours)

`docker-compose up -d`

To view the progress of the sync just open a log view 

`docker-compose logs -f --tail=100`

### Step 3 - Setup and registration

To simplify the processes wait for the nodes to complete the sync before creating the wallets.  

**IMPORTANT**   
This script will open the api ports to the local computer for example `-apiuri=http://0.0.0.0:17103`, in this example the port is `17103` this is so we can interact with the node, however if those ports are open on the router or firewall the node might be accesible to the ourside world.  
If using a home pc make sure those ports are not open.  
If using a cloud make sure those ports are blocked.  
If in doubt you can always test by going to this url `http://[my-public-ip]:17103` when the node is running.  

#### Creating mainchain wallet

Open the swagger api on main chain `http://localhost:17103/swagger/index.html`
If you already have a wallet recover it, alternatively create one and top it up with 100k for each masternode.  
This can be doen in the wallet apis `/api/Wallet/create` or `/api/Wallet/recover`  

The network requires that each masternode has 100k STRAX in a unique address as collateral, this means we need to make sure our designated addresses have the correct amounts.  
When sending regular transactions the wallet will normally allocate a new change address, to bypass that we must build the transaction correctly.  
First find two designated addresses by using the api call `/api/Wallet/addresses`

To top up those addresses from funds already in the wallet you can use this command  
`/api/Wallet/build-transaction`  
And pass the following json  
```
{
  "password": "[wallet-password]",
  "segwitChangeAddress": false,
  "walletName": "[wallet-name]",
  "accountName": "account 0",
  "feeType": "low",
  "recipients": [
    {
      "destinationAddress": "[masternode1-address]",
      "subtractFeeFromAmount": true,
      "amount": "100000"
    },
    {
      "destinationAddress": "[masternode2-address]",
      "subtractFeeFromAmount": true,
      "amount": "100000"
    }
  ]
}
```
**I recommend to use small amounts on a test transaction first, unless the user is familiar with this process**  

Once a transaction is created use the api call `/api/Wallet/send-transaction` to send it to the network and wait for it to confirm  
Use the api call `/api/Wallet/addresses` to see the coins in the correct address  

### Staking on mainchain  
To start staking call the api command `/api/Staking/startstaking`  
You will need to do this after every restart.  

To have the node start staking on startup locate the config file (on windows it's under `\\wsl$\docker-desktop-data\data\docker\volumes\masternode-strax-data\_data\strax\StraxMain\strax.conf` and add the following entries  
```
walletname=[wallet-name]
walletpassword=[wallet-password]
stake=1
```

**Important** - Caution, this will leave your password in plain text in a file on the machine.

#### Creating sidechain wallet

On the sidechain create or recover a wallet, navigate to `http://localhost:37223/swagger/index.html`  
(If its the second masternode then change the port accordingly)  

Top up the sidechain node with 501 CRS, this will be needed for the masternode registration.  

#### Creating federation keys

Follow this link to download the [`Stratis KeyGen Utility`]( https://academy.stratisplatform.com/Operation%20Guides/InterFlux%20Masternodes/Creating%20Masternode/creating-a-masternode.html#stratis-keygen-utility)

Follow the steps to generate a federation key  

Once you have a key copy it to the sidechain node root directory, to find that directory call the command `docker inspect cirrus1-mn-chain`.  
On windows this should normally be under `\\wsl$\docker-desktop-data\data\docker\volumes\masternode-cirrus1-data`  
Navigate to the `\masternode-cirrus1-data\_data\cirrus\CirrusMain` folder and copy the `federationKey.dat` to that folder.  

Follow the same steps for the second masternode (and any additional masternodes you want to add)  

You are all set.

#### Register as a masternode

To start producing blocks on the masternode we need to register as a masternode, this costs 500 CRS.  
And we must make sure we have 100k as collateral on the mainchain.

On the masternode swagger api (assuming masternode1 go to `http://localhost:37223/swagger/index.html`)  
Locate the api call `/api/Collateral/joinfederation` and populate the following entries
```
{
  "collateralAddress": "[the address with the 100k collateral on STRAX mainchain]",
  "collateralWalletName": "[The wallet name of the collateral address]",
  "collateralWalletPassword": "[The wallet password of the collateral address]",
  "walletName": "[wallet name on the sidechain, we will take the 500 CRS from this wallet]",
  "walletPassword": "[wallet password on the sidechain]",
  "walletAccount": "account 0"
}
```

Calling this function will register your node as a masternode.  

To follow the voting progress call the api method `/api/Voting/polls/pending` `votetype=1` and specify your federation public key (you got it when generating the `federationKey.dat`)

It will take 240 blocks before your Masternode will be accepted into the federation.  

### Step 4 - Dashboard
Right now I could not get the dashboard to work without additional twicking

To configure the dashboard go to the dashboard data folder (normally located at `\\wsl$\docker-desktop-data\data\docker\volumes\masternode-cirrus1-dashboard\_data`).  
And set the following in the files `appsettings.json` and `appsettings.Development.json`  
```
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "DefaultEndpoints": {
    "StratisNode": "http://strax-mn-chain:17103",
    "SidechainNode": "http://cirrus1-mn-chain:37223",
    "SidechainNodeType": "10K",
    "EnvType": "MainNet",
    "IntervalTime": "30"
  },
  "AllowedHosts": "*",
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://0.0.0.0:37000"
      }
    }
  }
}
```

Adapt the ports and host name for each masternode (for example the masternode2 host is configured as `http://cirrus2-mn-chain:37224` and Url is `http://0.0.0.0:37001`) look in the docker-compse file to be sure.  

Navigate to `http://localhost:37000/` to view the dashboard

### FAQ

- *If I restart my pc do I need to do anythng?*  
No docker is pretty resilient it will bring all the containers back online and continue staking (for mainchain you may have to start staking unless you setup auto staking in config)   
- *My masternode will not fully start with the error 'gateway not ready'*  
Check the strax mainchain to see it's fully synced, remember to also check the address indexer.  
- *If I want to add a new masternode later can I do that?*  
Sure just follow the same process to add masternode3 (or 4 or 5 etc...) give unique ports and names and run the command `docker-compose up -d` to add the new containers, **Important** make sure you don't remove collateral from the other addresses when topping up the new collateral address.  
- *Can I monitor my staking coins on mainchain and sidechain from my mobile phone or another computer?*  
Yes you can, either do that using our coinvault/blockcore explorers https://explorer.coinvault.io/ - https://explorer.blockcore.net/ or import your wallets to the coinvault/blockcore non-custodial wallets under https://wallet.coinvault.io/ - https://wallet.blockcore.net/ and view your rewards there (for staking wallets we recommend the quick mode)

Good luck...

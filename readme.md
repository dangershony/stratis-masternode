## Running Stratis sidechain (Cirrus) masternode in docker-compose

This guide will show how to run Cirrus masternode in docker compose

### Step 1 - Build
We will build the images from the repo itself  

#### Building the Fullnode image

First find out which release of the code you want to run go to releases,  
https://github.com/stratisproject/StratisFullNode/releases  

To create the image run this command, set the APPVERSION or keep blank to build master

`docker build -f "Dockerfile-cirrusd" --build-arg APPVERSION=1.3.2.4 -t masternode/cirrus-miner:1.3.2.4 .`


#### Building the Dashboard image

Dashboard is always master

`docker build -f "Dockerfile-dashboard" -t masternode/cirrus-dashboard:latest .`

### Step 2 - Deploy and sync the nodes

#### Running the docker compose containers

TBD

### Step 3 - Setup and registration

TBD

#### Creating federation keys

TBD

#### Creating wallets

TBD

#### Register as a masternode

TBD

### Step 4 - Dashboard

### Setting up the dashboard

TBD

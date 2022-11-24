## Running Stratis sidechain (Cirrus) masternode in docker

This repo will show how to run Cirrus masternode in docker compose

### Building the Fullnode image
We will build the images from the repo itself

First find out which release of the code you want to run go to releases,  
https://github.com/stratisproject/StratisFullNode/releases  

To create the image run this command, set the APPVERSION or keep blank to build master

`docker build -f "Dockerfile-cirrusd" --build-arg APPVERSION=1.3.2.4 -t masternode/cirrus-miner:1.3.2.4 .`


#### Building the Dashboard image

Dashboard is always master

`docker build -f "Dockerfile-dashboard" -t masternode/cirrus-dashboard:latest .`

### Running the docker compose containers

TBD

### Creating federation keys and wallets

TBD

### Setting up the dashboard

TBD

How to run this

make sure the version you want to deploy (edit the Dockerfiles)

### create miner image use the version you like (or remove the APPVERSION to clone master)

docker build -f "Dockerfile-cirrusd" --build-arg APPVERSION=1.3.2.4 -t masternode/cirrus-miner:1.3.2.4 .

### create teh dashboard image

docker build -f "Dockerfile-dashboard" -t masternode/cirrus-dashboard:latest .

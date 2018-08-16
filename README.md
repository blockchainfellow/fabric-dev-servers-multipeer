# Multi Machine Fabric Setup: Org on each Machine

#### Here are the full terminal instructions starting from a basic Ubuntu 18.04 install 

```
sudo apt update && sudo apt upgrade
sudo apt install nano git make gcc g++ libltdl-dev curl python pkg-config
```

### Install Docker/ Docker Compose
```
curl -fsSL test.docker.com | sh
sudo usermod -aG docker $USER
exec sudo su -l $USER
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Install nodejs / Composer
```
wget https://nodejs.org/dist/v8.11.2/node-v8.11.2-linux-x64.tar.xz
tar -xf node-v8.11.2-linux-x64.tar.xz 
mv node-v8.11.2-linux-x64/ node/
echo 'export PATH=~/node/bin:$PATH' >> ~/.profile
source .profile
npm install -g npm 
npm install -g grpc
npm install -g composer-cli@0.16.6 composer-playground@0.16.6 generator-hyperledger-composer@0.16.6
```

### Install Go / Hyperledger Binaries
```
wget https://dl.google.com/go/go1.10.2.linux-amd64.tar.gz
sudo tar -C /usr/local -xvf go1.10.2.linux-amd64.tar.gz
echo 'export GOPATH=/opt/go' >> ~/.profile
echo 'export GOBIN=/opt/go/bin' >> ~/.profile
echo 'export PATH=/usr/local/go/bin:$PATH' >> ~/.profile
source ~/.profile
sudo mkdir -p /opt/go/bin
sudo mkdir /opt/go/src
cd $GOPATH
sudo chown -R $USER /opt/go
cd src
mkdir -p github.com/hyperledger
cd github.com/hyperledger
git clone https://gerrit.hyperledger.org/r/fabric
cd fabric
make release
```

### Download the repo for Multi Org
```
cd ~
curl -sSL https://goo.gl/byy2Qj | bash -s 1.0.4
mkdir fabric-binaries
mv bin fabric-binaries/bin
echo 'export PATH=~/fabric-binaries/bin:$PATH' >> ~/.profile
source ~/.profile
git clone https://github.com/InflatibleYoshi/fabric-dev-servers-multipeer
cd fabric-dev-servers-multipeer
cd composer
nano howtobuild.sh //Replace the IP addresses in HOST1 and HOST2 with your own IPs or FQDNs
./howtobuild.sh
cd ../..
```

At this point, if you have done these instructions for one machine, either duplicate your VM at this time or prepare another environment with the same steps as described so far until you get to git cloning this repo. Instead of cloning the repo, scp -r the fabric-dev-servers-multipeer folder from the first machine to the second.

On the first machine run
```
./startFabric.sh
```

Copying on second machine
```
a. On Local: scp -i "blockchain.pem" blockchain.pem ubuntu@IP1:/home/ubuntu
b. On IP1: scp -i "blockchain.pem" -r fabric-dev-servers-multipeer/ ubuntu@IP2:/home/ubuntu
c. Enable the SG group on AWS to allow all.
```

On the second machine run
```
.startFabric-Peer2.sh
```

After this continue on the first machine.

### Create the Composer profile on the First Machine and start Composer Playground and Blockchain Explorer
```
./createPeerAdminCard.sh
nohup composer-playground -p 8181 &
cd ..
git clone https://github.com/InflatibleYoshi/blockchain-explorer
sudo apt install postgresql postgresql-contrib
cd blockchain-explorer
git checkout release-3
```
Edit config.json with the correct tlscerts path. You do not need them functionally but they are there because there have been reported issues when not including the tls certs.
```
sudo -u postgres psql
\i app/db/explorerpg.sql
\q
npm install
cd app/test
npm install
npm run test
cd ../../client
npm install
npm test -- -u –coverage
npm run build
cd ..
./start.sh
```

At this point you should be able to navigate a browser to http:/{HOST1-DOMAIN/IP}:8080 and connect to either alice or bob's trade network instances.

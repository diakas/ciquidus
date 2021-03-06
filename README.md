Ciquidus Alpha - 1.7.2
================

fork for BUGOCOIN

INSTALL

apt install zram-config

sysctl vm.swappiness=10

cat /proc/sys/vm/swappiness


sysctl -p

swapon -s

free -m

(add-apt-repository ppa:bitcoin/bitcoin  --- 18.04

sudo apt-get update

sudo apt-get install libdb4.8-dev libdb4.8++-dev build-essential libtool autotools-dev autoconf pkg-config sudo libssl-dev libboost-all-dev git npm nodejs nodejs-legacy libminiupnpc-dev redis-server ) ????


apt install libssl-dev libdb++-dev libboost-all-dev libqrencode-dev miniupnpc libminiupnpc-dev

после этого 

wget https://github.com/diakas/bugocoin/releases/download/pre/bugod16.04.zip

или https://github.com/diakas/bugocoin/releases/download/pre/bugod18.04.zip

распаковываем

chmod u+x bugod

./bugod -daemon -txindex

правим конф ПИШЕМ НОДЫ

/bugod getinfo

проверяем счетчик блоков


https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-18-04


apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927

echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-3.2.list

apt-get update

apt-get install -y mongodb-org

apt-get install upstart-sysv -y

apt-get install libgmp-dev - без него не работало????

reboot

service mongod start

service mongod stop

service mongod restart


Installing Nodejs

apt-get update

apt-get install nodejs nodejs-legacy npm -y


Creating a MongoDB Database

mongo

> use explorerdb

> db.createUser( { user: "dbuser-bugo-conf", pwd: "dbpassw-bugo-conf", roles: [ "readWrite" ] } )

> exit

вставить такие же в settings.json потом


Installing the Explorer


apt install libkrb5-dev

git clone https://github.com/diakas/ciquidus.git

cd ciquidus && npm install --production

cp ./settings.json.template ./settings.json


редактировать settings.json с кучей нюансов см. 
https://youtu.be/laeV2slJgc8

некоторые моменты:
1)"genesis_tx": "4592ecfada9248d7847196fa0c2796a59185a01b29ddc38242b4b9a9c7f2a3c7",
  "genesis_block": "0000af00b9e88a91147276bdf4689d3e202eb5d6d1c21e3035967f08d65a8d74",
  
2)"api": {
    "blockindex": 95774,
    "blockhash": "527c372b964e7e1e8a3c706d404fc2863fd512d620fe3af238d383eef7ce2a5f",
    "txhash": "79ed5f307c8c8fbdb14fa7748175157dd0d638d822f5e993b873e123ecdf99eb",
    "address": "Bbv7jhbtZbCwZKizM6fPDYs8oUjGtMvm1m"
  },
  
это вытаскивается из любого кошелька заходим в кансоль getblockhash 0 это будет genesis block
потом пишем getblock и то, что нам выдало предыдущее и берём оттуда genesis_tx

Переходим в папку (если не там) cd ciquidus

npm start

проверяем по адресу сервера х.х.х.х:3001/

теперь реиндекс для этого сначала Install Forever to keep the js running - чтоб можно закрыть окно - работает как служба в фоне

npm install forever -g

Start the Explorer в фоновом режиме

forever start -c "npm start" ./

проверка работы: forever list (остановка forever stop -c "npm start" ./ )


node scripts/sync.js index reindex - занимает много времени в зависимости разменра блокчейна и скорости системы - 2-3 часа а у больших блокчейнов дней и недель!!!


forever start -c "node scripts/sync.js index update" ./


теперь нужно настроить автообновление кронтаб чтоб реиндексировал например раз в 5 минут 

Add Cron to File to update the explorer

crontab -e

*/5 * * * * cd /root/ciquidus && /usr/bin/node scripts/sync.js index update > /dev/null 2>&1

*/20 * * * * cd /root/ciquidus && /usr/bin/node scripts/peer.js > /dev/null 2>&1

@reboot /root/bugod -daemon -txindex > /dev/null 2>&1
@reboot rm -f /root/ciquidus/tmp/index.pid
@reboot cd /root/ciquidus && /usr/bin/node forever start -c "npm start"./ > /dev/null 2>&1



если сбросило и пишет "script is already running."

rm tmp/index.pid

node scripts/sync.js index update


если хочешь обновить настройки

forever list

cd ciquidus

forever stop -c "npm start" ./

правим nano settings.json

forever start -c "npm start" ./

The Chaincoin block explorer.

This project is a fork of [Iquidus Explorer](https://github.com/iquidus/explorer) so massive thanks go out to Luke Williams for his code! Thank you!!!

### See it in action

*  [explorer.chaincoin.org](https://explorer.chaincoin.org)


### Requires

*  node.js >= 0.10.28
*  mongodb 2.6.x
*  *coind

### Create database

Enter MongoDB cli:

    $ mongo

Create databse:

    > use explorerdb

Create user with read/write access:

    > db.createUser( { user: "ciquidus", pwd: "3xp!0reR", roles: [ "readWrite" ] } )

*note: If you're using mongo shell 2.4.x, use the following to create your user:

    > db.addUser( { user: "username", pwd: "password", roles: [ "readWrite"] })

### Get the source

    git clone https://github.com/suprnurd/ciquidus explorer

### Install node modules

    cd explorer && npm install --production

### Configure

    cp ./settings.json.template ./settings.json

*Make required changes in settings.json*

### Start Explorer

    npm start

*note: mongod must be running to start the explorer*

As of version 1.4.0 the explorer defaults to cluster mode, forking an instance of its process to each cpu core. This results in increased performance and stability. Load balancing gets automatically taken care of and any instances that for some reason die, will be restarted automatically. For testing/development (or if you just wish to) a single instance can be launched with

    node --stack-size=10000 bin/instance

To stop the cluster you can use

    npm stop

### Syncing databases with the blockchain

sync.js (located in scripts/) is used for updating the local databases. This script must be called from the explorers root directory.

    Usage: node scripts/sync.js [database] [mode]

    database: (required)
    index [mode] Main index: coin info/stats, transactions & addresses
    market       Market data: summaries, orderbooks, trade history & chartdata

    mode: (required for index database only)
    update       Updates index from last sync to current block
    check        checks index for (and adds) any missing transactions/addresses
    reindex      Clears index then resyncs from genesis to current block

    notes:
    * 'current block' is the latest created block when script is executed.
    * The market database only supports (& defaults to) reindex mode.
    * If check mode finds missing data(ignoring new data since last sync),
      index_timeout in settings.json is set too low.


*It is recommended to have this script launched via a cronjob at 1+ min intervals.*

**crontab**

*Example crontab; update index every minute and market data every 2 minutes*

    */1 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js index update > /dev/null 2>&1
    */2 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/sync.js market > /dev/null 2>&1
    */5 * * * * cd /path/to/explorer && /usr/bin/nodejs scripts/peers.js > /dev/null 2>&1

forcesync.sh and forcesynclatest.sh (located in scripts/) can be used to force the explorer to sync at the specified block heights

### Wallet

The wallet connected to Ciquidus must be running with atleast the following flags:

    -daemon -txindex

### Donate
    
    CHC: CLkWg5YSLod772uLzsFRxHgHiWVGAJSezm
    BTC: 1J8Chi5teDJrvBtSuQhioNCHfTNBCcCrPx

### Known Issues

**script is already running.**

If you receive this message when launching the sync script either a) a sync is currently in progress, or b) a previous sync was killed before it completed. If you are certian a sync is not in progress remove the index.pid from the tmp folder in the explorer root directory.

    rm tmp/index.pid

**exceeding stack size**

    RangeError: Maximum call stack size exceeded

Nodes default stack size may be too small to index addresses with many tx's. If you experience the above error while running sync.js the stack size needs to be increased.

To determine the default setting run

    node --v8-options | grep -B0 -A1 stack_size

To run sync.js with a larger stack size launch with

    node --stack-size=[SIZE] scripts/sync.js index update

Where [SIZE] is an integer higher than the default.

*note: SIZE will depend on which blockchain you are using, you may need to play around a bit to find an optimal setting*

### License

Copyright (c) 2017, The Chaincoin Community  
Copyright (c) 2015, Iquidus Technology  
Copyright (c) 2015, Luke Williams  
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

* Neither the name of Iquidus Technology nor the names of its
  contributors may be used to endorse or promote products derived from
  this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

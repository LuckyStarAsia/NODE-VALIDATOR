# Cosmovisor

![image](https://github.com/user-attachments/assets/c8313273-31af-498e-80c9-deae9aa16a76)

Recommended Hardware: 6 Cores, 16GB RAM, 400GB of storage (NVME), 100 Mb/s

### install dependencies, if needed

```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### install go, if needed

```
cd $HOME
VER="1.22.11"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

OR (for user account)

```
[ ! -f $HOME/.bash_profile ] && touch $HOME/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
[ ! -d $HOME/go/bin ] && mkdir -p $HOME/go/bin
```

Check version

```
go version
```

### set vars

Port: `14` (you can change to another port)

```
echo "export MONIKER="Your-Monkier"" >> $HOME/.bash_profile
echo "export STORY_CHAIN_ID="aeneid"" >> $HOME/.bash_profile
echo "export STORY_PORT="14"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### install Story-geth

```
cd $HOME && \
rm -rf $HOME/story-geth && \
git clone https://github.com/piplabs/story-geth.git && \
cd $HOME/story-geth && \
git checkout v1.1.0 && \
make geth && \
mv build/bin/geth  $HOME/go/bin/
```

#### install Story

```
cd $HOME && \
rm -rf $HOME/story && \
git clone https://github.com/piplabs/story && \
cd $HOME/story && \
git checkout v1.3.0 && \
go build -o story ./client  && \
mv ./story $HOME/go/bin/story
```

### Cosmovisor Setup

#### Install and init Cosmovisor:

```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
echo "export DAEMON_NAME="story"" >> $HOME/.bash_profile
echo "export DAEMON_HOME="$HOME/.story/story"" >> $HOME/.bash_profile
source $HOME/.bash_profile
cosmovisor init $(which story)
```

create directory:

```
cd $HOME
mkdir -p $HOME/.story/cosmovisor/genesis/bin
mkdir -p $HOME/.story/cosmovisor/upgrades
cp $HOME/go/bin/story $HOME/.story/cosmovisor/genesis/bin/
```

Check Story version

```
$HOME/go/bin/story version
```

### UPGRADE v1.3.0:

Create a directory and download the current version of story:

```
cd $HOME
mkdir -p $HOME/.story/story/cosmovisor/upgrades/v1.3.0/bin
cp $HOME/go/bin/story $HOME/.story/story/cosmovisor/upgrades/v1.3.0/bin
```

#### Check version:

```
$HOME/.story/story/cosmovisor/upgrades/v1.3.0/bin/story version
```

result

```
Version       v1.3.0-stable
Git Commit    3c01046
Git Timestamp 2025-06-17T18:27:57Z
```

#### Check upgrade info

```
cat $HOME/.story/story/cosmovisor/upgrades/v1.3.0/upgrade-info.json
```

result: `{`"name":"v1.3.0","time":"0001-01-01T00:00:00Z","height":6008000`}`

#### Create upgrade simlink:

```
echo '{"name":"v1.3.0","time":"0001-01-01T00:00:00Z","height":6008000}' > $HOME/.story/story/cosmovisor/upgrades/v1.3.0/upgrade-info.json
```

#### Check current simlink:

```
ls -l $HOME/.story/story/cosmovisor/current
```

result: `lrwxrwxrwx 1 story story 15 May 5 17:20 /home/story/.story/story/cosmovisor/current -> upgrades/v1.2.0`

#### Check upgrade info:

```
cat $HOME/.story/story/cosmovisor/upgrades/v1.3.0/upgrade-info.json
```

result: `{"name":"v1.3.0","time":"0001-01-01T00:00:00Z","height":6008000}`

#### Sets up an automatic upgrade in Cosmovisor:

```
cosmovisor add-upgrade v1.3.0 $HOME/.story/story/cosmovisor/upgrades/v1.3.0/bin/story --force --upgrade-height 6008000
```

result:

```
10:13AM INF Using /home/story/.story/story/cosmovisor/upgrades/v1.3.0/bin/story for v1.3.0 upgrade module=cosmovisor
10:13AM INF Upgrade binary located at /home/story/.story/story/cosmovisor/upgrades/v1.3.0/bin/story module=cosmovisor
10:13AM INF /home/story/.story/story/data/upgrade-info.json created, v1.3.0 upgrade binary will switch at height 6008000 module=cosmovisor
```

### Create service file:

```
sudo tee /etc/systemd/system/story.service > /dev/null << EOF
[Unit]
Description=story node service
After=network-online.target

[Service]
User=$USER
Environment="DAEMON_NAME=story"
Environment="DAEMON_HOME=$HOME/.story/story"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_DATA_BACKUP_DIR=$HOME/.story/story/data"
ExecStart=$(which cosmovisor) run run
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### init story app

```
story init --moniker $MONIKER --network odyssey
```

### set seeds and peers

```
SEEDS="944e8889ecd7c13623ef1081aae4555d6f525041@b1-b.odyssey-devnet.storyrpc.io:26656"
PEERS="3b1aaa03f996d619cb2f4230ebace45686ab3b8a@34.140.167.127:26656,36ca8b119bf5851cd1e37060af914cb07dec24f9@34.79.40.193:26656,2a28bd1a6ecb0a1d8ceade599b311d202447d635@193.122.141.78:26656,b540a4a88399bee252207ab9cf783c14fcefd4dc@65.108.30.59:26656,b21b772ca2e4067844f881e8a79a7447dc435217@65.108.141.109:29456,6c89fb9e0791ffa67468b9f9923891a2bfcad80f@141.94.143.203:56356,1ff566a5ac0bd3605e8af09e92cacc43927aed7f@161.35.70.64:26656,2e00c3e558f382e48fe7511f50c069fde44a6468@150.136.128.196:26656,20a1a828469c42047601529a50f527ecf9301251@35.211.53.224:26656,b965eed902107d29df3669b2ff9a93859db236a3@49.12.92.82:56356,817a54d7ed4f3b618d37ea80448c135b20fc34e1@34.143.143.252:26656,0eda723784a874798b173df8f17545f9984b86e6@35.211.230.141:26656,155bcba7d521ced31042bd99100841c6cf057f36@35.211.9.151:26656,7e311e22cff1a0d39c3758e342fa4c2ee1aea461@188.166.224.194:28656,59201fade719c1e4ded98b2304e555377d2b4cef@116.202.217.20:28656,9d34ab3819aa8baa75589f99138318acfa0045f5@95.217.119.251:30900,45938d3dfe2877e1eb45cbce10f2d02c676f50a0@198.244.176.117:33656,580be4f3e5f505ed0ea15510997aeeb74e35408e@35.211.167.181:26656,944e8889ecd7c13623ef1081aae4555d6f525041@35.211.57.203:26656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.story/story/config/config.toml
```

### download genesis and addrbook

```
wget -O $HOME/.story/story/config/genesis.json https://story-testnet-services.luckystar.asia/story/genesis.json
wget -O $HOME/.story/story/config/addrbook.json  https://story-testnet-services.luckystar.asia/story/addrbook.json
```

### set custom ports in story.toml file

```
sed -i.bak -e "s%:1317%:${STORY_PORT}317%g;
s%:8551%:${STORY_PORT}551%g" $HOME/.story/story/config/story.toml
```

### set custom ports in config.toml file

```
sed -i.bak -e "s%:26658%:${STORY_PORT}658%g;
s%:26657%:${STORY_PORT}657%g;
s%:26656%:${STORY_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${STORY_PORT}656\"%;
s%:26660%:${STORY_PORT}660%g" $HOME/.story/story/config/config.toml
```

### enable prometheus and disable indexing

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.story/story/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.story/story/config/config.toml
```

### create geth servie file

```
sudo tee /etc/systemd/system/story-geth.service > /dev/null <<EOF
[Unit]
Description=Story Geth
After=network-online.target

[Service]
User=$USER
ExecStart=$HOME/go/bin/geth --aeneid --syncmode full --port ${STORY_PORT}303 --http --http.api eth,net,web3,engine --http.vhosts '*' --http.addr 0.0.0.0 --http.port ${STORY_PORT}545 --authrpc.addr 127.0.0.1 --authrpc.port ${STORY_PORT}551 --authrpc.vhosts=* --ws --ws.api eth,web3,net,txpool --ws.addr 0.0.0.0 --ws.port ${STORY_PORT}546 --ws.origins '*'
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

### download snapshots

#### backup priv\_validator\_state.json

```
cp $HOME/.story/story/data/priv_validator_state.json $HOME/.story/story/priv_validator_state.json.backup
```

#### remove old data and unpack Story snapshot

```
rm -rf $HOME/.story/story/data
curl https://server-3.itrocket.net/testnet/story/story_2024-11-21_713429_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.story/story
```

#### restore priv\_validator\_state.json

```
mv $HOME/.story/story/priv_validator_state.json.backup $HOME/.story/story/data/priv_validator_state.json
```

#### delete geth data and unpack Geth snapshot

```
rm -rf $HOME/.story/geth/odyssey/geth/chaindata
mkdir -p $HOME/.story/geth/odyssey/geth
curl https://server-3.itrocket.net/testnet/story/geth_story_2024-11-21_713429_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.story/geth/odyssey/geth
```

#### enable and start geth, story

```
sudo systemctl daemon-reload
sudo systemctl enable story story-geth
sudo systemctl restart story-geth && sleep 5 && sudo systemctl restart story
```

#### check logs (in new Tmux)

```
journalctl -u story -u story-geth -f
```

#### Check sync status:

```
curl localhost:${STORY_PORT}657/status | jq
```

```
curl -s localhost:${STORY_PORT}657/status | jq .result.sync_info
```

```
curl -s localhost:${STORY_PORT}657/status | jq .result.sync_info.catching_up
```

## END SETUP:

### Note

**From now please do not stop and restart node before block 858,000 . Because it will run forcely with new binary.**

If you need to restart the node unexpectedly, please setup again:

#### Remove folder

```
rm -rf $HOME/.story/story/cosmovisor/upgrades/v0.13.0
```

#### Remove symlink:

```
rm $HOME/.story/story/cosmovisor/current
```

#### Setup symlink v0.12.1 back:

```
ln -s $HOME/.story/story/cosmovisor/upgrades/v0.12.1 $HOME/.story/story/cosmovisor/current
```

#### _Finally Start node & setup v0.13.0 with Cosmosvisor again._

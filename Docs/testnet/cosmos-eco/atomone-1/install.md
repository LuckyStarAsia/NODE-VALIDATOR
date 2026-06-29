# INSTALL

## GNOLAND <img src="../../../.gitbook/assets/Gnoland_400x400.png" alt="" data-size="line">

Find all important links at [https://gno.land/links](https://gno.land/links)\
[https://docs.gno.land/](https://docs.gno.land/)\
[https://gnops.io/articles/guides/become-testnet-validator/](https://gnops.io/articles/guides/become-testnet-validator/)

[https://x.com/\_gnoland](https://x.com/_gnoland)\
[https://discord.gg/gnoland](https://discord.gg/gnoland)\
[https://www.youtube.com/@\_gnoland](https://www.youtube.com/@_gnoland)[https://github.com/gnolang/gno/blob/chain/test13/misc/deployments/test13.gno.land/VALIDATOR.md](https://github.com/gnolang/gno/blob/chain/test13/misc/deployments/test13.gno.land/VALIDATOR.md)

[https://gnoscan.io/](https://gnoscan.io/)\
[https://rpc.test13.testnets.gno.land/](https://rpc.test13.testnets.gno.land/)

### Install

```
sudo apt update && sudo apt upgrade -y && \
sudo apt install nano tree curl tar wget clang pkg-config libssl-dev libleveldb-dev jq build-essential bsdmainutils git make ncdu screen unzip bc fail2ban htop lz4 gcc net-tools tmux just -y
```

#### Install GO

Go Installation (for even user)

```
cd $HOME
VER="1.23.1"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f $HOME/.bash_profile ] && touch $HOME/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
echo "export GOROOT=$(go1.21.13 env GOROOT) PATH=$GOROOT/bin:$PATH" >> $HOME/.bash_profile
source $HOME/.bash_profile
[ ! -d $HOME/go/bin ] && mkdir -p $HOME/go/bin
```

```
go version --long | tail
```

#### Install Binary

```
cd $HOME
rm -rf $HOME/gno
git clone https://github.com/gnolang/gno.git && \
cd $HOME/gno && \
git checkout chain/test13 && \
make install_gnokey && \
make -C gno.land install.gnoland && make -C contribs/gnogenesis install
```

```
gnoland version
```

Result

```
gnoland version: chain/test13
```

#### Set Vars:

```
echo "export WALLET="GnolandTest13"" >> $HOME/.bash_profile && \
echo "export MONIKER="Your-Moniker"" >> $HOME/.bash_profile && \
echo "export GNO_CHAIN_ID="test-13"" >> $HOME/.bash_profile && \
echo "export GNO_PORT="24"" >> $HOME/.bash_profile && \
echo 'export GNOROOT=$HOME/gno' >> $HOME/.bash_profile && \
source $HOME/.bash_profile
```

#### Config and Init App:

```
cd $HOME && \
gnoland secrets init && \
gnoland config init && \
gnoland config set rpc.laddr tcp://0.0.0.0:${GNO_PORT}657 && \
gnoland config set p2p.laddr tcp://0.0.0.0:${GNO_PORT}656 && \
gnoland config set proxy_app tcp://127.0.0.1:${GNO_PORT}658 && \
gnoland config set moniker $MONIKER && \
gnoland config set consensus.peer_gossip_sleep_duration 10ms && \
gnoland config set consensus.timeout_commit 3s && \
gnoland config set mempool.size 10000 && \
gnoland config set p2p.flush_throttle_timeout 10ms && \
gnoland config set p2p.max_num_outbound_peers 40 && \
gnoland config set p2p.persistent_peers g1lxkf9gn7kddrr26c640ww5wg3ezsm22we8cjpc@99.81.240.125:26656,g17su28ydtj8jsdqt2c7m3jn3mysqlz6n57vxd5t@62.210.125.225:26656,g1zz2yvc23ts7uk05gemxmj9dlrhdt7w8pxtcdy5@62.210.207.58:26656,g1uycj5lkvu97jddywjttd8xq53u3p6eyhh2js25@62.210.124.8:26656,g142k7zc2qym3c0u6jmkf6rv26llgr2f4nakmlmt@54.145.44.95:26656,g12gxe0qpq90vhhpp5gtavafgr2nl9cntntvrjkj@186.233.184.95:37656
```

#### Add Genesis File and Addrbook:

```
wget -O $HOME/gnoland-data/config/genesis.json https://snapshots.luckystar.asia/gnolandtest/genesis.json
```

#### Peers:

```
curl -sS https://gnoland-testnet13-rpc.luckystar.asia/net_info | jq -r '.result.peers | map(.node_info.net_address) | join(",")'
```

Result

```
g16f7dlfvrk6qugcdmaq5npw0sjhlf79vrz7ge6c@5.9.8.148:30800,g1rrj6gvxp7ph8d4rsy9vegrhp8nx4lyxzgmc4ad@65.109.124.135:54656,g1uud06624z4lhynkl0t62r5dhs6ksnqced2m9w5@148.251.2.253:36656,g1uycj5lkvu97jddywjttd8xq53u3p6eyhh2js25@62.210.124.8:26656,g17qp5xc8a607svp77h3ttl05mg4fuzyl5fyvc2r@37.60.255.34:27656,g1ghl8lhhdnhwwvp94z0gsadvz3c03et30cqyhu2@117.1.104.188:26656,g1lxkf9gn7kddrr26c640ww5wg3ezsm22we8cjpc@99.81.240.125:26656,g17t7vlg7hjvsldqj06zenkxpktzntkp62thg9a7@65.109.106.214:7656,g1kcdk47njpk6sezdv38wxpt2zruwmklkn33d57z@159.195.5.169:54656,g1d8qlrhwm97mz5dcjdrj5vgps9e33336nhg5tvk@157.180.97.166:17656,g1v5unyq7fsndekzegp2xh0tw45nsxpkx5qsgy8u@135.181.232.179:26656,g142k7zc2qym3c0u6jmkf6rv26llgr2f4nakmlmt@54.145.44.95:26656
```

#### Set Service File:

```
sudo tee /etc/systemd/system/gnolandtest.service > /dev/null <<EOF
[Unit]
Description=Gnoland node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME
Environment="GNOROOT=$HOME/gno"
ExecStart=$(which gnoland) start --genesis  $HOME/gnoland-data/config/genesis.json --data-dir $HOME/gnoland-data --skip-genesis-sig-verification
Restart=always
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

## Enable and Start Service:

```
sudo systemctl daemon-reload && \
sudo systemctl enable gnolandtest
sudo systemctl restart gnolandtest && sudo journalctl -u gnolandtest -f
```

```
sudo systemctl stop gnolandtest
sudo systemctl disable gnolandtest
```

#### Check Sync:

```
curl -s http://127.0.0.1:${GNO_PORT}657/status | jq '.result.sync_info'
```

#### Get validator address and public key

```
VALOPER=$(gnoland secrets get validator_key | jq -r '.address')
PUBKEY=$(gnoland secrets get validator_key | jq -r '.pub_key')
```

#### Var

```
echo "export ADDRESS="$ADDRESS"" >> $HOME/.bash_profile
echo "export VALOPER="$VALOPER"" >> $HOME/.bash_profile
echo "export PUBKEY="$PUBKEY"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Key management

#### Add new key

```
gnokey add $WALLET
```

#### Recover existing key

```
gnokey add $WALLET --recover
```

#### Wallet balance

```
gnokey query -remote "https://rpc.test13.testnets.gno.land" auth/accounts/${ADDRESS}
```

### VALIDATOR:

#### Validator detail:

```
DETAILS=$(cat <<'EOF'
your validator description
EOF
)
```

#### Create Validator

```
gnokey maketx call \
    -pkgpath "gno.land/r/gnops/valopers" \
    -func "Register" \
    -gas-fee 1000000ugnot \
    -gas-wanted 50000000 \
    -broadcast \
    -chainid "test-13" \
    -args "$MONIKER" \
    -args "$DETAILS" \
    -args "on-prem" \
    -args "$ADDRESS" \
    -args "$PUBKEY" \
    -remote "https://rpc.test13.testnets.gno.land:443" \
    $WALLET
```

### Remove node

_**- Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv\_validator\_key.json!**_

```
sudo systemctl stop gnolandtest.service && \
sudo systemctl disable gnolandtest.service && \
sudo rm -rf /etc/systemd/system/gnolandtest.service && \
sudo systemctl daemon-reload && \
sudo rm -f $(which gnoland) && \
sudo rm -f $(which gnokey) && \
sudo rm -rf $(which gnogenesis) && \
sudo rm -rf $HOME/gnoland-data && \
sudo rm -rf $HOME/gno
```

## END

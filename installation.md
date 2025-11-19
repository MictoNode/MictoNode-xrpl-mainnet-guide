# 🔌 Installation

## 1️⃣ Installation packages and dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### ➡️ Go Installation

```bash
cd $HOME
VER="1.23.4"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

## 2️⃣ Install node

> **Reminder**  
> You can change the port and wallet name.

```bash
echo "export XRPL_MAINNET_WALLET="mictowallet"" >> $HOME/.bash_profile
echo "export XRPL_MAINNET_PORT="47"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

```bash
mkdir -p $HOME/.exrpd/cosmovisor/upgrades/v8.0.0/bin
rm -rf xrpl-mainnet
git clone https://github.com/xrplevm/node.git xrpl-mainnet
cd xrpl-mainnet
git checkout v8.0.0
make build
mv $HOME/xrpl-mainnet/bin/exrpd $HOME/.exrpd/cosmovisor/upgrades/v8.0.0/bin/
```

```bash
sudo ln -sfn $HOME/.exrpd/cosmovisor/upgrades/v8.0.0 $HOME/.exrpd/cosmovisor/current
sudo ln -sfn $HOME/.exrpd/cosmovisor/current/bin/exrpd /usr/local/bin/exrpd
```

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
cd
```

### ➡️ Create a service

```bash
sudo tee /etc/systemd/system/exrpd.service > /dev/null <<EOF
[Unit]
Description=xrpl node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --home $HOME/.exrpd
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.exrpd"
Environment="DAEMON_NAME=exrpd"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.exrpd/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```

### ➡️ Let's activate it

```bash
sudo systemctl daemon-reload
sudo systemctl enable exrpd
```

### ➡️ Initialize the node & other

```bash
exrpd init "Moniker-Name" --chain-id xrplevm_144000-1
```

```bash
sed -i -e '/^chain-id = /c\chain-id = "xrplevm_144000-1"' $HOME/.exrpd/config/client.toml
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${XRPL_MAINNET_PORT}657\"|" $HOME/.exrpd/config/client.toml
sed -i -e '/^keyring-backend = /c\keyring-backend = "test"' $HOME/.exrpd/config/client.toml
```

### ➡️ Genesis addrbook

```bash
curl https://files.mictonode.com/xrpl-mainnet/genesis/genesis.json -o ~/.exrpd/config/genesis.json
```

> **Info**  
> Updated every 6 hours.

```bash
curl https://files.mictonode.com/xrpl-mainnet/addrbook/addrbook.json -o ~/.exrpd/config/addrbook.json
```

### ➡️ Port

```bash
sed -i.bak -e "s%:26658%:${XRPL_MAINNET_PORT}658%g;
s%:26657%:${XRPL_MAINNET_PORT}657%g;
s%:6060%:${XRPL_MAINNET_PORT}060%g;
s%:26656%:${XRPL_MAINNET_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${XRPL_MAINNET_PORT}656\"%;
s%:26660%:${XRPL_MAINNET_PORT}660%g" $HOME/.exrpd/config/config.toml
```

```bash
sed -i.bak -e "s%:1317%:${XRPL_MAINNET_PORT}317%g;
s%:8080%:${XRPL_MAINNET_PORT}080%g;
s%:9090%:${XRPL_MAINNET_PORT}090%g;
s%:9091%:${XRPL_MAINNET_PORT}091%g;
s%:8545%:${XRPL_MAINNET_PORT}545%g;
s%:8546%:${XRPL_MAINNET_PORT}546%g;
s%:6065%:${XRPL_MAINNET_PORT}065%g" $HOME/.exrpd/config/app.toml
```

### ➡️ Peers and Seeds

```bash
SEEDS=""
PEERS="2dc8d776176a275bc64421e31d9b1ed6af228fa5@seed-0.mainnet.xrplevm.org:26656,8f8183786ac761e90998d6ab582ab5c0b1cb8133@8.52.196.180:53656,3620f7e655bf094114e5301d9733323b0c283107@195.20.18.175:26656,5227628bdceb67b8d57f40a8b406568882b8fb8d@168.119.143.51:10656,e1279ea70948c34cd676ca9d69bf28a452cfbe03@54.39.128.229:26636"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.exrpd/config/config.toml
```

### ➡️ Pruning

```bash
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.exrpd/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.exrpd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" $HOME/.exrpd/config/app.toml
```

### ➡️ Indexer & Other

```bash
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.exrpd/config/config.toml
```

### ➡️ Gas Settings

```bash
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0axrp"|g' $HOME/.exrpd/config/app.toml
```

### ➡️ Prometheus

```bash
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.exrpd/config/config.toml
```

### ➡️ Starter Snap

Check snapshot height
```bash
echo "XRPL Snapshot Height: $(curl -s https://files.mictonode.com/xrpl-mainnet/snapshot/block-height.txt)"
```

```bash
exrpd tendermint unsafe-reset-all --home $HOME/.exrpd --keep-addr-book

SNAPSHOT_URL="https://files.mictonode.com/xrpl-mainnet/snapshot/"
LATEST_SNAPSHOT=$(curl -s $SNAPSHOT_URL | grep -oP 'xrpl-mainnet_\d+\.tar\.lz4' | sort -t_ -k2 -n | tail -n 1)

if [ -n "$LATEST_SNAPSHOT" ]; then
  FULL_URL="${SNAPSHOT_URL}${LATEST_SNAPSHOT}"
  if curl -s --head "$FULL_URL" | head -n 1 | grep "200" > /dev/null; then
    curl "$FULL_URL" | lz4 -dc - | tar -xf - -C $HOME/.exrpd
  else
    echo "Snapshot URL not accessible"
  fi
else
  echo "No snapshot found"
fi
```

### ➡️ Let's get started

```bash
sudo systemctl restart exrpd
sudo journalctl -fu exrpd -o cat
```

### ➡️ Log Command

```bash
sudo journalctl -fu exrpd -o cat
```

### ➡️ Create wallet

> **Warning**  
> Don't forget to backup the wallet words!

```bash
exrpd keys add $XRPL_MAINNET_WALLET
```

### ➡️ Import wallet

```bash
exrpd keys add $XRPL_MAINNET_WALLET --recover
```

### ➡️ Import private key

```bash
exrpd keys unsafe-export-eth-key $XRPL_MAINNET_WALLET
```

### ➡️ Edit Validator

```bash
exrpd tx staking edit-validator \
--chain-id xrplevm_144000-1 \
--commission-rate 0.05 \
--new-moniker "validator-name" \
--identity "" \
--details "" \
--website "" \
--security-contact "" \
--from $XRPL_MAINNET_WALLET \
--node http://localhost:${XRPL_MAINNET_PORT}657 \
--gas="300000" --gas-prices="240000000000900000axrp" \
-y
```

### ➡️ Complete deletion

```bash
cd $HOME
sudo systemctl stop exrpd
sudo systemctl disable exrpd
sudo rm -rf /etc/systemd/system/exrpd.service
sudo systemctl daemon-reload
sudo rm -f /usr/local/bin/exrpd
sudo rm -f $(which exrpd)
sudo rm -rf $HOME/.exrpd
sed -i "/XRPL_MAINNET_PORT_/d" $HOME/.bash_profile
```

### ➡️ Block check

```bash
local_height=$(exrpd status | jq -r .sync_info.latest_block_height); network_height=$(curl -s https://xrpl-mainnet-rpc.mictonode.com/status | jq -r .result.sync_info.latest_block_height); blocks_left=$((network_height - local_height)); echo "Your node height: $local_height"; echo "MictoNode height: $network_height"; echo "Blocks left: $blocks_left"
```

* Your node height - the current block of your node
* Network height - the last block of the network
* Blocks left - how many blocks your node has left to sync.
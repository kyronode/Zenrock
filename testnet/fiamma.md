---
icon: arrows-to-circle
---

# Fiamma

### Fiamma Node Setup Guide

Recommended Hardware: \
CPU: 4 Cores \
RAM: 32GB \
Storage: 1000GB&#x20;

This guide will help you set up a Fiamma testnet node step by step.

***

### 1. Install Dependencies

Run the following commands to install the required dependencies:

```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

***

### 2. Install Go

Follow these steps to install Go:

```
cd $HOME
VER="1.22.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

***

### 3. Set Environment Variables

Set up the environment variables for your node:

```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export FIAMMA_CHAIN_ID="fiamma-testnet-1"" >> $HOME/.bash_profile
echo "export FIAMMA_PORT="37"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

***

### 4. Download and Build Fiamma Binary

Clone the repository and build the binary:

```
cd $HOME
rm -rf fiamma
git clone https://github.com/fiamma-chain/fiamma.git
cd fiamma
git checkout v1.0.0
make install
```

***

### 5. Initialize the Node

Initialize the Fiamma node:

```
fiammad init $MONIKER --chain-id $FIAMMA_CHAIN_ID
```

Configure the node settings:

```
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${FIAMMA_PORT}657\"|" $HOME/.fiamma/config/client.toml
sed -i -e "s|^keyring-backend *=.*|keyring-backend = \"os\"|" $HOME/.fiamma/config/client.toml
sed -i -e "s|^chain-id *=.*|chain-id = \"fiamma-testnet-1\"|" $HOME/.fiamma/config/client.toml
```

***

### 6. Download Genesis and Addrbook

Download the genesis file and addrbook:

```
wget -O $HOME/.fiamma/config/genesis.json https://server-5.itrocket.net/testnet/fiamma/genesis.json
wget -O $HOME/.fiamma/config/addrbook.json https://server-5.itrocket.net/testnet/fiamma/addrbook.json
```

***

### 7. Configure Node

Set seeds and peers:

```
SEEDS="1e8777199f1edb3a35937e653b0bb68422f3c931@fiamma-testnet-seed.itrocket.net:50656"
PEERS="16b7389e724cc440b2f8a2a0f6b4c495851934ff@fiamma-testnet-peer.itrocket.net:49656,bf2c0595ffb5af59a97febf092720bd841ee0a54@65.109.84.235:50656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.fiamma/config/config.toml
```

Set custom ports in `app.toml`:

```
sed -i.bak -e "s%:1317%:${FIAMMA_PORT}317%g;
s%:8080%:${FIAMMA_PORT}080%g;
s%:9090%:${FIAMMA_PORT}090%g;
s%:9091%:${FIAMMA_PORT}091%g;
s%:8545%:${FIAMMA_PORT}545%g;
s%:8546%:${FIAMMA_PORT}546%g;
s%:6065%:${FIAMMA_PORT}065%g" $HOME/.fiamma/config/app.toml
```

Set custom ports in `config.toml`:

```
sed -i.bak -e "s%:26658%:${FIAMMA_PORT}658%g;
s%:26657%:${FIAMMA_PORT}657%g;
s%:6060%:${FIAMMA_PORT}060%g;
s%:26656%:${FIAMMA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${FIAMMA_PORT}656\"%;
s%:26660%:${FIAMMA_PORT}660%g" $HOME/.fiamma/config/config.toml
```

Configure pruning and minimum gas price:

```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.fiamma/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.fiamma/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.fiamma/config/app.toml
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = \"0.00001ufia\"|g' $HOME/.fiamma/config/app.toml
```

***

### 8. Set Up Service

Create a service file for the node:

```
sudo tee /etc/systemd/system/fiammad.service > /dev/null <<EOF
[Unit]
Description=Fiamma Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.fiamma
ExecStart=$(which fiammad) start --home $HOME/.fiamma
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

Enable and start the service:

```
sudo systemctl daemon-reload
sudo systemctl enable fiammad
sudo systemctl start fiammad
sudo journalctl -u fiammad -f
```

***

### 9. Create Wallet

Create a new wallet:

Restore an existing wallet:

```
fiammad keys add $WALLET --recover
```

Save wallet and validator address:

```
WALLET_ADDRESS=$(fiammad keys show $WALLET -a)
VALOPER_ADDRESS=$(fiammad keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS=$WALLET_ADDRESS" >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS=$VALOPER_ADDRESS" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

***

### 10. Check Node Sync Status

Check the sync status:

***

### 11. Delete Node

If needed, delete the node with the following commands:

```
sudo systemctl stop fiammad
sudo systemctl disable fiammad
sudo rm -rf /etc/systemd/system/fiammad.service
sudo rm $(which fiammad)
sudo rm -rf $HOME/.fiamma
sed -i "/FIAMMA_/d" $HOME/.bash_profile
```

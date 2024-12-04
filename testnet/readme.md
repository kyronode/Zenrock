---
icon: arrows-to-circle
---

# Zenrock

## ⚙️ Installation Guide

### Official Documentation

**Recommended Hardware:**

* 4 Cores, 8GB RAM, 200GB of storage (NVME)

***

### Manual Installation

#### Step 1: Install Dependencies

Run the following commands to install required dependencies:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### Step 2: Install Go (if needed)

```bash
cd $HOME
VER="1.23.1"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"

[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### Step 3: Set Environment Variables

## Set environment variables

```bash
echo "export WALLET='wallet'" >> $HOME/.bash_profile
echo "export MONIKER='test'" >> $HOME/.bash_profile
echo "export ZENROCK_CHAIN_ID='gardia-2'" >> $HOME/.bash_profile
echo "export ZENROCK_PORT='56'" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Step 4: Download and Install Zenrock Binary

```bash
cd $HOME
curl -o zenrockd https://releases.gardia.zenrocklabs.io/zenrockd-latest
chmod +x $HOME/zenrockd
mv $HOME/zenrockd $HOME/go/bin/
```

### Step 5: Configure and Initialize App

```bash
zenrockd init $MONIKER --chain-id $ZENROCK_CHAIN_ID
zenrockd config set client chain-id $ZENROCK_CHAIN_ID
zenrockd config set client node tcp://localhost:${ZENROCK_PORT}657
```

### Step 6: Download Genesis and Addrbook

```bash
wget -O $HOME/.zrchain/config/genesis.json https://snapshot.node9x.com/zrchain/genesis.json
wget -O $HOME/.zrchain/config/addrbook.json https://snapshot.node9x.com/zrchain/addrbook.json
```

### Step 7: Set Seeds and Peers

```bash
SEEDS="50ef4dd630025029dde4c8e709878343ba8a27fa@zenrock-testnet-seed.itrocket.net:56656"
PEERS="5458b7a316ab673afc34404e2625f73f0376d9e4@65.108.132.123:11656,..."
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.zrchain/config/config.toml
```

### Step 8: Set Custom Ports in app.toml

```bash
sed -i.bak -e "s%:1317%:${ZENROCK_PORT}317%g; ... " $HOME/.zrchain/config/app.toml
```

### Step 9: Set Pruning Configuration

```bash
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.zrchain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.zrchain/config/app.toml
```

### Step 10: Configure Gas Price and Indexing

```bash
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0urock"|g' $HOME/.zrchain/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.zrchain/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.zrchain/config/config.toml
```

### Step 11: Create Service File

```bash
sudo tee /etc/systemd/system/zenrockd.service > /dev/null <<EOF
[Unit]
Description=Zenrock node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.zrchain
ExecStart=$(which zenrockd) start --home $HOME/.zrchain
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### Step 12: Reset and Download Snapshot

```bash
zenrockd tendermint unsafe-reset-all --home $HOME/.zrchain
if curl -s --head curl https://snapshot.node9x.com/zenrock_testnet.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://snapshot.node9x.com/zenrock_testnet.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.zrchain
else
  echo "no snapshot founded"
fi
```

### Step 13: Enable and Start Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable zenrockd
sudo systemctl restart zenrockd && sudo journalctl -u zenrockd -f
```

### Create Wallet

#### Step 1: Create a New Wallet

```bash
zenrockd keys add $WALLET
```

#### Step 2: Restore Existing Wallet

```bash
zenrockd keys add $WALLET --recover
```

#### Step 3: Save Wallet and Validator Address

```bash
WALLET_ADDRESS=$(zenrockd keys show $WALLET -a)
VALOPER_ADDRESS=$(zenrockd keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS=$WALLET_ADDRESS" >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS=$VALOPER_ADDRESS" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Check Sync Status

```bash
zenrockd status 2>&1 | jq
```

### Claim Token on Faucet

Visit the faucet here: https://gardia.zenrocklabs.io/workspaces

Or use this command:

```bash
curl https://faucet.gardia.zenrocklabs.io -XPOST -d'{"address":"zen1steffht....teqf"}'
```

### Node Sync Status Checker

```bash
#!/bin/bash
rpc_port=$(grep -m 1 -oP '^laddr = "\K[^"]+' "$HOME/.zrchain/config/config.toml" | cut -d ':' -f 3)
while true; do
  local_height=$(curl -s localhost:$rpc_port/status | jq -r '.result.sync_info.latest_block_height')
  network_height=$(curl -s https://zenrock-testnet-rpc.itrocket.net/status | jq -r '.result.sync_info.latest_block_height')

  if ! [[ "$local_height" =~ ^[0-9]+$ ]] || ! [[ "$network_height" =~ ^[0-9]+$ ]]; then
    echo -e "\033[1;31mError: Invalid block height data. Retrying...\033[0m"
    sleep 5
    continue
  fi

  blocks_left=$((network_height - local_height))
  if [ "$blocks_left" -lt 0 ]; then
    blocks_left=0
  fi

  echo -e "\033[1;33mNode Height:\033[1;34m $local_height\033[0m \033[1;33m| Network Height:\033[1;36m $network_height\033[0m \033[1;33m| Blocks Left:\033[1;31m $blocks_left\033[0m"

  sleep 5
done
```

### Create Validator

```bash
cd $HOME
```

**Create validator.json file**

```bash
echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(zenrockd comet show-validator | grep -Po '\"key\":\s*\"\K[^"]*')\"},
    \"amount\": \"1000000urock\",
    \"moniker\": \"test\",
    \"identity\": \"\",
    \"website\": \"\",
    \"security\": \"\",
    \"details\": \"I love blockchain ❤️\",
    \"commission-rate\": \"0.1\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\" 
}" > validator.json
```

### Create a validator using the JSON configuration

```bash
zenrockd tx staking create-validator \
--amount 1000000urock \
--pubkey $(zenrockd tendermint show-validator) \
--moniker $MONIKER \
--chain-id $ZENROCK_CHAIN_ID \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from $WALLET
```

### DONE

Thanks to: [NodeX9](https://node9x.com/)

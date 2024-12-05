---
icon: arrows-to-circle
---

# Pryzm

### PRYSM

Welcome to PRYSM

PRYSM is a blockchain that strives to challenge the limits of decentralized networks by integrating a combination of the finest features of existing blockchain technologies with new innovative ideas. A thriving diverse community is crucial for the success of any project, and PRYSM aims to achieve this by building a user-friendly ecosystem that is accessible to all. To this end, PRYSM will provide clear documentation that simplifies participation in the network for anyone who wants to take part.

***

### Official Links

* **Website**: [http://Prysm.Network](http://prysm.network/)
* **Twitter**: [https://twitter.com/PrysmNetwork](https://twitter.com/PrysmNetwork)
* **Telegram**: [https://t.me/PrysmNetwork](https://t.me/PrysmNetwork)
* **YouTube**: [https://www.youtube.com/@PrysmNetwork](https://www.youtube.com/@PrysmNetwork)
* **Medium**: [https://medium.com/@prysmnetwork](https://medium.com/@prysmnetwork)

***

### Services

* **Explorer**: [https://www.explorer.winnode.xyz/prysm/](https://www.explorer.winnode.xyz/prysm/)
* **RPC**: [https://rpc-prysm.winnode.xyz/](https://rpc-prysm.winnode.xyz/)
* **API**: [https://api-prysm.winnode.xyz/](https://api-prysm.winnode.xyz/)

***

### Manual Installation

#### **Recommended Hardware**

* **CPU**: 4 Cores
* **RAM**: 8 GB
* **Storage**: 200 GB (NVME preferred)

#### **Dependencies Installation**

```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

#### **Install Go**

```
cd $HOME
VER="1.22.6"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

#### **Set Variables**

```
echo "export WALLET=\"wallet\"" >> $HOME/.bash_profile
echo "export MONIKER=\"test\"" >> $HOME/.bash_profile
echo "export PRYSM_CHAIN_ID=\"prysm-devnet-1\"" >> $HOME/.bash_profile
echo "export PRYSM_PORT=\"25\"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### **Download Binary**

```
cd $HOME
rm -rf prysm
git clone https://github.com/kleomedes/prysm prysm
cd prysm
git checkout v0.1.0-devnet
make install
```

#### **Configuration**

```
prysmd config node tcp://localhost:${PRYSM_PORT}657
prysmd config keyring-backend os
prysmd config chain-id $PRYSM_CHAIN_ID
prysmd init $MONIKER --chain-id $PRYSM_CHAIN_ID
```

#### **Download Genesis and Addrbook**

```
wget -O $HOME/.prysm/config/genesis.json https://filex.winnode.xyz/snapshot/prysm/genesis.json
wget -O $HOME/.prysm/config/addrbook.json https://filex.winnode.xyz/snapshot/prysm/addrbook.json
```

#### **Set Seeds and Peers**

```
SEEDS="1b5b6a532e24c91d1bc4491a6b989581f5314ea5@prysm-testnet-seed.itrocket.net:25656"
PEERS="ff15df83487e4aa8d2819452063f336269958d09@prysm-testnet-peer.itrocket.net:25657,ea436c798572aef13813b5f9eeda24abb196569f@62.169.23.220:29656"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.prysm/config/config.toml
```

#### **Pruning Configuration**

```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.prysm/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.prysm/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"50\"/" $HOME/.prysm/config/app.toml
```

#### **Minimum Gas Price and Prometheus**

```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = \"0.0uprysm\"|g' $HOME/.prysm/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.prysm/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.prysm/config/config.toml
```

#### **Create Service File**

```
sudo tee /etc/systemd/system/prysmd.service > /dev/null <<EOF
[Unit]
Description=Prysm Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.prysm
ExecStart=$(which prysmd) start --home $HOME/.prysm
Restart=on-failure
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

#### **Reset and Download Snapshot**

```
prysmd tendermint unsafe-reset-all --home $HOME/.prysm
curl https://filex.winnode.xyz/snapshot/prysm/prysm-latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.prysm
```

#### **Enable and Start Service**

```
sudo systemctl daemon-reload
sudo systemctl enable prysmd
sudo systemctl restart prysmd
sudo journalctl -u prysmd -f
```

***

### Create Wallet

#### **Create New Wallet**

#### **Restore Wallet**

```
prysmd keys add $WALLET --recover
```

#### **Save Wallet and Validator Address**

```
WALLET_ADDRESS=$(prysmd keys show $WALLET -a)
VALOPER_ADDRESS=$(prysmd keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS=$WALLET_ADDRESS" >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS=$VALOPER_ADDRESS" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

***

### **Node Sync Status Checker**

```
#!/bin/bash
rpc_port=$(grep -m 1 -oP '^laddr = "\K[^"]+' "$HOME/.prysm/config/config.toml" | cut -d ':' -f 3)

while true; do
  local_height=$(curl -s localhost:$rpc_port/status | jq -r '.result.sync_info.latest_block_height')
  network_height=$(curl -s https://rpc-prysm.winnode.xyz/status | jq -r '.result.sync_info.latest_block_height')

  if ! [[ "$local_height" =~ ^[0-9]+$ ]] || ! [[ "$network_height" =~ ^[0-9]+$ ]]; then
    echo -e "\033[31mError: Invalid data. Retrying...\033[0m"
    sleep 5
    continue
  fi

  blocks_left=$((network_height - local_height))
  [ "$blocks_left" -lt 0 ] && blocks_left=0

  echo -e "\033[33mNode:\033[34m $local_height\033[0m \033[33m| Network:\033[36m $network_height\033[0m \033[33m| Left:\033[31m $blocks_left\033[0m"

  sleep 5
done
```

### **Create validator**

```
cd $HOME
# Create validator.json file
echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(prysmd comet show-validator | grep -Po '\"key\":\s*\"\K[^"]*')\"},
    \"amount\": \"1000000uprysm\",
    \"moniker\": \"test\",
    \"identity\": \"\",
    \"website\": \"\",
    \"security\": \"\",
    \"details\": \"\",
    \"commission-rate\": \"0.1\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\"
}" > validator.json
# Create a validator using the JSON configuration
prysmd tx staking create-validator validator.json \
    --from $WALLET \
    --chain-id prysm-devnet-1 \
	--gas auto --gas-adjustment 1.5 \
```

### **Firewall security**

```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw allow 25656/tcp
sudo ufw enable
```

### **Delete node**

```
sudo systemctl stop prysmd
sudo systemctl disable prysmd
sudorm -rf /etc/systemd/system/prysmd.service
sudorm$(which prysmd)sudorm -rf $HOME/.prysm
sed -i "/PRYSM_/d"$HOME/.bash_profile
```

### **Done**

---
description: Pell node guide
icon: arrows-to-circle
---

# Pell

## **Pell Node Setup Guide**

### **1. Install Dependencies**

Run the following commands to update your system and install necessary dependencies:

```sh
shCopyEditsudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### **2. Install Go**

```sh
shCopyEditcd $HOME
VER="1.22.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
mkdir -p ~/go/bin
```

### **3. Set Environment Variables**

```sh
shCopyEditecho "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export PELL_CHAIN_ID="ignite_186-1"" >> $HOME/.bash_profile
echo "export PELL_PORT="58"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### **4. Download Binary and WASMVM**

```sh
shCopyEditcd $HOME
wget -O pellcored https://github.com/0xPellNetwork/network-config/releases/download/v1.2.1/pellcored-v1.2.1-linux-amd64
chmod +x pellcored
mv pellcored ~/go/bin/

WASMVM_VERSION=v2.1.2
export LD_LIBRARY_PATH=~/.pellcored/lib
mkdir -p $LD_LIBRARY_PATH
wget "https://github.com/CosmWasm/wasmvm/releases/download/$WASMVM_VERSION/libwasmvm.$(uname -m).so" -O "$LD_LIBRARY_PATH/libwasmvm.$(uname -m).so"
echo "export LD_LIBRARY_PATH=$HOME/.pellcored/lib:$LD_LIBRARY_PATH" >> ~/.bash_profile
source ~/.bash_profile
```

### **5. Configure and Initialize Node**

```sh
shCopyEditpellcored config node tcp://localhost:${PELL_PORT}657
pellcored config keyring-backend os
pellcored config chain-id ignite_186-1
pellcored init "test" --chain-id ignite_186-1
```

### **6. Download Genesis and Addrbook**

```sh
shCopyEditwget -O $HOME/.pellcored/config/genesis.json https://server-5.itrocket.net/testnet/pell/genesis.json
wget -O $HOME/.pellcored/config/addrbook.json https://server-5.itrocket.net/testnet/pell/addrbook.json
```

### **7. Set Seeds and Peers**

```sh
shCopyEditSEEDS="5f10959cc96b5b7f9e08b9720d9a8530c3d08d19@pell-testnet-seed.itrocket.net:58656"
PEERS="d003cb808ae91bad032bb94d19c922fe094d8556@pell-testnet-peer.itrocket.net:58656,28c0fcd184c31ac7f3e2b3a91ae60dedc086b0c3@94.130.204.227:26656,1189d15c84a5b79a95cdf0cc65d007c6aa85b66d@49.12.82.124:26656,f9decf3cb78d88581d6c3c14828cb906a75fd060@95.216.97.238:26656,32fac46251436c7bee07b9aa5571f69b5fb765f4@193.34.212.164:57656"

sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.pellcored/config/config.toml
```

### **8. Configure Custom Ports**

```sh
shCopyEditsed -i.bak -e "s%:1317%:${PELL_PORT}317%g;
s%:8080%:${PELL_PORT}080%g;
s%:9090%:${PELL_PORT}090%g;
s%:9091%:${PELL_PORT}091%g;
s%:8545%:${PELL_PORT}545%g;
s%:6065%:${PELL_PORT}065%g" $HOME/.pellcored/config/app.toml
```

### **9. Set Pruning and Gas Price**

```sh
shCopyEditsed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.pellcored/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.pellcored/config/app.toml
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0apell"|g' $HOME/.pellcored/config/app.toml
```

### **10. Create and Start System Service**

```sh
shCopyEditsudo tee /etc/systemd/system/pellcored.service > /dev/null <<EOF
[Unit]
Description=Pell Node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.pellcored
ExecStart=$(which pellcored) start --home $HOME/.pellcored
Environment=LD_LIBRARY_PATH=$HOME/.pellcored/lib/
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable pellcored
sudo systemctl restart pellcored
sudo journalctl -u pellcored -fo cat
```

### **11. Check Sync Status**

```sh
shCopyEditpellcored status 2>&1 | jq
```

### **12. Create Wallet**

```sh
shCopyEditpellcored keys add $WALLET
pellcored keys add $WALLET --recover  # Restore wallet
```

### **13. Save Wallet and Validator Address**

```sh
shCopyEditWALLET_ADDRESS=$(pellcored keys show $WALLET -a)
VALOPER_ADDRESS=$(pellcored keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### **14. Create Validator**

```sh
shCopyEditcd $HOME
echo "{\"pubkey\":{\"@type\":\"/cosmos.crypto.ed25519.PubKey\",\"key\":\"$(pellcored comet show-validator | grep -Po '\"key\":\s*\"\K[^"]*')\"},
    \"amount\": \"1000000apell\",
    \"moniker\": \"test\",
    \"identity\": \"\",
    \"website\": \"\",
    \"security\": \"\",
    \"details\": \"I love blockchain ❤️\",
    \"commission-rate\": \"0.1\",
    \"commission-max-rate\": \"0.2\",
    \"commission-max-change-rate\": \"0.01\",
    \"min-self-delegation\": \"1\"}" > validator.json

pellcored tx staking create-validator validator.json \
    --from $WALLET \
    --chain-id ignite_186-1 \
    --fees 30apell --gas 300000
```

### **15. Security & Firewall**

```sh
shCopyEditsudo ufw default allow outgoing 
sudo ufw default deny incoming 
sudo ufw allow ssh/tcp 
sudo ufw allow ${PELL_PORT}656/tcp
sudo ufw enable
```

### **16. Delete Node**

```sh
shCopyEditsudo systemctl stop pellcored
sudo systemctl disable pellcored
sudo rm -rf /etc/systemd/system/pellcored.service
sudo rm $(which pellcored)
sudo rm -rf $HOME/.pellcored
sed -i "/PELL_/d" $HOME/.bash_profile
```

---
description: Galactica node guides
icon: arrows-to-circle
---

# Galactica

## **Galactica Node Installation Guide**

### **System Requirements**

Before starting, make sure you have a **VPS with Ubuntu 20.04/22.04** and at least:

* 4 vCPU
* 8GB RAM
* 100GB SSD

### **Step 1: Install Dependencies**

Run the following commands to install necessary dependencies:

```bash
bashCopyEditsudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### **Step 2: Install Go (if not installed)**

```bash
bashCopyEditcd $HOME
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
mkdir -p ~/go/bin
```

### **Step 3: Set Environment Variables**

```bash
bashCopyEditecho "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export GALACTICA_CHAIN_ID="galactica_9302-1"" >> $HOME/.bash_profile
echo "export GALACTICA_PORT="46"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### **Step 4: Download and Install the Galactica Binary**

```bash
bashCopyEditcd $HOME
rm -rf bin
mkdir bin
cd $HOME/bin
wget https://github.com/Galactica-corp/galactica/releases/download/v0.2.4/galacticad_v0.2.4-linux_amd64.zip
unzip galacticad_v0.2.4-linux_amd64.zip
chmod +x $HOME/bin/galacticad
sudo mv $HOME/bin/galacticad $HOME/go/bin
```

### **Step 5: Configure and Initialize the Node**

```bash
bashCopyEditgalacticad config node tcp://localhost:${GALACTICA_PORT}657
galacticad config keyring-backend os
galacticad config chain-id galactica_9302-1
galacticad init "test" --chain-id galactica_9302-1
```

### **Step 6: Download Genesis and Addrbook**

```bash
bashCopyEditwget -O $HOME/.galactica/config/genesis.json https://server-4.itrocket.net/testnet/galactica/genesis.json
wget -O $HOME/.galactica/config/addrbook.json  https://server-4.itrocket.net/testnet/galactica/addrbook.json
```

### **Step 7: Set Peers and Seeds**

```bash
bashCopyEditSEEDS="52ccf467673f93561c9d5dd4434def32ef2cd7f3@galactica-testnet-seed.itrocket.net:46656"
PEERS="c9993c738bec6a10cfb8bb30ac4e9ae6f8286a5b@galactica-testnet-peer.itrocket.net:11656,f3cd6b6ebf8376e17e630266348672517aca006a@46.4.5.45:27456"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.galactica/config/config.toml
```

### **Step 8: Configure Custom Ports**

```bash
bashCopyEditsed -i.bak -e "s%:1317%:${GALACTICA_PORT}317%g;
s%:8080%:${GALACTICA_PORT}080%g;
s%:9090%:${GALACTICA_PORT}090%g;" $HOME/.galactica/config/app.toml
```

### **Step 9: Configure Pruning and Security Settings**

```bash
bashCopyEditsed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.galactica/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.galactica/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.galactica/config/app.toml

sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "10agnet"|g' $HOME/.galactica/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.galactica/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.galactica/config/config.toml
```

### **Step 10: Create a Systemd Service File**

```bash
bashCopyEditsudo tee /etc/systemd/system/galacticad.service > /dev/null <<EOF
[Unit]
Description=Galactica node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.galactica
ExecStart=$(which galacticad) start --home $HOME/.galactica --chain-id galactica_9302-1
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### **Step 11: Reset and Download Latest Snapshot**

```bash
bashCopyEditgalacticad tendermint unsafe-reset-all --home $HOME/.galactica
curl https://server-4.itrocket.net/testnet/galactica/galactica_2025-02-03_4488910_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.galactica
```

### **Step 12: Start and Enable the Node**

```bash
bashCopyEditsudo systemctl daemon-reload
sudo systemctl enable galacticad
sudo systemctl restart galacticad && sudo journalctl -u galacticad -fo cat
```

***

### **Wallet Creation & Node Monitoring**

#### **Create a Wallet**

```bash
bashCopyEditgalacticad keys add $WALLET
```

Save the **mnemonic** phrase securely.

To restore a wallet:

```bash
bashCopyEditgalacticad keys add $WALLET --recover
```

#### **Check Sync Status**

```bash
bashCopyEditgalacticad status 2>&1 | jq 
```

Once your node is fully synced, the output will print **"false"**.

***

### **Create a Validator**

Before creating a validator, ensure your wallet has sufficient funds:

```bash
bashCopyEditgalacticad query bank balances $WALLET_ADDRESS
```

To create a validator:

```bash
bashCopyEditgalacticad tx staking create-validator \
--amount 1000000agnet \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(galacticad tendermint show-validator) \
--moniker "test" \
--identity "" \
--website "" \
--details "I love blockchain ❤️" \
--chain-id galactica_9302-1 \
--gas 200000 --gas-prices 10agnet \
-y
```

***

### **Security & Firewall Configuration**

Set up SSH key authentication and disable password authentication for better security.

Enable firewall settings:

```bash
bashCopyEditsudo ufw default allow outgoing 
sudo ufw default deny incoming 
sudo ufw allow ssh/tcp 
sudo ufw allow ${GALACTICA_PORT}656/tcp
sudo ufw enable
```

***

### **Uninstall Node**

If you want to remove the node:

```bash
bashCopyEditsudo systemctl stop galacticad
sudo systemctl disable galacticad
sudo rm -rf /etc/systemd/system/galacticad.service
sudo rm $(which galacticad)
sudo rm -rf $HOME/.galactica
sed -i "/GALACTICA_/d" $HOME/.bash_profile
```

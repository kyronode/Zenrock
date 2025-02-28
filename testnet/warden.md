---
description: Warden node guides
icon: arrows-to-circle
---

# Warden

## Install dependencies, if needed

sudo apt update && sudo apt upgrade -y sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y

## Install Go, if needed

cd $HOME VER="1.22.5" wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz" sudo rm -rf /usr/local/go sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz" rm "go$VER.linux-amd64.tar.gz" \[ ! -f \~/.bash\_profile ] && touch ~~/.bash\_profile echo "export PATH=$PATH:/usr/local/go/bin:~~/go/bin" >> \~/.bash\_profile source $HOME/.bash\_profile \[ ! -d \~/go/bin ] && mkdir -p \~/go/bin

## Set variables

echo "export WALLET="wallet"" >> $HOME/.bash\_profile echo "export MONIKER="test"" >> $HOME/.bash\_profile echo "export WARDEN\_CHAIN\_ID="chiado\_10010-1"" >> $HOME/.bash\_profile echo "export WARDEN\_PORT="18"" >> $HOME/.bash\_profile source $HOME/.bash\_profile

## Download binary

cd $HOME rm -rf bin mkdir bin && cd bin wget https://github.com/warden-protocol/wardenprotocol/releases/download/v0.5.4/wardend\_Linux\_x86\_64.zip unzip wardend\_Linux\_x86\_64.zip chmod +x wardend mv $HOME/bin/wardend $HOME/go/bin

## Config and init app

wardend init $MONIKER sed -i -e "s|^node _=._|node = "tcp://localhost:${WARDEN\_PORT}657"|" $HOME/.warden/config/client.toml

## Download genesis and addrbook

wget -O $HOME/.warden/config/genesis.json https://server-4.itrocket.net/testnet/warden/genesis.json wget -O $HOME/.warden/config/addrbook.json https://server-4.itrocket.net/testnet/warden/addrbook.json

## Set seeds and peers

SEEDS="8288657cb2ba075f600911685670517d18f54f3b@warden-testnet-seed.itrocket.net:18656" PEERS="b14f35c07c1b2e58c4a1c1727c89a5933739eeea@warden-testnet-peer.itrocket.net:18656,248a90408700cca7acc2f449252dc67ab3f9aec5@65.109.30.35:19656,cd62842978a2a35207d6790494d69916a0f539f6@144.76.70.103:13656" sed -i -e "/^\[p2p]/,/^\[/{s/^\[\[:space:]]\*seeds _=._/seeds = "$SEEDS"/}"\
-e "/^\[p2p]/,/^\[/{s/^\[\[:space:]]\*persistent\_peers _=._/persistent\_peers = "$PEERS"/}" $HOME/.warden/config/config.toml

## Set custom ports in app.toml

sed -i.bak -e "s%:1317%:${WARDEN\_PORT}317%g; s%:8080%:${WARDEN\_PORT}080%g; s%:9090%:${WARDEN\_PORT}090%g; s%:9091%:${WARDEN\_PORT}091%g; s%:8545%:${WARDEN\_PORT}545%g; s%:8546%:${WARDEN\_PORT}546%g; s%:6065%:${WARDEN\_PORT}065%g" $HOME/.warden/config/app.toml

## Set custom ports in config.toml file

sed -i.bak -e "s%:26658%:${WARDEN\_PORT}658%g; s%:26657%:${WARDEN\_PORT}657%g; s%:6060%:${WARDEN\_PORT}060%g; s%:26656%:${WARDEN\_PORT}656%g; s%^external\_address = ""%external\_address = "$(wget -qO- eth0.me):${WARDEN\_PORT}656"%; s%:26660%:${WARDEN\_PORT}660%g" $HOME/.warden/config/config.toml

## Config pruning

sed -i -e "s/^pruning _=._/pruning = "custom"/" $HOME/.warden/config/app.toml sed -i -e "s/^pruning-keep-recent _=._/pruning-keep-recent = "100"/" $HOME/.warden/config/app.toml sed -i -e "s/^pruning-interval _=._/pruning-interval = "19"/" $HOME/.warden/config/app.toml

## Set minimum gas price, enable Prometheus, and disable indexing

sed -i 's|minimum-gas-prices =.\*|minimum-gas-prices = "25000000award"|g' $HOME/.warden/config/app.toml sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.warden/config/config.toml sed -i -e "s/^indexer _=._/indexer = "null"/" $HOME/.warden/config/config.toml

## Create service file

sudo tee /etc/systemd/system/wardend.service > /dev/null <\<EOF \[Unit] Description=Warden node After=network-online.target

\[Service] User=$USER WorkingDirectory=$HOME/.warden ExecStart=$(which wardend) start --home $HOME/.warden Restart=on-failure RestartSec=5 LimitNOFILE=65535

\[Install] WantedBy=multi-user.target EOF

## Reset and download snapshot

wardend tendermint unsafe-reset-all --home $HOME/.warden if curl -s --head curl https://server-4.itrocket.net/testnet/warden/warden\_2025-02-28\_1897600\_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then curl https://server-4.itrocket.net/testnet/warden/warden\_2025-02-28\_1897600\_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.warden else echo "No snapshot found" fi

## Enable and start service

sudo systemctl daemon-reload sudo systemctl enable wardend sudo systemctl restart wardend && sudo journalctl -u wardend -fo cat

## Create wallet

wardend keys add $WALLET

## To restore existing wallet

wardend keys add $WALLET --recover

## Save wallet and validator address

WALLET\_ADDRESS=$(wardend keys show $WALLET -a) VALOPER\_ADDRESS=$(wardend keys show $WALLET --bech val -a) echo "export WALLET\_ADDRESS=$WALLET\_ADDRESS" >> $HOME/.bash\_profile echo "export VALOPER\_ADDRESS=$VALOPER\_ADDRESS" >> $HOME/.bash\_profile source $HOME/.bash\_profile

## Check sync status

wardend status 2>&1 | jq

## Check balance

wardend query bank balances $WALLET\_ADDRESS

## Create validator JSON file

echo "{"pubkey":{"@type":"/cosmos.crypto.ed25519.PubKey","key":"$(wardend comet show-validator | grep -Po '"key":\s\*"\K\[^"]\*')"}, "amount": "1000000award", "moniker": "test", "identity": "", "website": "", "security": "", "details": "I love blockchain ❤️", "commission-rate": "0.1", "commission-max-rate": "0.2", "commission-max-change-rate": "0.01", "min-self-delegation": "1"}" > validator.json

## Create validator using JSON

wardend tx staking create-validator validator.json\
\--from $WALLET\
\--chain-id chiado\_10010-1\
\--gas auto --gas-adjustment 1.6 --fees 250000000000000award

## Security setup

sudo ufw default allow outgoing sudo ufw default deny incoming sudo ufw allow ssh/tcp sudo ufw allow ${WARDEN\_PORT}656/tcp sudo ufw enable

## Delete node

sudo systemctl stop wardend sudo systemctl disable wardend sudo rm -rf /etc/systemd/system/wardend.service sudo rm $(which wardend) sudo rm -rf $HOME/.warden sed -i "/WARDEN\_/d" $HOME/.bash\_profile

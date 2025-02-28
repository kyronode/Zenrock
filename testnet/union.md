---
description: Union node guides
icon: arrows-to-circle
---

# Union

## Union Testnet Validator Setup Guide

### Step 1: Set Up Validator Name

Replace `YOUR_MONIKER_GOES_HERE` with your desired validator name:

```sh
MONIKER="YOUR_MONIKER_GOES_HERE"
```

***

### Step 2: Install Dependencies

Update the system and install necessary build tools:

```sh
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential
sudo apt -qy upgrade
```

***

### Step 3: Install Go

Remove any existing Go installation and install the latest version:

```sh
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.23.5.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
```

Update environment variables:

```sh
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```

***

### Step 4: Download Binaries

Create the required directories and download the project binaries:

```sh
mkdir -p $HOME/.union/cosmovisor/genesis/bin
wget -O $HOME/.union/cosmovisor/genesis/bin/uniond https://snapshots.kjnodes.com/union-testnet/uniond-v0.25.0-linux-amd64
chmod +x $HOME/.union/cosmovisor/genesis/bin/uniond
```

Create application symlinks:

```sh
ln -s $HOME/.union/cosmovisor/genesis $HOME/.union/cosmovisor/current -f
sudo ln -s $HOME/.union/cosmovisor/current/bin/uniond /usr/local/bin/uniond -f
```

***

### Step 5: Install Cosmovisor and Create a Service

Install Cosmovisor:

```sh
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
```

Create a systemd service file:

```sh
sudo tee /etc/systemd/system/union-testnet.service > /dev/null << EOF
[Unit]
Description=union node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --home=$HOME/.union
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.union"
Environment="DAEMON_NAME=uniond"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.union/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```

Reload systemd and enable the service:

```sh
sudo systemctl daemon-reload
sudo systemctl enable union-testnet.service
```

***

### Step 6: Initialize the Node

Set up an alias:

```sh
alias uniond='uniond --home=$HOME/.union/'
```

Configure the node:

```sh
uniond config set client chain-id union-testnet-9
uniond config set client keyring-backend test
uniond config set client node tcp://localhost:17157
```

Initialize the node:

```sh
uniond init $MONIKER --chain-id union-testnet-9 --home=$HOME/.union
```

Download genesis and addrbook:

```sh
curl -Ls https://snapshots.kjnodes.com/union-testnet/genesis.json > $HOME/.union/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/union-testnet/addrbook.json > $HOME/.union/config/addrbook.json
```

Set seeds:

```sh
sed -i -e "s|^seeds *=.*|seeds = \"3f472746f46493309650e5a033076689996c8881@union-testnet.rpc.kjnodes.com:17159\"|" $HOME/.union/config/config.toml
```

Set minimum gas price:

```sh
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0muno\"|" $HOME/.union/config/app.toml
```

Optimize pruning settings:

```sh
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.union/config/app.toml
```

Set custom ports:

```sh
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:17158\"%";
sed -i -e "s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:17157\"%";
sed -i -e "s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:17156\"%";
sed -i -e "s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":17166\"%" $HOME/.union/config/config.toml
```

***

### Step 7: Download Latest Chain Snapshot

```sh
curl -L https://snapshots.kjnodes.com/union-testnet/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.union
[[ -f $HOME/.union/data/upgrade-info.json ]] && cp $HOME/.union/data/upgrade-info.json $HOME/.union/cosmovisor/genesis/upgrade-info.json
```

***

### Step 8: Start Service and Check Logs

Start the service:

```sh
sudo systemctl start union-testnet.service
```

Check logs:

```sh
sudo journalctl -u union-testnet.service -f --no-hostname -o cat
```

***

This guide provides a structured approach to setting up a **Union Testnet Validator** for seamless participation in the network.

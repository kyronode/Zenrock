---
description: Lumera Installation Guide
icon: arrows-to-circle
---

# Lumera

### Prerequisites

* A fresh Linux-based server (Ubuntu recommended)
* `curl`, `wget`, `tar`, and `systemd` installed
* Root or sudo privileges

***

### 1. Install Golang

```bash
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.23.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
rm go1.23.4.linux-amd64.tar.gz

echo 'export GOROOT=/usr/local/go' >> ~/.bash_profile
echo 'export GOPATH=$HOME/go' >> ~/.bash_profile
echo 'export GO111MODULE=on' >> ~/.bash_profile
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bash_profile
source ~/.bash_profile

#Check GO version 
go version
```

***

### 2. Download Lumera Binary

```bash
wget https://github.com/LumeraProtocol/lumera/releases/download/v0.4.1/lumera_v0.4.1_linux_amd64.tar.gz
tar -xvzf lumera_v0.4.1_linux_amd64.tar.gz
```

***

### 3. Set Up Cosmovisor

```bash
#Install Cosmovisor if you don't have
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@latest

#Set Cosmovisor directory
mkdir -p $HOME/.lumera/cosmovisor/genesis/bin
cp ./lumerad $HOME/.lumera/cosmovisor/genesis/bin/
chmod +x $HOME/.lumera/cosmovisor/genesis/bin/lumerad
sudo rm /usr/local/bin/lumerad
sudo ln -s $HOME/.lumera/cosmovisor/genesis/bin/lumerad /usr/local/bin/lumerad
```

***

### 4. Initialize the Node

```bash
lumerad init YOUR_MONIKER --chain-id lumera-testnet-1
```

***

### 5. Download Address Book

```bash
wget -O addrbook.json https://snapshots.polkachu.com/testnet-addrbook/lumera/addrbook.json --inet4-only
mv addrbook.json ~/.lumera/config/
```

***

### 6. Configure `config.toml` and `app.toml`

```bash
CONFIG_TOML="$HOME/.lumera/config/config.toml"
APP_TOML="$HOME/.lumera/config/app.toml"

# Set minimum gas price
sed -i 's/^minimum-gas-prices *=.*/minimum-gas-prices = "0.25ulume"/' "$APP_TOML"

# Set seeds
sed -i 's|^seeds *=.*|seeds = "20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:30756,10a50e7a88561b22a8d1f6f0fb0b8e54412229ab@seeds.lumera.io:26656,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:30756"|' "$CONFIG_TOML"

# Set persistent peers
sed -i 's|^persistent_peers *=.*|persistent_peers = "0f85ddad45882ba2f770de01308dafcc833abe81@94.130.23.254:30756,221d579ff9b35887952ac3b686f9b230da21e19e@157.90.67.237:26656,84d98f2821fa97952c8446d9afb8d959a1ea0069@49.12.87.14:30756,3e00e110111612ac9c3d93b6ba75796c82a7bd1d@141.94.141.165:26856,85c44e89370a614aa9df951ab348c937b2bf6094@184.107.110.139:59700,d7cc6ea2afbb0a96ffd5e6dbd7955739fe019cf4@65.109.126.23:14556,0df5af293889fa827d14dade600daa11f548fbd6@95.217.152.99:26656,4ce2b8dea40cbf867bde32c17a24b19b26c981a3@65.21.220.178:26656,7c37a7eb1292b432a8f98ba40d32cb3dd4b3beeb@164.132.247.253:56416,9ae8787e6519141369b8858f6c240336767c1f7c@95.217.76.186:26656,97bfa489b34bf5b125dc40667623b45350b7dba4@136.38.55.33:26656,7afeb06db4edc7e7a6c018909876808408c4d1d7@148.113.165.128:56416,93d774235e67d1ec5f613f284fb0a200a0822252@136.243.59.103:31656,4ee99f2039eee00535e2197f77cfd23f61803cc0@66.129.102.28:26656,8a24daaac44fad0bf9bcc27ee1f91b24a32a7f51@154.91.1.115:26656,dafc81d3a24a2a96f8da1f3ce7bd0b83601726ec@160.250.106.37:30756,c03fce64f173d50a6b98309658d43bb3fa09ae17@65.109.120.211:36656,e667a6d70f8a07e5d766b8e5cc7173b8e0a6274e@135.181.79.101:30656,ada9b1c31d2ac183ddcfcf7378ed408cd056391a@65.21.29.250:3690,4a92caaced8428358e1f84f5778126c21c038dc3@95.217.62.179:11456,49e22975a1d6c5204072f25eb71c01faf54b4b92@88.99.149.170:17656,7f0c7ae8eae8108cc8ed6dbc66efd16f9676bac9@3.218.250.158:26656,1edeedc5f5c3d08a1999a7c334e11ad10d252039@88.99.136.168:26656,4f3f98fa837890bf2386ec60bc6a64b62e0d36c5@96.230.25.243:26656,54bb17ce612313b941ec6aee00413d6a08032e69@64.203.83.66:26656,ee7b29ce2ef3c7d18a58dc96ab9e6516755bfa0f@65.108.131.104:30756,2fe5e18e23c24accacfa9fada0f2683b9721cb0c@95.217.77.229:30756,8d7557e2e4e8c5d3b409663764d9fdc22d2c3449@44.204.100.172:26656,cd26df61f5d469d574fd89e3c8a08a323187090f@18.191.254.213:26656,d1c934296c3e3f6379677a94a06e944e3dec17f3@94.130.143.184:17656,a2cac656019665fcd03d3039fee5940e77a035c4@37.27.239.10:26656,5437a5fd010fe7bc1ab078f343a8b423edf3e6a6@65.109.24.208:26656,5c2a752c9b1952dbed075c56c600c3a79b58c395@195.3.223.139:27616,c96e213f718967bd0a00f3d9f61485d28b02cba6@93.147.218.214:26656,5e8af106ab8273479eaf76c81ceef6fcd0a42555@152.53.110.139:63656,4902d5db03ad6754e287ffdd051a5507a595927a@3.236.181.141:26656,13fe76b868a6dc601b4603c4f3c0febe9b17008c@65.21.67.40:36656,020caf16885970199588fcfa44c49eedc5e97421@88.198.52.46:30756,9ed3e540ca5ff2d57a58cd9b62128a48f0fe01c3@65.109.59.22:30756"|' "$CONFIG_TOML"
```

***

### 7. Create Systemd Service

```bash
sudo tee /etc/systemd/system/lumerad.service > /dev/null <<EOF
[Unit]
Description=Lumera Node (Cosmovisor)
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.lumera"
Environment="DAEMON_NAME=lumerad"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reexec
sudo systemctl enable lumerad
sudo systemctl start lumerad
journalctl -u lumerad -f
```

***

### 8. (Optional) Reset and Restore from Snapshot

```bash
sudo systemctl stop lumerad
cp ~/.lumera/data/priv_validator_state.json ~/.lumera/priv_validator_state.json
lumerad tendermint unsafe-reset-all --home $HOME/.lumera --keep-addr-book
curl -o - -L https://snapshots.polkachu.com/testnet-snapshots/lumera/lumera_1243620.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.lumera
sudo systemctl start lumerad
```

***

9. Check Node Sync

```bash
#Paste this to your terminal
bash <(curl -s https://raw.githubusercontent.com/kyronode/Lumera/main/lumera-sync.sh)
```

***

### Useful Commands

* Check logs: `journalctl -u lumerad -f`
* Restart node: `sudo systemctl restart lumerad`
* Stop node: `sudo systemctl stop lumerad`
* Start node: `sudo systemctl start lumerad`
* Node info: `lumerad status` or `lumerad tendermint show-node-id`

***

### Credits

* [LumeraProtocol GitHub](https://github.com/LumeraProtocol)
* [Polkachu Snapshots](https://polkachu.com/testnets/lumera)

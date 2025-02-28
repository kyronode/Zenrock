---
description: Swisstronik node guides
icon: arrows-to-circle
---

# Swisstronik

#### **1. Instal Dependensi**

```bash
bashCopyEditsudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

#### **2. Instal Go**

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

#### **3. Set Variabel Lingkungan**

```bash
bashCopyEditecho 'export WALLET="wallet"' >> $HOME/.bash_profile
echo 'export MONIKER="test"' >> $HOME/.bash_profile
echo 'export SWISS_CHAIN_ID="swisstronik_1291-1"' >> $HOME/.bash_profile
echo 'export SWISS_PORT="44"' >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### **4. Unduh Binary Swisstronik**

```bash
bashCopyEditcd $HOME
wget https://github.com/SigmaGmbH/swisstronik-chain/releases/download/testnet-v1.0.7/swisstronikd.zip
rm -rf bin
unzip swisstronikd.zip
sudo cp ~/bin/libsgx_wrapper_v1.0.7.x86_64.so /usr/lib
cp bin/v1.0.7_enclave.signed.so $HOME/.swisstronik-enclave
sudo mv $HOME/bin/swisstronikd $HOME/go/bin
```

#### **5. Konfigurasi Node**

```bash
bashCopyEditswisstronikd config node tcp://localhost:${SWISS_PORT}657
swisstronikd config keyring-backend os
swisstronikd config chain-id swisstronik_1291-1
swisstronikd init "$MONIKER" --chain-id swisstronik_1291-1
```

#### **6. Unduh Genesis dan Addrbook**

```bash
bashCopyEditwget -O $HOME/.swisstronik/config/genesis.json https://server-1.itrocket.net/testnet/swisstronik/genesis.json
wget -O $HOME/.swisstronik/config/addrbook.json https://server-1.itrocket.net/testnet/swisstronik/addrbook.json
```

#### **7. Set Peers dan Seeds**

```bash
bashCopyEditSEEDS="ec00cbf5c72ad261e28fd8de61d09cc2936456ed@swisstronik-testnet-seed.itrocket.net:44656"
PEERS="f05c4343d2df801ba05a5ec7bd9954d8728fdb36@swisstronik-testnet-peer.itrocket.net:26656"

sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" \
       $HOME/.swisstronik/config/config.toml
```

#### **8. Atur Custom Port**

```bash
bashCopyEditsed -i.bak -e "s%:26657%:${SWISS_PORT}657%g" $HOME/.swisstronik/config/config.toml
```

#### **9. Konfigurasi Pruning, Minimum Gas Price, dan Prometheus**

```bash
bashCopyEditsed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "7uswtr"|g' $HOME/.swisstronik/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.swisstronik/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.swisstronik/config/config.toml
```

#### **10. Buat Service Systemd**

```bash
bashCopyEditsudo tee /etc/systemd/system/swisstronikd.service > /dev/null <<EOF
[Unit]
Description=Swisstronik node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.swisstronik
ExecStart=$(which swisstronikd) start --home $HOME/.swisstronik
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

#### **11. Reset dan Download Snapshot**

```bash
bashCopyEditswisstronikd tendermint unsafe-reset-all --home $HOME/.swisstronik
if curl -s --head curl https://server-1.itrocket.net/testnet/swisstronik/swisstronik_2025-02-28_10547729_snap.tar.lz4 | head -n 1 | grep "200" > /dev/null; then
  curl https://server-1.itrocket.net/testnet/swisstronik/swisstronik_2025-02-28_10547729_snap.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.swisstronik
else
  echo "No snapshot found"
fi
```

#### **12. Jalankan Node**

```bash
bashCopyEditsudo systemctl daemon-reload
sudo systemctl enable swisstronikd
sudo systemctl restart swisstronikd && sudo journalctl -u swisstronikd -fo cat
```

***

### **Pembuatan Wallet**

```bash
bashCopyEditswisstronikd keys add $WALLET
# atau untuk mengembalikan wallet lama
swisstronikd keys add $WALLET --recover
```

#### **Simpan Alamat Wallet**

```bash
bashCopyEditWALLET_ADDRESS=$(swisstronikd keys show $WALLET -a)
VALOPER_ADDRESS=$(swisstronikd keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### **Cek Status Sinkronisasi**

```bash
bashCopyEditswisstronikd status 2>&1 | jq .sync_info
```

***

### **Membuat Validator**

Pastikan wallet sudah memiliki saldo sebelum membuat validator.

```bash
bashCopyEditswisstronikd tx staking create-validator \
  --amount 1000000uswtr \
  --from $WALLET \
  --commission-rate 0.1 \
  --commission-max-rate 0.2 \
  --commission-max-change-rate 0.01 \
  --min-self-delegation 1 \
  --pubkey $(swisstronikd tendermint show-validator) \
  --moniker "test" \
  --identity "" \
  --website "" \
  --details "I love blockchain ❤️" \
  --chain-id swisstronik_1291-1 \
  --gas-adjustment 1.4 --gas auto --gas-prices 7uswtr \
  -y
```

***

### **Monitoring Node**

Buat skrip untuk memantau status sinkronisasi:

```bash
bashCopyEdit#!/bin/bash
rpc_port=$(grep -m 1 -oP '^laddr = "\K[^"]+' "$HOME/.swisstronik/config/config.toml" | cut -d ':' -f 3)
while true; do
  local_height=$(curl -s localhost:$rpc_port/status | jq -r '.result.sync_info.latest_block_height')
  network_height=$(curl -s https://swisstronik-testnet-rpc.itrocket.net/status | jq -r '.result.sync_info.latest_block_height')

  if ! [[ "$local_height" =~ ^[0-9]+$ ]] || ! [[ "$network_height" =~ ^[0-9]+$ ]]; then
    echo "Error: Invalid block height data. Retrying..."
    sleep 5
    continue
  fi

  blocks_left=$((network_height - local_height))
  echo "Node Height: $local_height | Network Height: $network_height | Blocks Left: $blocks_left"
  sleep 5
done
```

***

### **Hapus Node**

```bash
bashCopyEditsudo systemctl stop swisstronikd
sudo systemctl disable swisstronikd
sudo rm -rf /etc/systemd/system/swisstronikd.service
sudo rm $(which swisstronikd)
sudo rm -rf $HOME/.swisstronik
sed -i "/SWISS_/d" $HOME/.bash_profile
```

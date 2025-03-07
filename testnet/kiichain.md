---
description: KiiChain node guide documentation
icon: arrows-to-circle
---

# KiiChain

## Kiichain Validator Node Setup Guide with Cosmovisor

This guide walks you through the process of setting up a Kiichain validator node on Testnet Oro using cosmovisor for seamless chain upgrades.

***

### Requirements

* **Golang:** v1.21.x or v1.22.x (Golang v1.23.x or higher will cause compilation errors)
* **Build Tools:** `build-essential` package (for Linux)

***

### 1. Install Kiichain CLI

Clone the repository and build the binary:

```bash
bashCopyEditgit clone https://github.com/KiiChain/kiichain.git
cd kiichain
make install
```

Verify the installation:

```bash
bashCopyEditkiichaind version
```

***

### 2. Join the Testnet

You have two options to bootstrap your node:

#### a. Standard Full Node

```bash
bashCopyEditcurl -O https://raw.githubusercontent.com/KiiChain/testnets/refs/heads/main/testnet_oro/join_oro.sh
chmod +x join_oro.sh
./join_oro.sh
```

#### b. Node with Cosmovisor (Recommended)

```bash
bashCopyEditcurl -O https://raw.githubusercontent.com/KiiChain/testnets/refs/heads/main/testnet_oro/join_oro_cv.sh
chmod +x join_oro_cv.sh
./join_oro_cv.sh
```

***

### 3. Initial Node Configuration

#### a. Backup and Clean Existing Configurations

```bash
bashCopyEdit# Backup old configuration (if any)
cp -r $HOME/.kiichain3 $HOME/.kiichain3-bk
# Remove old configuration
rm -r $HOME/.kiichain3
```

#### b. Set Environment Variables

```bash
bashCopyEditPERSISTENT_PEERS="5b6aa55124c0fd28e47d7da091a69973964a9fe1@uno.sentry.testnet.v3.kiivalidator.com:26656,5e6b283c8879e8d1b0866bda20949f9886aff967@dos.sentry.testnet.v3.kiivalidator.com:26656"
CHAIN_ID=kiichain3
NODE_HOME=$HOME/.kiichain3
NODE_MONIKER=testnet_oro
GENESIS_URL=https://raw.githubusercontent.com/KiiChain/testnets/refs/heads/main/testnet_oro/genesis.json
```

#### c. Initialize the Node

```bash
bashCopyEditkiichaind init $NODE_MONIKER --chain-id $CHAIN_ID --home $NODE_HOME
```

#### d. Set Persistent Peers

```bash
bashCopyEditsed -i -e "/persistent-peers =/ s^= .*^= \"$PERSISTENT_PEERS\"^" $NODE_HOME/config/config.toml
```

#### e. Enable Database Features and Increase Concurrency

```bash
bashCopyEditsed -i.bak -e "s|^occ-enabled *=.*|occ-enabled = true|" $NODE_HOME/config/app.toml
sed -i.bak -e "s|^sc-enable *=.*|sc-enable = true|" $NODE_HOME/config/app.toml
sed -i.bak -e "s|^ss-enable *=.*|ss-enable = true|" $NODE_HOME/config/app.toml
sed -i.bak -e 's/^# concurrency-workers = 20$/concurrency-workers = 500/' $NODE_HOME/config/app.toml
```

#### f. Download and Set the Genesis File

```bash
bashCopyEditwget $GENESIS_URL -O genesis.json
mv genesis.json $NODE_HOME/config/genesis.json
```

(Optional) Verify the genesis file's SHA256 checksum:

```bash
bashCopyEditsha256sum $NODE_HOME/config/genesis.json
```

_Expected SHA256:_ `e22442f19149db7658bcf777d086b52b38d834ea17010c313cd8aece137b647a`

***

### 4. Configure as Validator Node

Change the node mode from full node to validator:

```bash
bashCopyEditsed -i 's/mode = "full"/mode = "validator"/g' $NODE_HOME/config/config.toml
```

***

### 5. (Optional) Setup State Sync

State sync speeds up synchronization by downloading recent state data.

#### a. Determine Sync Block Height

```bash
bashCopyEditTRUST_HEIGHT_DELTA=500
LATEST_HEIGHT=$(curl -s https://rpc.uno.sentry.testnet.v3.kiivalidator.com/block | jq -r ".block.header.height")
if [[ "$LATEST_HEIGHT" -gt "$TRUST_HEIGHT_DELTA" ]]; then
  SYNC_BLOCK_HEIGHT=$(($LATEST_HEIGHT - $TRUST_HEIGHT_DELTA))
else
  SYNC_BLOCK_HEIGHT=$LATEST_HEIGHT
fi
```

#### b. Get the Sync Block Hash

```bash
bashCopyEditSYNC_BLOCK_HASH=$(curl -s "https://rpc.uno.sentry.testnet.v3.kiivalidator.com/block?height=$SYNC_BLOCK_HEIGHT" | jq -r ".block_id.hash")
```

#### c. Update the `config.toml` for State Sync

```bash
bashCopyEditsed -i.bak -e "s|^enable *=.*|enable = true|" $NODE_HOME/config/config.toml
sed -i.bak -e "s|^rpc-servers *=.*|rpc-servers = \"https://rpc.uno.sentry.testnet.v3.kiivalidator.com,https://rpc.dos.sentry.testnet.v3.kiivalidator.com\"|" $NODE_HOME/config/config.toml
sed -i.bak -e "s|^db-sync-enable *=.*|db-sync-enable = false|" $NODE_HOME/config/config.toml
sed -i.bak -e "s|^trust-height *=.*|trust-height = $SYNC_BLOCK_HEIGHT|" $NODE_HOME/config/config.toml
sed -i.bak -e "s|^trust-hash *=.*|trust-hash = \"$SYNC_BLOCK_HASH\"|" $NODE_HOME/config/config.toml
```

***

### 6. Start the Node

Run your Kiichain node:

```bash
bashCopyEditkiichaind start --home $NODE_HOME
```

***

### 7. Create and Register Your Validator

#### a. Create a New Key

Generate a key for transactions:

```bash
bashCopyEditkiichaind keys add $VALIDATOR_KEY_NAME
```

_Keep the mnemonic safe—it is required for account recovery._

#### b. Get Your Validator Public Key

```bash
bashCopyEditkiichaind tendermint show-validator
```

#### c. Create the Validator

Replace `<your-moniker>` and `$VALIDATOR_KEY_NAME` with your values:

```bash
bashCopyEditCHAIN_ID=kiichain3
MONIKER=<your-moniker>
AMOUNT=1000000000ukii   # 1000 kii for self-delegation
COMMISSION_MAX_CHANGE_RATE=0.1
COMMISSION_MAX_RATE=0.1
COMMISSION_RATE=0.1
MIN_SELF_DELEGATION_AMOUNT=1000000000

kiichaind tx staking create-validator \
  --amount=$AMOUNT \
  --pubkey=$(kiichaind tendermint show-validator) \
  --moniker=$MONIKER \
  --chain-id=$CHAIN_ID \
  --commission-rate=$COMMISSION_RATE \
  --commission-max-rate=$COMMISSION_MAX_RATE \
  --commission-max-change-rate=$COMMISSION_MAX_CHANGE_RATE \
  --min-self-delegation=$MIN_SELF_DELEGATION_AMOUNT \
  --gas="auto" \
  --gas-adjustment 1.3 \
  --gas-prices="0.01ukii" \
  --from=$VALIDATOR_KEY_NAME
```

This transaction must be sent from the machine running your node.

***

### 8. (Optional) Configure as an Archival Node

If you wish to save all historical states, update the pruning setting in `$NODE_HOME/config/config.toml`:

```toml
tomlCopyEditpruning = "nothing"
```

Other pruning options include `default`, `everything`, and `custom`.

***

### 9. Cosmovisor Upgrade Management

Cosmovisor is used to manage automatic chain upgrades. The cosmovisor bootstrap script (`join_oro_cv.sh`) already sets up your node for automated upgrades.

#### Adding a New Upgrade

1. **Compile the new binary** (ensure it is built on the correct upgrade tag, e.g., v1.0.1 or v2.0.0).
2.  **Verify the binary version:**

    ```bash
    bashCopyEditkiichaind version
    ```
3.  **Add the upgrade:**

    ```bash
    bashCopyEditcosmovisor add-upgrade <upgrade-name> <path-to-binary>
    ```

    * `<upgrade-name>`: The on-chain upgrade name.
    * `<path-to-binary>`: Full path to the new binary (e.g., `/home/ubuntu/kiichain/build/kiichaind`).

***

### Final Notes

* **System Requirements:** Recommended specs are 16 vCPU, 64 GB RAM, and 1 TB NVME SSD for optimal performance.
* **Monitoring:** Regularly monitor your node and review logs to troubleshoot issues.
* **Documentation:** For additional details, consult the official Kiichain and Cosmos validator documentation.

***



Happy validating!

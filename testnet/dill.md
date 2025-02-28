---
description: Dill node guides
---

# DILL

## Dill Node Setup Guide

### Node Requirements

| Node Type       | CPU     | Memory | Disk  | Bandwidth | OS Type                 |
| --------------- | ------- | ------ | ----- | --------- | ----------------------- |
| Light Validator | 2 Cores | 2GB    | 20GB  | 8Mb/s     | Ubuntu LTS 20.04+/MacOS |
| Full Validator  | 4 Cores | 8GB    | 256GB | 64Mb/s    | Ubuntu LTS 20.04+/MacOS |

***

### Connect to Alps Testnet

Use the following information to connect to the **Alps Testnet**:

* **RPC URL**: [https://rpc-alps.dill.xyz](https://rpc-alps.dill.xyz/)
* **Chain ID**: `102125`
* **Currency Symbol**: `DILL`
* **Explorer**: [https://alps.dill.xyz](https://alps.dill.xyz/)

For a guide on adding a custom network to MetaMask, please refer to [this link](https://chatgpt.com/c/67c1a5c3-f894-800a-b472-0341ebe590a2).

***

### Get the Test Token

To receive test tokens, please **join our Discord server**.

***

### Setting Up Your Dill Node

Run the following command to start your **Light** or **Full Node**:

```sh
curl -sO https://raw.githubusercontent.com/DillLabs/launch-dill-node/main/dill.sh  && chmod +x dill.sh && ./dill.sh
```

***

### Staking

You can stake your tokens and become a validator by visiting: [https://staking.dill.xyz/](https://staking.dill.xyz/)

#### Upload Your Deposit Info

* Paste the contents of your `deposit_data-xxxx.json` file into the input box.
* Click the **"Continue"** button to proceed with staking.

#### Connect to Your Wallet

* If the wallet address you are using is the same as your withdrawal address, simply check the confirmation box.
* Otherwise, you must manually enter your withdrawal address.

#### Send Deposit Transaction

Use **MetaMask** to send your deposit transaction.

***

### Validator Info

You can find the validator list here: [https://alps.dill.xyz/validators](https://alps.dill.xyz/validators)

After completing the staking process, search for your validator information on the page using your validator **public key**. It may take **30 minutes to 1 hour** to appear.

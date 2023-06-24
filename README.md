<p style="font-size:14px" align="right">
<a href="https://kjnodes.com/" target="_blank">Visit our website <img src="https://user-images.githubusercontent.com/50621007/168689709-7e537ca6-b6b8-4adc-9bd0-186ea4ea4aed.png" width="30"/></a>
<a href="https://discord.gg/EY35ZzXY" target="_blank">Join our discord <img src="https://user-images.githubusercontent.com/50621007/176236430-53b0f4de-41ff-41f7-92a1-4233890a90c8.png" width="30"/></a>
<a href="https://kjnodes.com/" target="_blank">Visit our website <img src="https://user-images.githubusercontent.com/50621007/168689709-7e537ca6-b6b8-4adc-9bd0-186ea4ea4aed.png" width="30"/></a>
</p>

<p style="font-size:14px" align="right">
<a href="https://hetzner.cloud/?ref=y8pQKS2nNy7i" target="_blank">Deploy your VPS using our referral link to get 20€ bonus <img src="https://user-images.githubusercontent.com/50621007/174612278-11716b2a-d662-487e-8085-3686278dd869.png" width="30"/></a>
</p>

# Teritori node setup for Testnet — teritori-testnet-v2

Official documentation:

> -   [Validator setup instructions](https://github.com/TERITORI/teritori-chain/tree/main/testnet)

Explorer:

> -   https://teritori.explorers.guru/

## Hardware Requirements

Like any Cosmos-SDK chain, the hardware requirements are pretty modest.

### Minimum Hardware Requirements

-   2x CPUs; the faster clock speed the better
-   2GB RAM
-   80GB Disk
-   Ubuntu 18.04 LTS
-   Permanent Internet connection (traffic will be minimal during testnet; 10Mbps will be plenty - for production at least 100Mbps is expected)

Notes on the configurations.

-   Multicore is important, regardless the less CPU time usage
-   teritorid uses less than 1GB memory and 2GB should be enough for now. Once your new server is running, login to the server and upgrade your packages.

## Set up your teritori fullnode

### Option 1 (automatic)

You can setup your teritori fullnode in few minutes by using automated script below. It will prompt you to input your validator node name!

```
wget -O teritori.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/teritori/teritori.sh && chmod +x teritori.sh && ./teritori.sh
```

### Option 2 (manual)

You can follow [manual guide](https://github.com/kj89/testnet_manuals/blob/main/teritori/manual_install.md) if you better prefer setting up node manually

## Post installation

When installation is finished please load variables into system

```
source $HOME/.bash_profile
```

Next you have to make sure your validator is syncing blocks. You can use command below to check synchronization status

```
teritorid status 2>&1 | jq .SyncInfo
```

### Create wallet

To create new wallet you can use command below. Don’t forget to save the mnemonic

```
teritorid keys add $WALLET
```

(OPTIONAL) To recover your wallet using seed phrase

```
teritorid keys add $WALLET --recover
```

To get current list of wallets

```
teritorid keys list
```

### Save wallet info

Add wallet and valoper address and load variables into the system

```
TERITORI_WALLET_ADDRESS=$(teritorid keys show $WALLET -a)
TERITORI_VALOPER_ADDRESS=$(teritorid keys show $WALLET --bech val -a)
echo 'export TERITORI_WALLET_ADDRESS='${TERITORI_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export TERITORI_VALOPER_ADDRESS='${TERITORI_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Fund your wallet

In order to create validator first you need to fund your wallet with testnet tokens.
To top up your wallet join Teritori discord server to request fund on the Faucet channel.

-   **#faucet** for tokens

To request a faucet grant:

```
$request <YOUR_WALLET_ADDRESS>
```

### Create validator

Before creating validator please make sure that you have at least 1 qck (1 qck is equal to 1000000 uqck) and your node is synchronized

To check your wallet balance:

```
teritorid query bank balances $TERITORI_WALLET_ADDRESS --chain-id $TERITORI_CHAIN_ID
```

> If your wallet does not show any balance than probably your node is still syncing. Please wait until it finish to synchronize and then continue

To create your validator run command below

```
teritorid tx staking create-validator \
  --amount 1000000utori \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.05" \
  --min-self-delegation=1000000 \
  --pubkey $(teritorid tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $TERITORI_CHAIN_ID
```

## Security

To protect you keys please make sure you follow basic security rules

### Set up ssh keys for authentication

Good tutorial on how to set up ssh keys for authentication to your server can be found [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04)

### Basic Firewall security

Start by checking the status of ufw.

```
sudo ufw status
```

Sets the default to allow outgoing connections, deny all incoming except ssh and 26656. Limit SSH login attempts

```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow ${TERITORI_PORT}656,${TERITORI_PORT}660/tcp
sudo ufw enable
```

## Monitoring

To monitor and get alerted about your validator health status you can use my guide on [Set up monitoring and alerting for teritori validator](https://github.com/kj89/testnet_manuals/blob/main/teritori/monitoring/README.md)

## Calculate synchronization time

This script will help you to estimate how much time it will take to fully synchronize your node\
It measures average blocks per minute that are being synchronized for period of 5 minutes and then gives you results

```
wget -O synctime.py https://raw.githubusercontent.com/kj89/testnet_manuals/main/teritori/tools/synctime.py && python3 ./synctime.py
```

### Get list of validators

```
teritorid q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Get currently connected peer list with ids

```
curl -sS http://localhost:${TERITORI_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

## Usefull commands

### Service management

Check logs

```
journalctl -fu teritorid -o cat
```

Start service

```
sudo systemctl start teritorid
```

Stop service

```
sudo systemctl stop teritorid
```

Restart service

```
sudo systemctl restart teritorid
```

### Node info

Synchronization info

```
teritorid status 2>&1 | jq .SyncInfo
```

Validator info

```
teritorid status 2>&1 | jq .ValidatorInfo
```

Node info

```
teritorid status 2>&1 | jq .NodeInfo
```

Show node id

```
teritorid tendermint show-node-id
```

### Wallet operations

List of wallets

```
teritorid keys list
```

Recover wallet

```
teritorid keys add $WALLET --recover
```

Delete wallet

```
teritorid keys delete $WALLET
```

Get wallet balance

```
teritorid query bank balances $TERITORI_WALLET_ADDRESS
```

Transfer funds

```
teritorid tx bank send $TERITORI_WALLET_ADDRESS <TO_TERITORI_WALLET_ADDRESS> 1000000utori
```

### Voting

```
teritorid tx gov vote 1 yes --from $WALLET --chain-id=$TERITORI_CHAIN_ID
```

### Staking, Delegation and Rewards

Delegate stake

```
teritorid tx staking delegate $TERITORI_VALOPER_ADDRESS 10000000utori --from=$WALLET --chain-id=$TERITORI_CHAIN_ID --gas=auto
```

Redelegate stake from validator to another validator

```
teritorid tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000utori --from=$WALLET --chain-id=$TERITORI_CHAIN_ID --gas=auto
```

Withdraw all rewards

```
teritorid tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$TERITORI_CHAIN_ID --gas=auto
```

Withdraw rewards with commision

```
teritorid tx distribution withdraw-rewards $TERITORI_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$TERITORI_CHAIN_ID
```

### Validator management

Edit validator

```
teritorid tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$TERITORI_CHAIN_ID \
  --from=$WALLET
```

Unjail validator

```
teritorid tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$TERITORI_CHAIN_ID \
  --gas=auto
```

### Delete node

This commands will completely remove node from server. Use at your own risk!

```
sudo systemctl stop teritorid
sudo systemctl disable teritorid
sudo rm /etc/systemd/system/teritori* -rf
sudo rm $(which teritorid) -rf
sudo rm $HOME/.teritori* -rf
sudo rm $HOME/teritori -rf
```

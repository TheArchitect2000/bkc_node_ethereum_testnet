# How to build Cardano Relay Server



## 1. Prepare cardano-node and make it executable.
### Prepare a machine with the Ubuntu operating system.



### Update and install packages.
```
sudo apt-get update -y
sudo apt-get -y install build-essential pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make g++ tmux git jq wget libncursesw5
```



### Download and unpack the Hydra binaries of `cardano-node`.
```
cd ~
mkdir cardano-node
cd cardano-node
wget https://hydra.iohk.io/build/12997298/download/1/cardano-node-1.34.0-linux.tar.gz \
tar xvzf cardano-node-1.34.0-linux.tar.gz \
rm -rf cardano-node-1.34.0-linux.tar.gz
```



### Add `cardano-node` and `cardano-cli` to environmetal variables.
#### Create `~/.local/bin` directory then copy/move `cardano-node` and `cardano-cli` to it.
```
mkdir -p ~/.local/bin
cp -p ~/cardano-node/cardano-node ~/.local/bin
cp -p ~/cardano-cli/cardano-node ~/.local/bin
```
*To Move files, replace `cp -p` with `mv`*



#### Add `PATH` to last line of `.bashrc` to make cardano-node files executable.
```
cd ~
nano .bashrc
```
```
export PATH="~/.local/bin:$PATH"
```
*Use `Ctrl + o` and then press `Enter` to save and `Ctrl + x` to exit the file*



#### Enable `PATH` in current terminal.
```
cd ~
source .bashrc
```



## 2. Download and setup config files for relay node.
### Download `genesis`, `configuration` and `topology` files.
```
cd ~
mkdir relay
cd relay
wget https://hydra.iohk.io/build/8111119/download/1/testnet-config.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-byron-genesis.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-shelley-genesis.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-alonzo-genesis.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-topology.json
```



### Edite `testnet-topology.json`
```
nano testnet-topology.json
```
Replace `<block generator IP>` with block generator server IP Address.
```
{
  "Producers": [
    {
      "addr": "<block generator IP>",
      "port": 3001,
      "valency": 2
    },
    {
      "addr": "relays-new.cardano-testnet.iohkdev.io",
      "port": 3001,
      "valency": 2
    }
  ]
}
```
*Use `Ctrl + o` and then press `Enter` to save and `Ctrl + x` to exit the file*



## 3. Start Cardano Relay Node
Replace `<server IP address>` with IP Address of server that is runing on it.
```
cardano-node run +RTS -N -A16m -qg -qb -RTS \
--topology relay/testnet-topology.json \
--database-path relay/db \
--socket-path relay/db/node.socket \
--host-addr <server IP address> \
--port 3001 \
--config relay/testnet-config.json
```






# How to build Cardano Block Generator Server



## 1. Prepare cardano-node and make it executable.
### Prepare a machine with the Ubuntu operating system.



### Update and install packages.
```
sudo apt-get update -y
sudo apt-get -y install build-essential pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make g++ tmux git jq wget libncursesw5
```



### Download and unpack the Hydra binaries of `cardano-node`.
```
cd ~
mkdir cardano-node
cd cardano-node
wget https://hydra.iohk.io/build/12997298/download/1/cardano-node-1.34.0-linux.tar.gz \
tar xvzf cardano-node-1.34.0-linux.tar.gz \
rm -rf cardano-node-1.34.0-linux.tar.gz
```



### Add `cardano-node` and `cardano-cli` to environmetal variables.
#### Create `~/.local/bin` directory then copy/move `cardano-node` and `cardano-cli` to it.
```
mkdir -p ~/.local/bin
cp -p ~/cardano-node/cardano-node ~/.local/bin
cp -p ~/cardano-cli/cardano-node ~/.local/bin
```
*To Move files, replace `cp -p` with `mv`*



#### Add `PATH` to last line of `.bashrc` to make cardano-node files executable.
```
cd ~
nano .bashrc
```
```
export PATH="~/.local/bin:$PATH"
```
*Use `Ctrl + o` and then press `Enter` to save and `Ctrl + x` to exit the file*



#### Enable `PATH` in current terminal.
```
cd ~
source .bashrc
```



## 2. Download and setup config files for pool node.
### Download `genesis`, `configuration` and `topology` files.
```
cd ~
mkdir pool
cd pool
wget https://hydra.iohk.io/build/8111119/download/1/testnet-config.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-byron-genesis.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-shelley-genesis.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-alonzo-genesis.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-topology.json
```



### Edite `testnet-topology.json`
```
nano testnet-topology.json
```
Replace `<relay IP>` with Relay server IP Address.
```
{
  "Producers": [
    {
      "addr": "<relay IP>",
      "port": 3001,
      "valency": 2
    }
  ]
}
```
*Use `Ctrl + o` and then press `Enter` to save and `Ctrl + x` to exit the file*



## 3. Creating *Key Pairs*, *Addresses* and *Certifications*
> **Warning**
> ### Basic core node firewall configuration:
> - Make sure you can only login with SSH Keys, not password.
> - Make sure you cannot login as root
> - Make sure to setup SSH connections in a port different than the default 22
> - Make sure to configure the firewall to only allow connections from your relay nodes by setting up their ip addresses.
> ### Basic relay node firewall configuration:
> - Make sure you can only login with SSH Keys, not password.
> - Make sure you cannot login as root
> - Make sure to setup SSH connections in a port different than the default 22.
> - Make sure you only have the strictly necessary ports opened.
> See Firewall Additional Materials from [Amazon](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) and [Ubuntu](http://manpages.ubuntu.com/manpages/focal/man8/ufw.8.html)



### Create a new directory to store files
```
cd ~
mkdir keys
cd keys
```



#### Generating Payment and Stake key pairs
> **Warning**
> 🔥 **Critical Operational Security Advice:** Payment and Stake keys must be generated and used to build transactions in an cold environment. In other words, an air-gapped offline machine. Copy cardano-cli binary over to the offline machine and run the CLI method or mnemonic method. The only steps performed online in a hot environment are those steps that require live data. Namely the follow type of steps:
> - querying the current slot tip
> - querying the balance of an address
> - submitting a transaction



> **Note**
> Payment keys are used to send and receive payments.
> Stake keys are used to manage stake delegations.

```
###
### On air-gapped offline machine,
###
cardano-cli address key-gen \
--verification-key-file payment.vkey \
--signing-key-file payment.skey
```

```
###
### On air-gapped offline machine,
###
cardano-cli stake-address key-gen \
--verification-key-file stake.vkey \
--signing-key-file stake.skey
```



#### Generating Payment and Stake addresses
```
cardano-cli address build \
--payment-verification-key-file payment.vkey \
--out-file payment.addr \
--testnet-magic 1097911063
```
Run the following to find payment address.
```
cat payment.addr
```

> **Note**
> Stake address CAN'T receive payments but will receive the rewards from participating in the protocol.

```
cardano-cli stake-address build \
--stake-verification-key-file stake.vkey \
--out-file stake.addr \
--testnet-magic 1097911063
```
Run the following to find stake address.
```
cat stake.addr
```



#### Regenerate payment address
Now that we have a stake address, it is time to regenerate a payment address. This time we use both the stake verification key and payment verification key to build the address. With this, both addresses will be linked together and associated with one another.
```
cardano-cli address build \
--payment-verification-key-file payment.vkey \
--stake-verification-key-file stake.vkey \
--out-file paymentwithstake.addr \
--testnet-magic 1097911063
```



### Generate stake pool keys
> **Warning**
> 🔥 **Cold keys must be generated and stored on air-gapped offline machine.**
> Make sure you are not online until you have put your **cold keys** in a secure storage.

The block-producer node requires 3 keys as defined in the [Shelley ledger specs](https://hydra.iohk.io/build/2473732/download/1/ledger-spec.pdf):
- stake pool cold key (node.cert)
- stake pool hot key (kes.skey)
- stake pool VRF key (vrf.skey)

#### Generate Cold Keys and a Cold_counter
```
cardano-cli node key-gen \
--cold-verification-key-file cold.vkey \
--cold-signing-key-file cold.skey \
--operational-certificate-issue-counter-file cold.counter
```



#### Generate VRF Key pair
```
cardano-cli node key-gen-VRF \
--verification-key-file vrf.vkey \
--signing-key-file vrf.skey
```



#### Generate the KES Key pair
```
cardano-cli node key-gen-KES \
--verification-key-file kes.vkey \
--signing-key-file kes.skey
```



### Generate the Operational Certificate
> **Note**
> Wait for the relay node to sync before continuing.
> Folowing commands (**Obtaining KESPeriod** and **Obtaining current slotNo**) must executed in the **Relay server terminal**.
> If you get `cardano-cli: Network.Socket.connect: : does not exist (No such file or directory)` error message after sync, set `CARDANO_NODE_SOCKET_PATH` using command below

```
export CARDANO_NODE_SOCKET_PATH=~/relay/db/node.socket
```



#### Obtaining KESPeriod
```
cat ~/pool/testnet-shelley-genesis.json | grep KESPeriod
```
Output will be similar to `> "slotsPerKESPeriod": 3600,`



#### Obtaining current slotNo
```
cardano-cli query tip --testnet-magic 1097911063
```
Output will be similar to `{
    "blockNo": 27470,
    "headerHash": "bd954e753c1131a6cb7ab3a737ca7f78e2477bea93db54511cedefe8899ebed0",
    "slotNo": 656260
}`



#### Calculating the KES Period
Replace *slotNo* and *slotsPerKESPeriod*
```
expr slotNo / slotsPerKESPeriod
```
`> 182`
> **Note**
> `slotNo` and `Kes-period` will be different after times. Make sure to calculate them right before use.



#### Generate the Operational Certificate file
> **Note**
> Must be generated on the air-gapped offline machine.
> Using a cold storage like a USB flash drive, move `node.cert` to **Block Generator Server** `~/keys`.
```
cardano-cli node issue-op-cert \
--kes-verification-key-file kes.vkey \
--cold-signing-key-file cold.skey \
--operational-certificate-issue-counter cold.counter \
--kes-period 182 \
--out-file node.cert
```



## 4. Start Cardano Block Generator Node
Replace `<server IP address>` with IP Address of server that is runing on it.
```
cardano-node run +RTS -N -A16m -qg -qb -RTS \
--topology pool/testnet-topology.json \
--database-path pool/db \
--socket-path pool/db/node.socket \
--host-addr <server IP address> \
--port 3001 \
--config pool/testnet-config.json \
--shelley-kes-key keys/kes.skey \
--shelley-vrf-key keys/vrf.skey \
--shelley-operational-certificate keys/node.cert
```



#### Creating `Protocol Parameters` file
> **Note**
> Wait for the block-producing node to start syncing before continuing if you get this error message: `cardano-cli: Network.Socket.connect: : does not exist (No such file or directory)`
> If you get `cardano-cli: Network.Socket.connect: : does not exist (No such file or directory)` error message after sync, set `CARDANO_NODE_SOCKET_PATH` using command below

```
export CARDANO_NODE_SOCKET_PATH=~/pool/db/node.socket
```
To Generate `Protocol Parameters` file:
```
 cardano-cli query protocol-parameters \
 --testnet-magic 1097911063 \
 --out-file protocol.json
```



## 5. Register stake address in the blockchain
we Created our stake keys and stake address, which allow us to participate in the protocol by delegating our stake or by creating a stake pool.
We need to register our stake key in the blockchain.



### Create a registration certificate
```
cardano-cli stake-address registration-certificate \
--stake-verification-key-file stake.vkey \
--out-file stake.cert
```
Once the certificate has been created, we must include it in a transaction to post it to the blockchain.



### Draft transaction
#### Query the UTXO
To finde TxHash of address
```
cardano-cli query utxo \
--address $(cat payment.addr) \
--testnet-magic 1097911063
```
Output 

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
c738a7a07f6aafd3dee874d0a883d752db71fb3bc2c14616ea2d34256e845053     0        1000000000 lovelace + TxOutDatumNone
```


#### Draft the transaction
```
cardano-cli transaction build-raw \
--tx-in c738a7a07f6aafd3dee874d0a883d752db71fb3bc2c14616ea2d34256e845053#0 \
--tx-out $(cat paymentwithstake.addr)+0 \
--ttl 0 \
--fee 0 \
--out-file tx.raw \
--certificate-file stake.cert
```
Replace `c738a7a07f6aafd3dee874d0a883d752db71fb3bc2c14616ea2d34256e845053` in `--tx-in` with **TxHash** and **TxIx** with number in `--tx-in #number`.



#### Calculate fees and deposit
This transaction has only 1 input (the UTXO used to pay the transaction fee) and 1 output (our `paymentwithstake.addr` to which we are sending the change).
```
cardano-cli transaction calculate-min-fee \
--tx-body-file tx.raw \
--tx-in-count 1 \
--tx-out-count 1 \
--witness-count 1 \
--byron-witness-count 0 \
--testnet-magic 1097911063 \
--protocol-params-file protocol.json
```
Output `> 171485 Lovelace`.



In this transaction we have to not only pay transaction fees, but also include a deposit (which we will get back when we deregister the key) as stated in the protocol parameters:
The deposit amount can be found in the `protocol.json` under `stakeAddressDeposit`:
```
...
"stakeAddressDeposit": 2000000,
...
```



If we had 1000 ada (can be found in Query the UTXO `Amount`), to calculate the change to send back to `payment.addr`:
```
expr 1000000000 - 171485 - 2000000
```
Output `997828515`.



#### Determine the TTL (Time To Live) for the transaction
We need the CURRENT TIP of the blockchain, this is, the height of the last block produced. We are looking for the value of SlotNo
```
cardano-cli query tip --testnet-magic 1097911063
```
Output `{
    "era": "Alonzo",
    "syncProgress": "100.00",
    "hash": "bfb3aa34cae0ad2cb72eaa2e5a2059b73d7c1029250e68f712cf76cc15093419",
    "epoch": 207,
    "slot": 59189349,
    "block": 3581656
}`
So at this moment the tip is on block: 369215 .
To build the transaction we need to specify the TTL (Time To Live), this is the block height limit for our transaction to be included in a block, if it is not in a block by that slot the transaction will be cancelled.
From protocol.json we can find **slot per second**. If we had `1 slot per second`  and it take us 10 minutes to build the transaction, and that we want to give it another 10 minutes window to be included in a block. So we need 20 minutes or 1200 slots. So we add 1200 to the current tip.
```expr 59189349 + 1200```
Output`> 59190549`.
So our TTL is 59190549



#### Submit the certificate with a transaction
```
cardano-cli transaction build-raw \
--tx-in c738a7a07f6aafd3dee874d0a883d752db71fb3bc2c14616ea2d34256e845053#0 \
--tx-out $(cat paymentwithstake.addr)+997828515 \
--ttl 59190549 \
--fee 171485 \
--out-file tx.raw \
--certificate-file stake.cert
```



#### Signing the transaction
This transaction has to be signed by both the payment signing key `payment.skey` and the stake signing key `stake.skey`; and includes the certificate `stake.cert`.
```
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--signing-key-file stake.skey \
--testnet-magic 1097911063 \
--out-file tx.signed
```



#### submiting the transaction
```
cardano-cli transaction submit \
--tx-file tx.signed \
--testnet-magic 1097911063
```
stake key is now registered on the blockchain.


## 6. Register Stake Pool with Metadata
> **Note** **WARNING**: Generating the **stake pool registration certificate** and the **delegation certificate** requires the cold keys. You may want to generate these certificates in a local machine and then move theme to block generator server using cold storage. Take the proper security measures to avoid exposing the cold keys to the internet.



### Create pool's metadata file
#### Creating a JSON file for pool's metadata
```
nano pool_metadata.json
```
Change `Pool_Name`, `Description of The pool`, `Ticker_Name`, `http://Relay Server IP` and save the file.
```
{
"name": "Pool_Name",
"description": "Description of The pool",
"ticker": "Ticker_Name",
"homepage": "http://Relay Server IP"
}
```
*Use `Ctrl + o` and then press `Enter` to save and `Ctrl + x` to exit the file*

Store the file in the URL of you control. For example https://stakepoolURL.com/pool_metadata.json .
Also it is possible to use a GIST in Github to store the definitions. For GIST file add `/raw` to the end of Github GIST URL and use https://t.ly or other Short Links Creator to make Short Link. Ensure that the Stake pool metadata consists of at most 512 bytes, with the URL being less than 65 characters long.

Example: 
- GIST URL https://gist.githubusercontent.com/gramezan/c1d5eee514acc5e36d05d09280dfae60/raw
- t.ly URL https://t.ly/y9dv



#### Get the hash of metadata file
```
cardano-cli stake-pool metadata-hash --pool-metadata-file <(curl -s -L -k https://t.ly/y9dv)
```
Output will be like `> 13dd1d128bd1a1d1283bd5f38c8de49659f89accb31e5399c61e428576ecb186`



### Generate Stake pool registration certificate
```
cardano-cli stake-pool registration-certificate \
--cold-verification-key-file cold.vkey \
--vrf-verification-key-file vrf.vkey \
--pool-pledge 100000000 \
--pool-cost 500000000 \
--pool-margin 0.00 \
--pool-reward-account-verification-key-file stake.vkey \
--pool-owner-stake-verification-key-file stake.vkey \
--testnet-magic 1097911063 \
--pool-relay-ipv4 185.110.190.37 \
--pool-relay-port 3001 \
--metadata-url https://t.ly/y9dv \
--metadata-hash 13dd1d128bd1a1d1283bd5f38c8de49659f89accb31e5399c61e428576ecb186 \
--out-file pool-registration.cert
```

|Parameter | Explanation | Sample|
| -------- | ------------| -----|
|cold-verification-key-file | verification cold key| cold.vkey|
|vrf-verification-key-file | verification VRS |key| vrf.vkey|
|pool-pledge | pledge lovelace | 100000000|
|pool-cost | operational costs per epoch lovelace | 500000000|
|pool-margin | share of total ada rewards that the operator takes, must be from 0.00 to 1.00 | 0.15 (15%)|
|pool-reward-account-verification-key-file | verification staking key for the rewards | stake.vkey|
|pool-owner-staking-verification-key-file | verification staking keys for the pool owners | stake.vkey|
|pool-relay-ipv4 | relay node ip address | 185.110.190.37|
|pool-relay-port | port | 3001|
|metadata-url | url of your json file | https://t.ly/y9dv |
|metadata-hash | the hash of pools json metadata file | 13dd1d128bd1a1d1283bd5f38c8de49659f89accb31e5399c61e428576ecb186|
|out-file | output file to write the certificate to | pool-registration.cert|

> **Note**
> You can use a different key for the rewards, and you can provide more than one owner key if there were multiple owners who share the pledge.
> - protocol.json file contain `"minPoolCost": 340000000`, `"stakeAddressDeposit": 2000000` and `"stakePoolDeposit": 500000000`.


The `pool-registration.cert` file should look like this:
```
{
    "type": "CertificateShelley",
    "description": "Stake Pool Registration Certificate",
    "cborHex": "8a03581c0e702bc8838307d35cc65111843a3296ea22e2861abb6743fc7944ef5820adb824c7fd1f97aea47ea7d145cb21eb44259ff6641350984ac2a4f79b7649c81a05f5e1001a1443fd00d81e820314581de0c30016fd86ae44258df0018acd14d53c2b1b97dc427326b5db383cb681581cc30016fd86ae44258df0018acd14d53c2b1b97dc427326b5db383cb6818400190bb944b96ebe25f6827168747470733a2f2f742e6c792f79396476582013dd1d128bd1a1d1283bd5f38c8de49659f89accb31e5399c61e428576ecb186"
}
```


### Generate delegation certificate pledge
We have to honor our pledge by delegating at least the pledged amount to our pool, so we have to create a delegation certificate to achieve this.
```
cardano-cli stake-address delegation-certificate \
--stake-verification-key-file stake.vkey \
--cold-verification-key-file cold.vkey \
--out-file delegation.cert
```

This creates a delegation certificate which delegates fund from all stake addresses associated with key stake.vkey to the pool belonging to cold key cold.vkey. If we had used different staking keys for the pool owners in the first step, we would need to create delegation certificates for all of them instead.


### Submit the pool certificate and delegation certificate to the blockchain
#### Query the UTXO
To finde TxHash of address
```
cardano-cli query utxo \
--address $(cat payment.addr) \
--testnet-magic 1097911063
```
Output 

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
c738a7a07f6aafd3dee874d0a883d752db71fb3bc2c14616ea2d34256e845053     0        1000000000 lovelace + TxOutDatumNone
```


#### Draft the transaction
```
cardano-cli transaction build-raw \
--tx-in c738a7a07f6aafd3dee874d0a883d752db71fb3bc2c14616ea2d34256e845053#0 \
--tx-out $(cat paymentwithstake.addr)+0 \
--ttl 0 \
--fee 0 \
--out-file tx.raw \
--certificate-file pool-registration.cert \
--certificate-file delegation.cert
```
Replace `c738a7a07f6aafd3dee874d0a883d752db71fb3bc2c14616ea2d34256e845053` in `--tx-in` with **TxHash** and **TxIx** with number in `--tx-in #number`.



#### Calculate fees and deposit
This transaction has only 1 input (the UTXO used to pay the transaction fee) and 1 output (our `paymentwithstake.addr` to which we are sending the change).
```
cardano-cli transaction calculate-min-fee \
--tx-body-file tx.raw \
--tx-in-count 1 \
--tx-out-count 1 \
--testnet-magic 1097911063 \
--witness-count 1 \
--byron-witness-count 0 \
--protocol-params-file protocol.json
```
Output `> 184685 Lovelace`.



We have to pay a deposit for the stake pool registration. The deposit amount is specified in the genesis file and in `protocol.json` under `stakeAddressDeposit`:
```
...
"poolDeposit": 500000000
...
```



If we had 1000 ada (can be found in Query the UTXO `Amount`), to calculate the change to send back to `payment.addr`:
```
expr 1000000000 - 184685 - 500000000
```
Output `499815315`.



#### Determine the TTL (Time To Live) for the transaction
```
cardano-cli query tip --testnet-magic 1097911063
```
Output `{
    "era": "Alonzo",
    "syncProgress": "100.00",
    "hash": "bfb3aa34cae0ad2cb72eaa2e5a2059b73d7c1029250e68f712cf76cc15093419",
    "epoch": 207,
    "slot": 59189349,
    "block": 3581656
}`

```expr 59189349 + 1200```
Output`> 59190549`.
So our TTL is 59190549



#### Submit the certificate with a transaction
```
cardano-cli transaction build-raw \
--tx-in c738a7a07f6aafd3dee874d0a883d752db71fb3bc2c14616ea2d34256e845053#0 \
--tx-out $(cat paymentwithstake.addr)+499815315 \
--ttl 59190549 \
--fee 171485 \
--certificate-file pool-registration.cert \
--certificate-file delegation.cert \
--out-file tx.raw
```



#### Signing the transaction
```
cardano-cli transaction sign \
--tx-body-file tx.raw \
--signing-key-file payment.skey \
--signing-key-file stake.skey \
--signing-key-file cold.skey \
--testnet-magic 1097911063 \
--out-file tx.signed
```



#### submiting the transaction
```
cardano-cli transaction submit \
--tx-file tx.signed \
--testnet-magic 1097911063
```



### Verifying Stake Pool Operation
```
cardano-cli stake-pool id --cold-verification-key-file cold.vkey --output-format hex > stakepoolid.txt
cat stakepoolid.txt
```

verify it's included in the blockchain.
```
cardano-cli query stake-snapshot --stake-pool-id $(cat stakepoolid.txt) --testnet-magic 1097911063 
```
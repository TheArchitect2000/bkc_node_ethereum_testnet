# Cardano stake pool


## How to build cardano Relay server
1. Prepare cardano-node and make it executable
------
Step 1: Prepare a machine with the operating Ubuntu system.

Step 2: Update and install packages.
```
sudo apt-get update -y
sudo apt-get -y install build-essential pkg-config libffi-dev libgmp-dev libssl-dev libtinfo-dev libsystemd-dev zlib1g-dev make g++ tmux git jq wget libncursesw5
```

Step 3: Download and unpack the Hydra binaries of `cardano-node`.
```
cd ~
mkdir cardano-node
cd cardano-node
wget https://hydra.iohk.io/build/12997298/download/1/cardano-node-1.34.0-linux.tar.gz \
tar xvzf cardano-node-1.34.0-linux.tar.gz \
rm -rf cardano-node-1.34.0-linux.tar.gz
```

Step 4: creat and copy / move `cardano-node` and `cardano-cli` to `~/.local/bin`.
```
mkdir -p ~/.local/bin
cp -p ~/cardano-node/cardano-node ~/.local/bin
cp -p ~/cardano-cli/cardano-node ~/.local/bin
```
*To Move `cardano-node` and `cardano-cli` files to new directory, replace `cp -p` with `mv`*

Step 5: Add `PATH` to last line of `.bashrc` to creat cardano-node executable files.
```
cd ~
nano .bashrc
```
```
export PATH="~/.local/bin:$PATH"
```
*Use `Ctrl + o` and then press `Enter` to save and `Ctrl + x` to exit the file*

Step 6: Enable `PATH` in current terminal.
```
cd ~
source .bashrc
```

2. Download and setup config files for relay
------
Step 1: Download `genesis`, `configuration` and `topology` files.
```
cd
mkdir relay
cd relay
wget https://hydra.iohk.io/build/8111119/download/1/testnet-config.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-byron-genesis.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-shelley-genesis.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-alonzo-genesis.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-topology.json
```

Change the file content of testnet-topology.json. *Replace `<block generator IP>` with your block generator server IP*
------
```
nano testnet-topology.json
```
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
Use this commands to save and exit the file
Ctrl+o
Enter
Ctrl+x
___

Add cardano-node and cardano-cli to environmetal variables
------
```
cd
mkdir ~/.local/bin
cp -p cardano-node/cardano-node ~/.local/bin \
cp -p cardano-node/cardano-cli ~/.local/bin \
nano .bashrc
```
Add this line to the end of the file
```
export PATH="~/.local/bin:$PATH"
```
Use this commands to save and exit the file
Ctrl+o
Enter
Ctrl+x
```
source .bashrc
```
___

To run cardano relay: *replace `<server IP address>` with IP of your server*
------
```
cardano-node run +RTS -N -A16m -qg -qb -RTS \
--topology relay/testnet-topology.json \
--database-path relay/db \
--socket-path relay/db/node.socket \
--host-addr <server IP address> \
--port 3001 \
--config relay/testnet-config.json
```
___

## How to prepare cardano Block Generator server
Download and extract cardano-node
------
```
cd
mkdir cardano-node
cd cardano-node
wget https://hydra.iohk.io/build/12997298/download/1/cardano-node-1.34.0-linux.tar.gz \
tar xvzf cardano-node-1.34.0-linux.tar.gz \
rm -rf cardano-node-1.34.0-linux.tar.gz
```

Download config files
------
```
cd
mkdir pool
cd pool
wget https://hydra.iohk.io/build/8111119/download/1/testnet-config.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-byron-genesis.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-shelley-genesis.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-alonzo-genesis.json \
wget https://hydra.iohk.io/build/8111119/download/1/testnet-topology.json
```
___

Change the file content of testnet-topology.json. *Replace `<relay IP>` with your relay server IP*
------
```
nano testnet-topology.json
```
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
Use this commands to save and exit the file
Ctrl+o
Enter
Ctrl+x
___

Add cardano-node and cardano-cli to environmetal variables
------
```
cd
mkdir ~/.local/bin
cp -p cardano-node/cardano-node ~/.local/bin \
cp -p cardano-node/cardano-cli ~/.local/bin \
nano .bashrc
```
Add this line to the end of the file
```
export PATH="~/.local/bin:$PATH"
```
Use this commands to save and exit the file
Ctrl+o
Enter
Ctrl+x
```
source .bashrc
```
___

### To creat key pairs, addresses and certifications:
```
cd
mkdir keys
cd keys
```

Payment key pair:
------
```
cardano-cli address key-gen \
--verification-key-file payment.vkey \
--signing-key-file payment.skey
```
```
cat payment.vkey
```

Payment address:
------
```
cardano-cli address build \
--payment-verification-key-file payment.vkey \
--out-file payment.addr \
--testnet-magic 1097911063
```
```
cat payment.addr
```

Stake key pair:
------
```
cardano-cli stake-address key-gen \
--verification-key-file stake.vkey \
--signing-key-file stake.skey
```

Stake address:*This address CAN'T receive payments but will receive the rewards from participating in the protocol.*
------
```
cardano-cli stake-address build \
--stake-verification-key-file stake.vkey \
--out-file stake.addr \
--testnet-magic 1097911063
```
```
cat stake.addr
```

Regenerate payment address:
------
```
cardano-cli address build \
--payment-verification-key-file payment.vkey \
--stake-verification-key-file stake.vkey \
--out-file paymentwithstake.addr \
--testnet-magic 1097911063
```

Generate Cold Keys and a Cold_counter:
------
```
cardano-cli node key-gen \
--cold-verification-key-file cold.vkey \
--cold-signing-key-file cold.skey \
--operational-certificate-issue-counter-file cold.counter
```

Generate VRF Key pair:
------
```
cardano-cli node key-gen-VRF \
--verification-key-file vrf.vkey \
--signing-key-file vrf.skey
```

Generate the KES Key pair:
------
```
cardano-cli node key-gen-KES \
--verification-key-file kes.vkey \
--signing-key-file kes.skey
```

Generate the Operational Certificate:
------
```
cat testnet-shelley-genesis.json | grep KESPeriod
cardano-cli query tip --testnet-magic 1097911063
```
*replace slotNo and slotsPerKESPeriod*
```
expr slotNo / slotsPerKESPeriod
```
*in my test it was 27*

```
cardano-cli node issue-op-cert \
--kes-verification-key-file kes.vkey \
--cold-signing-key-file cold.skey \
--operational-certificate-issue-counter cold.counter \
--kes-period 27 \
--out-file node.cert
```

To run cardano block generator: *replace `<server IP address>` with IP of your server*
------
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
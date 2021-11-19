- [Steem Node Requirement](#steem-node-requirement)
  - [Hardware](#hardware)
  - [OS](#os)
  - [Setup a Witness Node](#setup-a-witness-node)
    - [Install Node](#install-node)
    - [Activate the Witness Node:](#activate-the-witness-node)
    - [Disable the Witness Node:](#disable-the-witness-node)
    - [Configuration](#configuration)
  - [Setup a Full Node](#setup-a-full-node)
- [Steem Blockchain Explorer](#steem-blockchain-explorer)
  - [Confirmed Transaction](#confirmed-transaction)

# Steem Node Requirement

## Hardware
|  | Witness | Full Node |
|:----:|:----:|:-------:|
| RAM | 16 GB | 64 GB|
| Disk | 1TB | 2 TB|
| Network | 100mb/s | 100mb/s |
| CPU | 8 cores | 16 cores |

## OS
Ubuntu is preferred. Minimal requirement is Ubuntu 16.04. No testing on Windows or MAC.

## Setup a Witness Node
Dependencies
```bash
sudo apt install libffi-dev libssl-dev python3 python3-dev python3-pip
pip3 install -U git+git://github.com/Netherdrake/steem-python
pip3 install -U git+https://github.com/Netherdrake/conductor
```
Import a key:
```bash
steempy addkey
```
Enter the active key at the following prompt:
```
Private Key (wif) [Enter to quit]:
```
Secure the local wallet by adding a password:
```
Please provide a password for the new wallet
```
Initialize the witness:
```bash
conductor init
```
Confirm the witness parameters:
```
What is your witness account name?: XXXXXX
Witness liuye does not exist. Would you like to create it? [y/N]: y
What should be your witness URL? [https://steemdb.io]: 
How much do you want the account creation fee to be (STEEM)? [0.500 STEEM]: 3 STEEM
What should be the maximum block size? [65536]: 
What should be the SBD interest rate? [0]: 
BIP38 Wallet Password: 
Witness XXXXXX created!
```
Setup the key-pairs:
```bash
conductor keygen
```
It will generate a pair of public/private keys.
```
+-----------------------------------------------------+-------------------------------------------------------+
| Private (install on your witness node)              | Public (publish with 'conductor enable' command)      |
+-----------------------------------------------------+-------------------------------------------------------+
| 5JXXXXXXX                                           | STMXXXXXXXXXX                                         |
+-----------------------------------------------------+-------------------------------------------------------+
```
### Install Node
The steem nodes can be easily set up via Docker:
```bash
sudo apt update
sudo apt install git curl wget
git clone https://github.com/DoctorLai/steem-docker.git
cd steem-docker
```
Install Docker:
```bash
./run.sh install_docker
```


### Activate the Witness Node:
```
conductor enable <Public Key in Last Step>
```
You will be asked for the wallet password before the transaction is broadcasted.
### Disable the Witness Node:
Disable the Node to avoid missing the blocks:
```bash
conductor disable
```
This is same as broadcasting the following signing key:
```
STM1111111111111111111111111111111114T1Anm 
```

### Configuration
In the `config.ini` we need to specify the plugins, for a minimal witness node, we need to turn on the following:
```bash
plugin = witness block_api webserver condenser_api
```

## Setup a Full Node
A Full Node requires turning on more plugins:

```bash
plugin = webserver p2p json_rpc witness account_by_key reputation market_history database_api account_by_key_api network_broadcast_api reputation_api market_history_api condenser_api block_api rc_api account_history_rocksdb account_history_api
```

# Steem Blockchain Explorer
The official steem blockchain explorer is:

```url
https://steemdb.io
```

## Confirmed Transaction
A transaction is irreversible if it is confirmed by 17 witnesses or more. 

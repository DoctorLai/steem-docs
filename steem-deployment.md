- [Steem Node Requirement](#steem-node-requirement)
  - [Hardware](#hardware)
  - [OS](#os)
  - [Setup a Witness Node](#setup-a-witness-node)
    - [Docker](#docker)
    - [Docker Management Script](#docker-management-script)
      - [Build from Source](#build-from-source)
    - [Replay / ReIndex](#replay--reindex)
    - [Activate the Witness Node:](#activate-the-witness-node)
    - [Disable the Witness Node:](#disable-the-witness-node)
    - [Configuration](#configuration)
  - [Setup a Full Node](#setup-a-full-node)
  - [Setup as a Witness Node](#setup-as-a-witness-node)
- [Steem Blockchain Explorer](#steem-blockchain-explorer)
  - [Confirmed Transaction](#confirmed-transaction)
    - [Irreversible Block Number](#irreversible-block-number)
- [Interact with Local Node](#interact-with-local-node)
- [Resources](#resources)
  - [Backups](#backups)

# Steem Node Requirement

## Hardware
SSD is not required but preferred.

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
The steem nodes can be set up via Docker or building from the source.

### Docker
```bash
sudo apt update
sudo apt install git curl wget git docker.io -y
```
On other distributions you can install docker with the native package manager or with the script from get.docker.com:

```bash
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh
```

The community is at the moment using 0.23.1 image for witness block production:

```url
https://hub.docker.com/layers/106244190/ety001/steem-full-mira/0.23.1/images/sha256-958027dbc0c4f737f0da3d99fe6b3931060540407f7a749d5d707a38a118bd70?context=repo
```

We can pull this image via:

```bash
docker pull ety001/steem-full-mira:0.23.1
```

### Docker Management Script
It is easier to manage the docker steem image via:

```bash
git clone https://github.com/DoctorLai/steem-docker.git
cd steem-docker
```

`./run.sh` provides:
- To Stop the container: `./run.sh stop`
- To Start the container: `./run.sh start`
- To Restart the container: `./run.sh restart`
- To Replay: `./run.sh replay`

#### Build from Source
We can refer to [this guide](https://github.com/steemit/steem/blob/master/doc/exchangequickstart.md) but make sure we checkout the latest branch e.g. 0.23.x

### Replay / ReIndex
We can start the steemd with `--replay-blockchain` parameter to replay the blockchain. For docker installation, we need to stop and start the steem container:

```bash
docker stop <steem container>
docker rm <steem container>
rm -rf blockchain/*
docker pull steemit/steem
docker run -d --name <steem container> --env TRACK_ACCOUNT=nameofaccount --env USE_PUBLIC_BLOCKLOG=1 -p 2001:2001 -p 8090:8090 -v /path/to/steemwallet:/var/steemwallet -v /path/to/blockchain:/var/lib/steemd/blockchain --restart always steemit/steem
```

or if we are using the [Docker Management Script](#docker-management-script), we can do this:

```bash
./run.sh replay
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

## Setup as a Witness Node
For a Witness Node, we have to fill in the `witness` and `private-key` in `config.ini`:

```bash
witness = "justyy"
private-key = 5JEXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

# Steem Blockchain Explorer
The official steem blockchain explorer is:

```url
https://steemdb.io
```

## Confirmed Transaction
A transaction is irreversible if it is confirmed by 17 witnesses or more. 

### Irreversible Block Number
We can get the irreversible block number via `condenser_api.get_dynamic_global_properties` to get the `last_irreversible_block_number`.

To be safe, we can wait 63 seconds (21 blocks) to ensure the transaction is irreversible.

# Interact with Local Node
We can interact with the local node once it is up and running which we can verify via viewing the logs e.g. `docker logs -f` seeing Blocks Synchronization messages like this:

```log
1181706ms p2p_plugin.cpp:212            handle_block         ] Got 8 transactions on block 59135809 by steem-agora -- Block Time Offset: -293 ms
1184898ms p2p_plugin.cpp:212            handle_block         ] Got 11 transactions on block 59135810 by rnt1 -- Block Time Offset: -101 ms
1188020ms p2p_plugin.cpp:212            handle_block         ] Got 3 transactions on block 59135811 by upvu.witness -- Block Time Offset: 20 ms
1190872ms p2p_plugin.cpp:212            handle_block         ] Got 5 transactions on block 59135812 by exnihilo.witness -- Block Time Offset: -127 ms
1193960ms p2p_plugin.cpp:212            handle_block         ] Got 6 transactions on block 59135813 by upvu.witness -- Block Time Offset: -39 ms
1196681ms p2p_plugin.cpp:212            handle_block         ] Got 5 transactions on block 59135814 by roundblocknew -- Block Time Offset: -318 ms
1199730ms p2p_plugin.cpp:212            handle_block         ] Got 7 transactions on block 59135815 by inwi -- Block Time Offset: -269 ms
1202771ms p2p_plugin.cpp:212            handle_block         ] Got 8 transactions on block 59135816 by protoss20 -- Block Time Offset: -228 ms
1205744ms p2p_plugin.cpp:212            handle_block         ] Got 9 transactions on block 59135817 by steem-agora -- Block Time Offset: -255 ms
1209248ms p2p_plugin.cpp:212            handle_block         ] Got 6 transactions on block 59135818 by hinomaru-jp -- Block Time Offset: 248 ms
1212058ms p2p_plugin.cpp:212            handle_block         ] Got 11 transactions on block 59135819 by future.witness -- Block Time Offset: 58 ms
1214739ms p2p_plugin.cpp:212            handle_block         ] Got 8 transactions on block 59135820 by smt-wherein -- Block Time Offset: -260 ms
```

For example, if we enable the `block_api` and `webserver` plugin, we can send a request to it via:

```python
# local node
node = "https://127.0.0.1:8089"
# block number
number_block = 12345 
data = {
    "jsonrpc": "2.0",
    "method": "condenser_api.get_block",
    "params": [number_block], 
    "id": 1
}
# send to local node
request = requests.post(url = node, json = data)
print(request.json())
```

# Resources
## Backups
A Replay (Re-sync) takes time - current replay time is ranging from a few weeks to a few months+ depending on the servers' specs. We can shorten the time of Replay by using the backup files.

You can backup the files at `data/witness_node_data_dir/blockchain/` or you can obtained the regular backup files at https://files.steem.fans/ However please be noted that the account history backup is not complete.

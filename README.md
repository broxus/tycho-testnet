## Tycho Testnet

A collection of resources for the Tycho Testnet.

## Links
* [Tycho Node](https://github.com/broxus/tycho)
* [Explorer](https://testnet.tychoprotocol.com)
* [Monitoring](https://monitoring.tychoprotocol.com)
* [Validator Guide](https://github.com/broxus/tycho/blob/master/docs/validator.md)
* [Global Config](/global-config.json)

## Recommended Hardware Requirements

- 16 cores \ 32 threads CPU
- 128 GB RAM
- 1TB Fast NVMe SSD
- 1 Gbit/s network connectivity
- Public IP address

## Prerequisites

Before building from source, make sure to install the following prerequisites:

Install [Rust](https://www.rust-lang.org/tools/install)
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Install build dependencies:
```bash
sudo apt update
sudo apt install build-essential git libssl-dev zlib1g-dev pkg-config clang jq lld
```

> [!NOTE]
> The latest Clang can also be installed via the official bash script:
> ```bash
> wget -qO- https://apt.llvm.org/llvm.sh | bash -s -- 19
> ```

## How To Install

**Clone and build the node:**
```bash
git clone https://github.com/broxus/tycho
cd tycho
git checkout tags/v0.2.1 -b mynode
./scripts/install.sh
```

> [!NOTE]
> Use the latest tag for `git checkout`

**Generate node keys and configs**

```bash
# --global-config: file path or URL to the network global config
# --stake: stake value per round
tycho init --systemd --global-config https://testnet.tychoprotocol.com/global-config.json \
    --validator --stake 500000
```

<details><summary><b>Output</b></summary>
<p>

```json
{
 "node_keys": {
   "public": "18794f07dfdca1e4a2ffa655bbe5da4d396852d7ea8e55e4f92e60d209d3ae1f",
   "path": "/home/my/.tycho/node_keys.json",
   "updated": true
 },
 "elections": {
   "wallet": "-1:eb1c97aa93dbb01ac6000ace8e7f08a8b8f0078ee53d861f2e924abcd7ca2c4c",
   "public": "344659d1171701530b61f475c0795657bfacaf5a7e850050c5be2c8235d3689d",
   "stake": "500000",
   "path": "/home/my/.tycho/elections.json",
   "updated": true
 },
 "node_config": {
   "path": "/home/my/.tycho/config.json",
   "updated": true
 },
 "global_config": {
   "path": "/home/my/.tycho/global-config.json",
   "updated": true
 },
 "systemd": {
   "tycho-elect": {
     "path": "/home/my/.config/systemd/my/tycho-elect.service",
     "updated": true
   },
   "tycho": {
     "path": "/home/my/.config/systemd/my/tycho.service",
     "updated": true
   }
 }
}
```
</p>
</details>



**Backup keys**

- `~/.tycho/node_keys.json` - node keys which are used for the network and validation stuff;
- `~/.tycho/elections.json` - keys and settings of the validator wallet;

**Configure `~/.tycho/config.json`**

- `port` - node will listen on this UDP port;
- `storage.root_dir` - DB path, you might want to move it to a separate disk partition;
- `metrics.listen_addr` - Prometheus exporter will listen on this address;
- `rpc` - set it to `null` to disable an RPC endpoint if you want to save some disk space;

## How To Run

- Enable `tycho` services:
  ```bash
  systemctl enable tycho --user --now
  systemctl enable tycho-elect --user --now

  # Extend services lifetime
  sudo loginctl enable-linger
  ```

- Wait until the node is synced:
  ```bash
  # Check the current status
  tycho node status
  ```

  <details><summary><b>Output</b></summary>
  <p>

  ```json
  {
    "tycho_version": "0.1.3",
    "tycho_build": "v0.1.3",
    "public_addr": "1.2.3.4:30000",
    "local_addr": "0.0.0.0:30000",
    "adnl_id": "b947de73cff90aafe5ba17f17b94d05930e1eed8b64f332538ef9480bbfc42d4",
    "collator": {
      "supported_block_version": 50,
      "supported_capabilities": [
        "CapCreateStatsEnabled",
        "CapBounceMsgBody",
        "CapReportVersion",
        "CapShortDequeue",
        "CapFastStorageStat",
        "CapInitCodeHash",
        "CapOffHypercube",
        "CapMyCode",
        "CapFixTupleIndexBug",
        "CapFullBodyInBounced",
        "CapStorageFeeToTvm",
        "CapsTvmBugfixes2022",
        "CapWorkchains",
        "CapStcontNewFormat",
        "CapFastStorageStatBugfix",
        "CapResolveMerkleCell",
        "CapBounceAfterFailedAction",
        "CapFeeInGasUnits",
        "CapSuspendedList"
      ]
    },
    "init_mc_seqno": 0,
    "init_mc_block_id": "-1:8000000000000000:0:795ea223590f5ffa6a687ba090d37c5db6481b9d55b4bce375b0b48568413bf7:5e409bc128e785d56965294c99db2f8918005054d72c2d6a8c4080ee47ecf203",
    "latest_mc_seqno": 294019,
    "latest_mc_block_id": "-1:8000000000000000:294019:1bf1b3684c618a24dbad823453942695ff77a8cb29c7cdc9bb48f91c62bb5150:b7bbe2520bffa5bb02d9297d3f941253588e41e85e9575df4287ddd2d719a6d1",
    "time_diff": 4,
    "is_synced": true,
    "in_current_vset": false,
    "in_next_vset": false,
    "is_elected": false
  }
  ```
  </p>
  </details>



- Send testnet tokens to the validator wallet. The required amount is `10 + 2 * stake`.
> [!NOTE]
> You can find the wallet address in `~/.tycho/elections.json`.

## How To Reset

In case of major updates or fatal failures.

- Stop node services:
  ```bash
  systemctl stop tycho --user
  systemctl stop tycho-elect --user
  ```

- Delete node database:
  ```bash
  # Default db path, can be changed in ~/.tycho/config.json
  rm -rf ~/.tycho/db
  ```

- Start services:
  ```bash
  systemctl start tycho --user
  systemctl start tycho-elect --user
  ```

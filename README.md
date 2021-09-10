# IBCRelayerSetup
IBC Relayer setup between kichain-t-4 and akash-testnet-6 chains.

# Relayer Setup Docs

## Relayer Client Installation and Requirements:
### Install Relayer: 
```sh
export RELAYER=$GOPATH/src/github.com/cosmos/relayer
mkdir -p $(dirname $RELAYER) && git clone https://github.com/cosmos/relayer $RELAYER && cd $RELAYER

make install
```

### Installed Relayer Version:
```
# rly version
version: 1.0.0-rc1-152-g112205b
commit: 112205bab8224295cb046badcda512d37ea1c7b8
cosmos-sdk: v0.42.4
go: go1.16.7 linux/amd64
```

### Minimum requirements to run relayer:
* 2 CPU Cores
* 4GB Ram
* 50GB SSD

### Configure Relayer & Make IBC transfer:
- **Step-1:** Init relayer config

    ```sh
    rly config init
    ```

- **Step-2:** Write `kichain-t-4` and `akash-testnet-6` chain configs into separate json files.

    Note: Please change gas-prices value based on your rpc-node.

    Also please update your keyname as wish. `testkey` is being used as an example in this document

    ```sh
    $ echo "{\"key\":\"testkey\",\"chain-id\":\"kichain-t-4\",\"rpc-addr\":\"http://localhost:26657\",\"account-prefix\":\"tki\",\"gas-adjustment\":1.5,\"gas-prices\":\"0.001utki\",\"trusting-period\":\"503h\"}" > kichain-t-4.json

    $ echo "{\"key\":\"testkey\",\"chain-id\":\"akash-testnet-6\",\"rpc-addr\":\"http://localhost:25657\",\"account-prefix\":\"akash\",\"gas-adjustment\":1.5,\"gas-prices\":\"0.0025uakt\",\"trusting-period\":\"503h\"}" > akash-testnet-6.json
    ```
- **Step-3:** Add above both chains to relayer
    ```sh
    rly ch a -f kichain-t-4.json
    rly ch a -f akash-testnet-6.json
    ```
- **Step-4** Export source and destination chain-ids to variables.

    Note: Please update the path name as you wish
    ```sh
    export SRC=kichain-t-4
    export DST=akash-testnet-6
    export PTH=ki-akash
    ```
- **Step-5** Create IBC light clients locally
    ```sh
    rly light init $SRC -f 
    rly l i $DST -f
    ```
- **Step-6** Add/Import keys
Ensure each chain has its appropriate key. Import your keys by using:
    ```sh
    rly keys restore $SRC testkey "{{mnemonic-words}}"
    rly keys restore $DST testkey "{{mnemonic-words}}"
    ```
    show key will return address of chain's default key i.e., testkey
    ```sh
    rly keys show $SRC
    rly keys show $DST
    ```

- **Step-7** Ensure you have funds on both chains
    ```sh
    rly query balance $SRC
    rly query balance $DST
    ```

    Note: If you don't have funds, you cannot make transactions

- **Step-8** Add path between chains
    ```sh
    $ echo "{\"src\":{\"chain-id\":\"$SRC\",\"port-id\":\"transfer\",\"order\":\"unordered\",\"version\":\"ics20-1\"},\"dst\":{\"chain-id\":\"$DST\",\"port-id\":\"transfer\",\"order\":\"unordered\",\"version\":\"ics20-1\"},\"strategy\":{\"type\":\"naive\"}}" > $PTH.json
    $ rly pth add $SRC $DST $PTH -f $PTH.json
    ```

- **Step-9** Link path (creates client, connections and channels)
    ```sh
    rly tx link $PTH
    ```

- **Step-10** Start the relayer on the created path. The relayer will periodically update the clients and listen for IBC messages to relay. **(Please start this in other shell as it will run forever.)**
    ```sh
    rly start $PTH
    ```

- **Step-11** Send some funds back and forth
    ```sh
    rly q balance $SRC
    rly q balance $DST

    # transfer tokens from source chain to dst chain
    rly tx transfer $SRC $DST {{amount}} $(rly ch addr $DST)
    
    # Please wait sometime as packets will be 
    # relayed automatically by above `rly start` process
    
    # check balance again
    rly q balance $SRC
    rly q balance $DST    

    # You can send back ibc tokens from dst to src
    rly tx transfer $DST $SRC {{amount}} $(rly ch addr $SRC)
    ```    

## Relayer Configuration

Below is the complete generated configuration of relayer between ki and akash testnets:

```
global:
  api-listen-addr: :5183
  timeout: 30s
  light-cache-size: 20
chains:
- key: ki
  chain-id: kichain-t-4
  rpc-addr: http://localhost:26657
  account-prefix: tki
  gas-adjustment: 1.5
  gas-prices: 0.0001utki
  trusting-period: 503h
- key: akash
  chain-id: akash-testnet-6
  rpc-addr: http://localhost:25657
  account-prefix: akash
  gas-adjustment: 1.5
  gas-prices: 0uakt
  trusting-period: 503h
paths:
  ki-akash:
    src:
      chain-id: kichain-t-4
      client-id: 07-tendermint-141
      connection-id: connection-154
      channel-id: channel-138
      port-id: transfer
      order: unordered
      version: ics20-1
    dst:
      chain-id: akash-testnet-6
      client-id: 07-tendermint-100
      connection-id: connection-94
      channel-id: channel-90
      port-id: transfer
      order: unordered
      version: ics20-1
    strategy:
      type: naive
```

## Channels created:

* Source Channel (kichain-t-4): **channel-138**
* Destination Channnel (akash-testnet-6):  **channel-90**

## Instructions to send cross chain transactions:

Below command will transfer tokens from `kichain-t-4` to `akash-testnet-6` via channel `channel-138`:

```shell=
kid tx ibc-transfer transfer transfer channel-138 akash1u0ysle0h03ll6wzzux4fhkylrhwkvygvm9axfn 1000utki --from keyname --fees 5000utki --chain-id kichain-t-4 --node http://localhost:26657
```

Please make sure to check `kid` already installed before running above command.

And please modify `fees, node` flag values based on your node.

## Hashes of the transactions:

* [57B54ABEF7E8FF0463807D54939F4A45937F0CB2EEA85E76D7A351C78F0BC99E](https://api-challenge.blockchain.ki/txs/57B54ABEF7E8FF0463807D54939F4A45937F0CB2EEA85E76D7A351C78F0BC99E)
    
* [78778A85A79613DF71701E352FBDDCEDB4860CE0DDE6F95280B0C9DEA99D7EDB](https://api-challenge.blockchain.ki/txs/78778A85A79613DF71701E352FBDDCEDB4860CE0DDE6F95280B0C9DEA99D7EDB)
    
* [93808BAC2C5810275F0D314CD376E12065E13773209CBC2FDEB6631C5B46EFA5](https://api-challenge.blockchain.ki/txs/93808BAC2C5810275F0D314CD376E12065E13773209CBC2FDEB6631C5B46EFA5)
    
* [E1853E93BD4391FB98D60F5555C6DBE0C5614D5A09F18A15B184954A19FB9634](https://api-challenge.blockchain.ki/txs/E1853E93BD4391FB98D60F5555C6DBE0C5614D5A09F18A15B184954A19FB9634)
    
* [B55F8A26B2277997C5BC04341490B36495269068B2AF222A4BEC2289D8282D51](https://api-challenge.blockchain.ki/txs/B55F8A26B2277997C5BC04341490B36495269068B2AF222A4BEC2289D8282D51)

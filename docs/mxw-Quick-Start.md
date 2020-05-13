# Getting Started

### Running Maxonrow using docker
This Maxonrow docker is useful for running maxonrow locally and testing purpose. It has some kyc accounts. Some fee settings are set and also maintenance group are defined. Download the dockerfile from here:

* [Maxonrow docker](https://github.com/maxonrow/maxonrow-go/tree/develop/docker "Maxonrow docker") : Run a Maxonrow docker 

### Running Full node with Validator 
`MAXONROW` blockchain can be run as a full node, syncing it's state with another node or validator.

```sh
# Initial the blockchain network with a valid chain-id, 
# which will generate the config, genesis and account in respective folder
mxwd init --home ~/.mxw

# Generate validator transcation, 
# which will create the gentx folder with gentx transcation of validator account-1
mxwd gentx --name acc-1 --home ~/.mxw

# Start the node
mxwd start --home ~/.mxw
```

### Run a Single node that connect to Full node  
```sh
# Start the node by connecting to Full node via p2p.seeds 
# which the value come from gentxs-memo of genesis.json inside Full node  
mxwd start --home ~/.mxw_node4 --log_level info --p2p.seeds 

```

* [Testnets](https://github.com/githubckgoh1439/mxw-testnets "Maxonrow Testnets") : Run a Maxonrow Blockchain Locally


#### Examples of Query Account

```sh
# Get all the account details
mxwcli keys list --home ~/.mxw

# Query the address of acc-1
mxwcli keys show acc-1 --address

# Query the account, which verify the balance of acc-1
mxwcli query account $(mxwcli keys show acc-1 --address) --chain-id maxonrow-chain
```


#### Examples of Send Transaction between TWO accounts

```sh
# Query the address of acc-1
mxwcli keys show acc-1 --address

# Query the address of acc-2
mxwcli keys show acc-2 --address

# Send Transaction
mxwcli tx send $(mxwcli keys show acc-1 --address) $(mxwcli keys show acc-2 --address) 1000cin --fees 10000000000000000cin --gas 0 --memo "TRANSFER" --chain-id maxonrow-chain
```

### Testing

`MAXONROW` come with the many test cases, you can find all our test case under test folder in project soruce.

```sh
# Run tests
make test
```


# Command Line Interface (CLI)
* Introduction 
* How to use
* CLI Reference
* Modules


MXW CLI is one of several ways to interact with Maxonrow Blockchain.

MXW CLI can be used as a local wallet, you can manage your keys via MXW CLI. You can add a new key or restore your key from mnemonic words. And you can list your keys and show specified key info.

With MXW CLI, you can send transactions to Maxonrow Blockchain, like placing an order, transferring token, issuing token and so on. In addition, you can do some simple queries through CLI. For example, you can query your account's balance, transaction detail by transaction hash and etc.

### How to use
When you have downloaded MXW CLI, you can use help subcommand to see all the available commands:

```
$  ./mxwcli help
maxonrow client

Usage:
  mxwcli [command]

Available Commands:
  status         Query remote node for status
  config         Create or query an application CLI configuration file
  query          Querying subcommands
  tx             Transactions subcommands
                 
  rest-server    Start LCD (light-client daemon), a local REST server
  keys           Add or view local private keys
                 
                 
  bech           bech encoding/decoding
  kyc            kyc register
  create-keypair create the account with mnemonic, private key, public key and address
  version        Print version info of maxonrow
  help           Help about any command

Flags:
      --chain-id string   Chain ID of tendermint node
  -e, --encoding string   Binary encoding (hex|b64|btc) (default "hex")
  -h, --help              help for mxwcli
      --home string       directory for config and data (default "/Users/einnity/.mxw")
  -o, --output string     Output format (text|json) (default "text")
      --trace             print out full stack trace on errors

Use "mxwcli [command] --help" for more information about a command.
```

### CLI Reference
For detailed usage, you can refer to:

### CLI Modules
The Maxonrow SDK uses the cobra library for CLI interactions. This library makes it easy for each module to expose its own commands. To get started defining the user's CLI interactions with your module, create the following files:

* ./x/modulename/client/cli/query.go
* ./x/modulename/client/cli/tx.go

For detailed usage, you can refer to:

* [Bank Module](mxw-Node-CLI-Bank.md "What is Bank Module?")
* [KYC Module](mxw-Node-CLI-Kyc.md "What is KYC Module?")
* [Maintenance Module](mxw-Node-CLI-Maintenance.md "What is Maintenance Module?") 
* [Fee Settings Module](mxw-Node-CLI-Fee-Setting.md "What is Fee Settings Module?") 
* [Nameservice Module](mxw-Node-CLI-Nameservice.md "What is Nameservice Module?")
* [Fungible Token Module](mxw-Node-CLI-Fungible-Token.md "What is Fungible Token Module?")
* [Non-fungible Token Module](mxw-Node-CLI-Non-Fungible-Token.md "What is Non-fungible Token Module?")

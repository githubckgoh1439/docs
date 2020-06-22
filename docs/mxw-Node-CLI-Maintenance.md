# Maintenance Module

### Application Goals
The goal of the module is to let users create and maintain status of a particular proposal. 

In this section, you will learn how these simple requirements translate to application design.

### Type

We will use this when creating/updating status of a particular proposal sent to the fullnode. 
Start by creating the file msgs.go in ./x/maintenance/ folder which 
will hold customs message types for the module.
To start the SDK module, define those relevant structs in the ./x/maintenance/msgs.go file as below:

* MsgSubmitProposal
-- This is the msg type used to create a proposal from maintainers. 

```
Usage :

mxwcli tx maintenance submit-proposal [proposal name] --fee-collector [address that wish to be set as fee collector] --fee-collector-module [fee collector for module] --proposal-type [proposal type] --title [proposal title] --from [maintainers] --chain-id [chain’s id] --fees 0cin --gas 0 --action [action] --description [proposal description] --node [remote node you wish to connect to]  

Example :

mxwcli tx maintenance submit-proposal add-token-feeCollector --fee-collector mxw1g797vyveqvn4dqr70cw9y56n2wurxt9ze2n7xf --fee-collector-module token --proposal-type fee --title Add-token-feeCollector --from acc-18 --chain-id uatnet --fees 0cin --gas 0 --action add --description "add token fee collector" --node https://uatnet.usdp.io:443  


Available proposal type: 

Fungible token 
1. Able to set authorised address 
2. Able to set issuer address 
3. Able to set provider address 

Kyc 
1. Able to set authorised address 
2. Able to set issuer address 
3. Able to set provider address 

Nameservice 
1. Able to set authorised address 
2. Able to set issuer address 
3. Able to set provider address 

Fee 
1. Able to set authorised address 
2. Able to set token/ nameservice collector address 

Validator 
1. Whitelist validator address 

Non-fungible token
1. Able to set authorised address 
2. Able to set issuer address 
3. Able to set provider address 
```

* MsgCastAction
-- This is the msg type used for maintainers to cast “approve” or “reject” on certain proposal.

```
Usage :
Mxwcli tx maintenance cast-action [proposal id] --action [approve/reject] --fees 0cin --from [maintainers] --gas 0 --chain-id [chain’s id] --node [remote node you wish to connect to]

Example :

mxwcli tx maintenance cast-action 1 --action approve --fees 0cin --from acc-19 --gas 0 --chain-id uatnet --node https://uatnet.usdp.io:443  
```

### Msgs

Msgs define your application's state transitions. 
They are encoded and passed around the network wrapped in Txs. 
Messages are "owned" by a single module, meaning they are routed to only one of your applications modules. 
Each module has its own set of messages that it uses to update its subset of the chain state. 
Maxonrow SDK relies on Cosmos SDK wraps and unwraps Msgs from Txs, which means developer only have to define the relevant Msgs. <br/><br/>  
Msgs must satisfy the following interface:

![Image-1](pic/node_cli_maintenance-01.png)   


### Handlers

Next we need to write a handler function to process the Messages contained 
in the transactions delivered in each block. 
Handlers determine what actions should be taken (eg. which stores need to get updated, how, and under what conditions) 
when a given Msg is received. In MVC terms this would be the 'controller'.

In this module you have TWO types of Msgs that users 
can send to interact with the application state: 

* MsgSubmitProposal 
* MsgCastAction

** They will each have an associated Handler.


### Keeper

The main core of a Maxonrow SDK module is a piece called the Keeper. 
Each module's Keeper is responsible for CRUD operations to the main datastore of the application. 
With more sophisticated applications, modules may have access to each other's Keepers 
for cross-module interactions.<br/>In MVC terms this would be the "model". 

![Image-2](pic/node_cli_maintenance-02.png)  


### Querier

This is the place to define which queries against application state users will be able to make. 
Now that we have a running distributed state machine, it's time to enable querying our blockchain state. 
This is done through Queriers. 
These define the queries that clients can send via websocket/rpc to which our application will respond. 
Your maintenance module will expose below queries:

* Proposal
-- This takes a proposal ID and returns the proposal info.

### Client with CLI  
A Command Line Interface (CLI) will help us interact with our app once it is running on a machine somewhere. Each Module has it's own namespace within the CLI that gives it the ability to create and sign Messages destined to be handled by that module. 

The CLI for this module is broken into two files called tx.go and query.go which are located in ./x/maintenance/client/cli/. One file is for making transactions that contain messages which will ultimately update our state. The other is for making queries which will give us the ability to read information from our state. Both files utilize the Cobra library.

### tx.go
The tx.go (refer module_client.go) file contains GetTxCmd which is a standard method within the Cosmos SDK. It is referenced later in the module.go file which describes exactly which attributes a modules has. This makes it easier to incorporate different modules for different reasons at the level of the actual application.

Inside GetTxCmd we create a new module-specific command and call is maintenance. Within this command we add a sub-command for each Message type we've defined:

* GetCmdSubmitProposal
* GetCmdCastAction

Each function takes parameters from the Cobra CLI tool to create a new msg, sign it and submit it to the application to be processed. 


![Image-3](pic/node_cli_maintenance-03.png)  


### query.go
The query.go (refer module_client.go) file contains similar Cobra commands that reserve a new name space for referencing our maintenance module. Instead of creating and submitting messages however, the query.go file creates queries and returns the results in human readable form, which handles below function:

* GetCmdGetProposal

![Image-4](pic/node_cli_maintenance-04.png)  



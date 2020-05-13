This is the message type used to remove relevant multiSig transactions.

<!-- type MsgDeleteMultiSigTx struct {
	GroupAddress sdkTypes.AccAddress `json:groupAddress`
	TxID         uint64              `json:txId`
	Sender       sdkTypes.AccAddress `json:sender`
} -->

## Parameters

The message type contains the following parameters:


| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| groupAddress | string | true   | Group account | | 
| txId| uint64 | true   | Transaction ID | | 
| sender| string | true   | Sender account | | 



#### Example
```


```

-dx
## Handler

The role of the handler is to define what action(s) needs to be taken when this MsgDeleteMultiSigTx message is received.

In the file (./x/auth/handler.go) start with the following code:

![Image-1](../pic/CreateMultiSigAccount_01.png)


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgDeleteMultiSigTx message in handleMsgDeleteMultiSigTx:

![Image-2](../pic/DeleteMultiSigTx_02.png)


In this function, requirements need to be met before emitted by the network.  

* xxAuthoriser, Issuer, provider must be authorised users.
* xxUser with valid account only can proceed for KYC process.  


## Events
This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

![Image-1](../pic/DeleteMultiSigTx_03.png)  


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using DeletedMultiSigTx(string,string,string)
* Signer
* Event Parameters as below: 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| signer | string | Account address| | 
| groupAddress | string | Account address| | 
| transactionId | string | Transaction ID| | 


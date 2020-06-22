This is the message type used to transfer the ownership of a fungible token.


## Parameters

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| |
| from | string | true   | Token owner| |
| to | string | true   | New token owner| |


#### Example
```
{
    "type": "token/transferFungibleTokenOwnership",
    "value": {
        "symbol": "TT-6",
        "from": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "to": "mxw14vl7sua9jkhu0vd66eur35kzgesj5tj8pmhdjw"
    }
}

```

## Handler

The role of the handler is to define what action(s) needs to be taken when this `MsgTypeTransferNonFungibleTokenOwnership` message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

![Image-1](../pic/AcceptFungibleTokenOwnership_01.png)


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgTypeTransferFungibleTokenOwnership message in `handleMsgTransferFungibleTokenOwnership`:

![Image-2](../pic/TransferFungibleTokenOwnership_02.png)


In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must be approved, and not yet be freeze.
* Signer must be valid token owner
* Action of Re-transfer-ownership is not allowed.


## Events
This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

![Image-1](../pic/TransferFungibleTokenOwnership_03.png)


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using TransferredFungibleTokenOwnership(string,string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |
| newOwner | string | New token owner| |

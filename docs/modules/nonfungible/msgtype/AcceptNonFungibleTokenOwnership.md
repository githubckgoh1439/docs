This is the message type used to accept-ownership of a non-fungible token.

## Parameters

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| |
| from | string | true   | Token owner| |


#### Example

```
{
    "type": "nonFungible/acceptNonFungibleTokenOwnership",
    "value": {
        "symbol": "TNFT",
        "from": "mxw1zrguzs0gyqqscjhk6y8zht9xknfz8e4pnvugy6"
    }
}
```

## Handler

The role of the handler is to define what action(s) needs to be taken when this `MsgTypeAcceptNonFungibleTokenOwnership` message is received.

In the file (./x/token/nonfungible/handler.go) start with the following code:

![Image-1](../pic/MintNonFungibleItem_01.png)


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgTypeAcceptNonFungibleTokenOwnership message in `handleMsgTypeAcceptNonFungibleTokenOwnership`:

![Image-2](../pic/AcceptNonFungibleTokenOwnership_01.png)


In this function, requirements need to be met before emitted by the network.

* A valid Token.
* A valid Token owner.
* A valid New token owner.
* Token must be approved and not be freeze.
* Action of Re-accept-ownership is not allowed.


## Events
This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

![Image-1](../pic/AcceptNonFungibleTokenOwnership_03.png)


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* eventSignature : Custom Event Signature that using AcceptedNonFungibleTokenOwnership(string,string)
* from : Token owner
* eventParam : Event Parameters as below

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| from | string | Token owner| |

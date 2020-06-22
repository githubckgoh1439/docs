This is the message type used to set the status of a fungible token.


## Parameters

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| owner | string | true   | Token owner| |
| payload | Payload | true   | Payload information| |
| signatures | []Signature | true   | Array of Signature| |


#### Payload Information
| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| token | TokenData | true   | Token data| |
| pub_key | nil | true   | crypto.PubKey| |
| signature | []byte | true   | Signature| |


#### Token Data Information
| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| from | string | true   | Token owner| |
| nonce | string | true   | Nonce| |
| status | string | true   | Status, eg. freeze or unfreeze | |
| symbol | string | true   | Token symbol| |
| burnable | bool | true   | Flag of burnable| |
| tokenFees,omitempty | []TokenFee | true   | Fee Setting information| |


#### Token Fee Information
| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| action | string | true   | Action | |
| feeName | string | true   | Fee setting| |



#### Example
```
 {
    "type": "token/setFungibleTokenStatus",
    "value": {
        "owner": "mxw1j4duwuaqdj2na054rmlg3pdzncpmwzdjwtfqht",
        "payload": {
            "token": {
                "from": "mxw1lmym5599yja76d2s463390he22pcpng3zzpx4p",
                "nonce": "0",
                "status": "FREEZE",
                "symbol": "TT-4",
                "burnable": true
            },
            "pub_key": {
                "type": "tendermint/PubKeySecp256k1",
                "value": "A8I4AHjOPpkQhEFy24L8SZkwd9UxQyEyudZYcYl9f4jK"
            },
            "signature": "2rX+NvGxvB5vmEa7aeOk3nICZuUcOpv3jgRn64PtGVoT3e9Nq1mBDSUtggr5XyWJFNA6ntBzt/yvL4K+MgW+ng=="
        },
        "signatures": [
            {
                "pub_key": {
                    "type": "tendermint/PubKeySecp256k1",
                    "value": "A4PwoBS8fl/2W+V1HXrWQBn5jXci5gnYUTQzDZFqD0vl"
                },
                "signature": "y92hR1sSHk53B1tSbybOE0q2VQ9yzHnxW0heTk64Q9wHuk5lnQhqoztfM+IqaGOMbUYc3MGVhntkAVUVoXKw8A=="
            }
        ]
    }
}
```

## Handler

The role of the handler is to define what action(s) needs to be taken when this `MsgTypeSetFungibleTokenStatus` message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

![Image-1](../pic/AcceptFungibleTokenOwnership_01.png)


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.

First, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-ApproveToken message in `handleMsgSetNonFungibleTokenStatus`:

![Image-2](../pic/SetFungibleTokenStatus_02.png)


In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must not be approved.
* Signer must be authorised.
* Action of Re-approved is not allowed.


Second, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-RejectToken message in `handleMsgSetNonFungibleTokenStatus`:

![Image-2](../pic/SetFungibleTokenStatus_04.png)


In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must not be approved.
* Signer must be authorised.
* Action of Re-reject is not allowed.


Thirth, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-FreezeToken message in `handleMsgSetNonFungibleTokenStatus`:

![Image-2](../pic/SetFungibleTokenStatus_06.png)


In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must not be freeze
* Signer must be authorised.
* Action of Re-freeze is not allowed.

Next, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-UnfreezeToken message in `handleMsgSetNonFungibleTokenStatus`:

![Image-2](../pic/SetFungibleTokenStatus_08.png)


In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must be freeze.
* Signer must be authorised.
* Action of Re-unfreeze is not allowed.

Next, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-ApproveTransferTokenOwnership message in `handleMsgSetNonFungibleTokenStatus`:

![Image-2](../pic/SetFungibleTokenStatus_10.png)


In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token with TransferTokenOwnership flag equals to true.
* Signer must be authorised.
* Action of Re-approve transfer token-ownership is not allowed.

Last, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-RejectTransferTokenOwnership message in handleMsgSetNonFungibleTokenStatus:

![Image-2](../pic/SetFungibleTokenStatus_12.png)


In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token TransferTokenOwnership flag equals to true.
* Signer must be authorised.
* Action of Re-reject transfer token-ownership is not allowed.



## Events
#### 1.
This tutorial describes how to create maxonrow events for scanner base on approve token after emitted by a network.

![Image-1](../pic/SetFungibleTokenStatus_02_2.png)


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using ApprovedFungibleToken(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


#### 2.
This tutorial describes how to create maxonrow events for scanner base on reject token after emitted by a network.

![Image-2](../pic/SetFungibleTokenStatus_04_2.png)

#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using RejectedFungibleToken(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


#### 3.
This tutorial describes how to create maxonrow events for scanner base on freeze token after emitted by a network.

![Image-3](../pic/SetFungibleTokenStatus_06_2.png)


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using FrozenFungibleToken(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


#### 4.
This tutorial describes how to create maxonrow events for scanner base on unfreeze token after emitted by a network.

![Image-4](../pic/SetFungibleTokenStatus_08_2.png)


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using UnfreezeFungibleToken(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


#### 5.
This tutorial describes how to create maxonrow events for scanner base on approve transfer token-ownership after emitted by a network.

![Image-5](../pic/SetFungibleTokenStatus_10_2.png)


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using ApprovedTransferTokenOwnership(string,string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |
| newOwner | string | New token owner| |


#### 6.
This tutorial describes how to create maxonrow events for scanner base on reject transfer token-ownership after emitted by a network.

![Image-6](../pic/SetFungibleTokenStatus_12_2.png)


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using RejectedTransferTokenOwnership(string,string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |
| newOwner | string | New token owner| |

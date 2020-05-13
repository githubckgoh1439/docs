This is the message type used to delete the fee setting.

<!-- type MsgDeleteSysFeeSetting struct {
	Name   string              `json:"name"`
	Issuer sdkTypes.AccAddress `json:"issuer"`
} -->


## Parameters

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| name | string | true   | Fee Setting| | 
| issuer | string | true   | Account address| | 


-dx
#### Example

```

```

## Handler

The role of the handler is to define what action(s) needs to be taken when this MsgDeleteSysFeeSetting message is received.

In the file (./x/fee/handler.go) start with the following code:

![Image-1](../pic/SysFeeSetting_01.png)


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgDeleteSysFeeSetting message in handleMsgDeleteSysFeeSetting:

![Image-2](../pic/DeleteSysFeeSetting_02.png)


In this function, requirements need to be met before emitted by the network.  

* Issuer must be authorised user.
* Current used of Fee setting not allowed to be removed.


## Events
This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

![Image-1](../pic/DeleteSysFeeSetting_03.png)  


#### Usage
This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using DeletedFeeSetting(string,string)
* Signer
* Event Parameters as below: 

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| signer | string | Account address| | 
| name | string | Fee setting| | 


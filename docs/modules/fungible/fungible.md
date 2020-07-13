### Application Goals

The goal of the module is to let users create and maintain fungible tokens being something (such as money or a commodity) of such a nature that one part or quantity may be replaced by another equal part or quantity in paying a debt or settling an account. Use cases that can be applied using Fungible Token such as Digital Asset or Payment, Utility Token, e-Currency, Asset Tokenization or STO. 

In this section, you will learn how these simple requirements translate to application design.

### Type of Message

In this module which consists of EIGHT types of messages that users 
can send to interact with the application state: 

### MsgCreateFungibleToken
This is the message type used to create the fungible token.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| name | string | true   | Token name| |
| symbol | string | true   | Token symbol, which must be unique| |
| decimals | int | true   | Decimals value| |
| metadata | string | true   | Metadata of token| |
| fixedSupply | bool | true   | Fixed Supply value| |
| owner | string | true   | Token owner| |
| maxSupply | int | true   | Maximum Supply value| |
| fee | Fee | true   | Fee information| |


Fee Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| to | string | true   | Fee-collector| |
| value | string | true   | Fee amount to be paid| |


Example
```
{
    "type": "token/createFungibleToken",
    "value": {
        "name": "TestToken-6",
        "symbol": "TT-6",
        "decimals": "8",
        "metadata": "",
        "fixedSupply": false,
        "owner": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "maxSupply": "100000",
        "fee": {
            "to": "mxw1g6cjz0pgtchedjyacjcsldhmxcvu2z4nrud9qt",
            "value": "100000"
        }
    }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgCreateFungibleToken`
 message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		'case MsgCreateFungibleToken:
			return handleMsgCreateFungibleToken(ctx, keeper, msg)'
		case MsgSetFungibleTokenStatus:
			return handleMsgSetFungibleTokenStatus(ctx, keeper, msg)
		case MsgMintFungibleToken:
			return handleMsgMintFungibleToken(ctx, keeper, msg)
		case MsgTransferFungibleToken:
			return handleMsgTransferFungibleToken(ctx, keeper, msg)
		case MsgBurnFungibleToken:
			return handleMsgBurnFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenAccountStatus:
			return handleMsgSetFungibleTokenAccountStatus(ctx, keeper, msg)
		case MsgTransferFungibleTokenOwnership:
			return handleMsgTransferTokenOwnership(ctx, keeper, msg)
		case MsgAcceptFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)
		default:
			errMsg := fmt.Sprintf("Unrecognized fungible Msg type: %v", msg.Type())
			return sdkTypes.ErrUnknownRequest(errMsg).Result()
		}
	}
}
```

NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgTypeCreateFungibleToken message in `handleMsgCreateFungibleToken`:

```
func (k *Keeper) CreateFungibleToken(ctx sdkTypes.Context, name string, symbol string, decimals int, owner sdkTypes.AccAddress, fixedSupply bool, maxSupply sdkTypes.Uint, metadata string, fee Fee,
) sdkTypes.Result {

	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, owner)

	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Not authorised to apply for token creation.").Result()
	}

	if k.TokenExists(ctx, symbol) {
		return types.ErrTokenExists(symbol).Result()
	}

	// Overwrite the cosmos sdk tags.
	// Application fee paid at ante.go, event created here.
	amt, parseErr := sdkTypes.ParseCoins(fee.Value + types.CIN)
	if parseErr != nil {
		return sdkTypes.ErrInvalidCoins("Parse value to coins failed.").Result()
	}
	applicationFeeResult := bank.MakeBankSendEvent(ctx, owner, fee.To, amt, *k.accountKeeper)

	zero := sdkTypes.NewUintFromString("0")

	token := &Token{
		Name:        name,
		Flags:       FungibleFlag,
		Symbol:      symbol,
		Decimals:    decimals,
		Owner:       owner,
		Metadata:    metadata,
		MaxSupply:   maxSupply,
		TotalSupply: zero,
	}

	if !fixedSupply {
		token.Flags.AddFlag(MintFlag)
	}

	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, owner.String(), fee.To.String(), fee.Value}
	eventSignature := "CreatedFungibleToken(string,string,string,bignumber)"
	event := types.MakeMxwEvents(eventSignature, owner.String(), eventParam)

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: applicationFeeResult.Events.AppendEvents(event),
		Log:    resultLog.String(),
	}

}

```

In this function, requirements need to be met before emitted by the network.

* Token must be unique.
* Token owner must be KYC authorised.
* A valid Fee will be charged base on this.
* Token with FixedSupply equals to TRUE is not allowed to mint-token but allowed to Transfer, Burn, Freeze and Transfer-ownership.
* Dynamic-supply is where FixedSupply equals to FALSE, is allowed to mint-token until the MaxSupply Limit been reached.
* Action of Re-create is not allowed.

`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{symbol, owner.String(), fee.To.String(), fee.Value}
eventSignature := "CreatedFungibleToken(string,string,string,bignumber)"
event := types.MakeMxwEvents(eventSignature, owner.String(), eventParam)

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: applicationFeeResult.Events.AppendEvents(event),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using CreatedFungibleToken(string,string,string,bignumber)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |
| to | string | Fee-collector| |
| value | bignumber | Fee amount to be paid| |


### MsgSetFungibleTokenStatus
This is the message type used to set the status of a fungible token.


`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| owner | string | true   | Token owner| |
| payload | Payload | true   | Payload information| |
| signatures | []Signature | true   | Array of Signature| |


Payload Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| token | TokenData | true   | Token data| |
| pub_key | nil | true   | crypto.PubKey| |
| signature | []byte | true   | Signature| |


Token Data Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| from | string | true   | Token owner| |
| nonce | string | true   | nonce signature| |
| status | string | true   | There are different type of status, which include APPROVE, REJECT, FREEZE, UNFREEZE, APPROVE_TRANFER_TOKEN_OWNERSHIP, REJECT_TRANFER_TOKEN_OWNERSHIP. All this keywords must be matched while come to this message type | |
| symbol | string | true   | Token-symbol| |
| burnable | bool | true   | flag of token burnable setting, either TRUE or FALSE| |
| tokenFees | []TokenFee | true   | Fee Setting information| | 


`status`

| status | Details                 |
| -------- | --------------------------- |
| APPROVE  | A valid new token which yet to be approved must be signed by authorised KYC Signer with valid signature will be proceed along with a valid Fee setting scheme that been provided. Token which already been approved is not allowed to do re-submit.| | 
| REJECT  | A valid new token which yet to be approved must be signed by authorised KYC Signer with valid signature will be proceed. Token which already been rejected is not allowed to do re-submit.| | 
| FREEZE  | A valid token which already been approved must be signed by authorised KYC Signer with valid signature will be proceed. Token which already been frozen is not allowed to do re-submit.| | 
| UNFREEZE  | A valid token which already been approved and frozen must be signed by authorised KYC Signer with valid signature will be proceed. Token which already been unfreeze is not allowed to do re-submit.| | 
| APPROVE_TRANFER_TOKEN_OWNERSHIP  | A valid token which TransferTokenOwnership flag equals to true must be signed by authorised KYC Signer with valid signature will be proceed. Token which already been approved for transfer token-ownership is not allowed to do re-submit.| | 
| REJECT_TRANFER_TOKEN_OWNERSHIP  | A valid token which TransferTokenOwnership flag equals to true must be signed by authorised KYC Signer with valid signature will be proceed. Token which already been rejected for transfer token-ownership is not allowed to do re-submit.| | 


Token Fee Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| action | string | true   | Action | |
| feeName | string | true   | Fee setting| |



Example
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

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgSetFungibleTokenStatus` message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateFungibleToken:
			return handleMsgCreateFungibleToken(ctx, keeper, msg)
		'case MsgSetFungibleTokenStatus:
			return handleMsgSetFungibleTokenStatus(ctx, keeper, msg)'
		case MsgMintFungibleToken:
			return handleMsgMintFungibleToken(ctx, keeper, msg)
		case MsgTransferFungibleToken:
			return handleMsgTransferFungibleToken(ctx, keeper, msg)
		case MsgBurnFungibleToken:
			return handleMsgBurnFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenAccountStatus:
			return handleMsgSetFungibleTokenAccountStatus(ctx, keeper, msg)
		case MsgTransferFungibleTokenOwnership:
			return handleMsgTransferTokenOwnership(ctx, keeper, msg)
		case MsgAcceptFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)
		default:
			errMsg := fmt.Sprintf("Unrecognized fungible Msg type: %v", msg.Type())
			return sdkTypes.ErrUnknownRequest(errMsg).Result()
		}
	}
}
```



NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.

First, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-ApproveToken message in `handleMsgSetFungibleTokenStatus`:

```
func (k *Keeper) approveFungibleToken(ctx sdkTypes.Context, symbol string, tokenFees []TokenFee, burnable bool, metadata string, signer sdkTypes.AccAddress) sdkTypes.Result {
	var token = new(Token)
	err := k.mustGetTokenData(ctx, symbol, token)
	if err != nil {
		return err.Result()
	}

	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, token.Owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid token owner.").Result()
	}

	if token.Flags.HasFlag(ApprovedFlag) {
		return types.ErrTokenAlreadyApproved(symbol).Result()
	}

	// Assign fee to token
	for _, tokenFee := range tokenFees {
		if !k.feeKeeper.FeeSettingExists(ctx, tokenFee.FeeName) {
			return types.ErrFeeSettingNotExists(tokenFee.FeeName).Result()
		}
		err := k.feeKeeper.AssignFeeToTokenAction(ctx, tokenFee.FeeName, token.Symbol, tokenFee.Action)
		if err != nil {
			return err.Result()
		}
	}

	var flags types.Bitmask

	// if fixed supply: fixed supply doesn't allow to mint.
	if !token.Flags.HasFlag(MintFlag) {
		if burnable {
			flags = FixedSupplyBurnableFungibleTokenMask
		} else {
			flags = FixedSupplyNotBurnableFungibleTokenMask
		}
	} else {
		flags = DynamicFungibleTokenMask
	}

	token.Flags = flags + ApprovedFlag
	token.Metadata = metadata

	account := k.getFungibleAccount(ctx, symbol, token.Owner)
	if account == nil {
		account = k.createFungibleAccount(ctx, symbol, token.Owner)
	}

	if account.Frozen {
		return sdkTypes.ErrInternal("Fungible token account is frozen.").Result()
	}

	// if fixed supply: fixed supply doesn't allow to mint.
	if !token.Flags.HasFlag(MintFlag) {
		addFungibleTokenErr := k.addFungibleToken(ctx, symbol, token.Owner, token.MaxSupply)
		if addFungibleTokenErr != nil {
			return addFungibleTokenErr.Result()
		}

		token.TotalSupply = token.TotalSupply.Add(token.MaxSupply)
	}

	k.storeToken(ctx, symbol, token)

	var transferEvents sdkTypes.Events
	if !account.Balance.IsZero() {
		// Event: After approve, added total supply into token owner account.
		transferEventParam := []string{symbol, "mxw000000000000000000000000000000000000000", ownerWalletAccount.GetAddress().String(), token.TotalSupply.String()}
		transferEventSignature := "TransferredFungibleToken(string,string,string,bignumber)"
		transferEvents = types.MakeMxwEvents(transferEventSignature, "mxw000000000000000000000000000000000000000", transferEventParam)
	}
	// Event: Approved fungible token
	eventParam := []string{symbol, token.Owner.String()}
	eventSignature := "ApprovedFungibleToken(string,string)"
	events := types.MakeMxwEvents(eventSignature, signer.String(), eventParam)

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: events.AppendEvents(transferEvents),
		Log:    resultLog.String(),
	}

}

```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must not be approved.
* Signer must be KYC authorised.
* Action of Re-approved is not allowed.


Second, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-RejectToken message in `handleMsgSetFungibleTokenStatus`:

```
func (k *Keeper) RejectToken(ctx sdkTypes.Context, symbol string, signer sdkTypes.AccAddress) sdkTypes.Result {

	var token = new(Token)

	if !k.IsAuthorised(ctx, signer) {
		return sdkTypes.ErrUnauthorized("Not authorised to reject").Result()
	}

	err := k.mustGetTokenData(ctx, symbol, token)
	if err != nil {
		return err.Result()
	}

	var isApproved = token.Flags.HasFlag(ApprovedFlag)
	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, token.Owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid token owner.").Result()
	}

	if isApproved {
		return types.ErrTokenAlreadyApproved(symbol).Result()
	}

	store := ctx.KVStore(k.key)

	tokenTypeKey := getTokenKey(symbol)
	store.Delete([]byte(tokenTypeKey))

	eventParam := []string{symbol, token.Owner.String()}
	eventSignature := "RejectedFungibleToken(string,string)"

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must not be approved.
* Signer must be KYC authorised.
* Action of Re-reject is not allowed.


Thirth, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-FreezeToken message in `handleMsgSetFungibleTokenStatus`:

```
func (k *Keeper) freezeFungibleToken(ctx sdkTypes.Context, symbol string, signer sdkTypes.AccAddress, metadata string) sdkTypes.Result {
	var token = new(Token)
	k.mustGetTokenData(ctx, symbol, token)

	signerAccount := k.accountKeeper.GetAccount(ctx, signer)
	if signerAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	token.Flags.AddFlag(FrozenFlag)
	token.Metadata = metadata
	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, token.Owner.String()}
	eventSignature := "FrozenFungibleToken(string,string)"

	accountSequence := signerAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must not be freeze
* Signer must be KYC authorised.
* Action of Re-freeze is not allowed.

Next, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-UnfreezeToken message in `handleMsgSetFungibleTokenStatus`:

```
func (k *Keeper) unfreezeFungibleToken(ctx sdkTypes.Context, symbol string, signer sdkTypes.AccAddress, metadata string) sdkTypes.Result {

	var token = new(Token)
	k.mustGetTokenData(ctx, symbol, token)

	signerAccount := k.accountKeeper.GetAccount(ctx, signer)
	if signerAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	if !token.Flags.HasFlag(FrozenFlag) {
		return sdkTypes.ErrUnknownRequest("Fungible token is not frozen.").Result()
	}

	token.Flags.RemoveFlag(FrozenFlag)
	token.Metadata = metadata

	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, token.Owner.String()}
	eventSignature := "UnfreezeFungibleToken(string,string)"

	accountSequence := signerAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must be freeze.
* Signer must be KYC authorised.
* Action of Re-unfreeze is not allowed.

Next, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-ApproveTransferTokenOwnership message in `handleMsgSetFungibleTokenStatus`:

```
func (k *Keeper) ApproveTransferTokenOwnership(ctx sdkTypes.Context, symbol string, from sdkTypes.AccAddress) sdkTypes.Result {
	if !k.IsAuthorised(ctx, from) {
		return sdkTypes.ErrUnauthorized("Not authorised to accept transfer token ownership.").Result()
	}

	fromWalletAccount := k.accountKeeper.GetAccount(ctx, from)
	if fromWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	var token = new(Token)
	err := k.mustGetTokenData(ctx, symbol, token)
	if err != nil {
		return err.Result()
	}

	if !token.Flags.HasFlag(TransferTokenOwnershipFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	token.Flags.AddFlag(AcceptTokenOwnershipFlag)
	token.Flags.AddFlag(ApproveTransferTokenOwnershipFlag)

	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, token.Owner.String(), token.NewOwner.String()}
	eventSignature := "ApprovedTransferTokenOwnership(string,string,string)"

	accountSequence := fromWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}
}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token with TransferTokenOwnership flag equals to true.
* Signer must be KYC authorised.
* Action of Re-approve transfer token-ownership is not allowed.

Last, you define the actual logic for handling the MsgTypeSetFungibleTokenStatus-RejectTransferTokenOwnership message in handleMsgSetFungibleTokenStatus:

```
func (k *Keeper) RejectTransferTokenOwnership(ctx sdkTypes.Context, symbol string, from sdkTypes.AccAddress) sdkTypes.Result {
	if !k.IsAuthorised(ctx, from) {
		return sdkTypes.ErrUnauthorized("Not authorised to accept transfer token ownership.").Result()
	}

	fromWalletAccount := k.accountKeeper.GetAccount(ctx, from)
	if fromWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	var token = new(Token)
	err := k.mustGetTokenData(ctx, symbol, token)
	if err != nil {
		return err.Result()
	}

	if !token.Flags.HasFlag(TransferTokenOwnershipFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	token.Flags.RemoveFlag(TransferTokenOwnershipFlag)

	var emptyAccAddr sdkTypes.AccAddress
	token.NewOwner = emptyAccAddr

	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, token.Owner.String(), token.NewOwner.String()}
	eventSignature := "RejectedTransferTokenOwnership(string,string,string)"

	accountSequence := fromWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}
}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token TransferTokenOwnership flag equals to true.
* Signer must be KYC authorised.
* Action of Re-reject transfer token-ownership is not allowed.



`Events`

1.This tutorial describes how to create maxonrow events for scanner base on approve token after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String()}
eventSignature := "ApprovedFungibleToken(string,string)"
events := types.MakeMxwEvents(eventSignature, signer.String(), eventParam)

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: events.AppendEvents(transferEvents),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using ApprovedFungibleToken(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


2.This tutorial describes how to create maxonrow events for scanner base on reject token after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String()}
eventSignature := "RejectedFungibleToken(string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
    Log:    resultLog.String(),
}
```
Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using RejectedFungibleToken(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


3.This tutorial describes how to create maxonrow events for scanner base on freeze token after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String()}
eventSignature := "FrozenFungibleToken(string,string)"

accountSequence := signerAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using FrozenFungibleToken(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


4.This tutorial describes how to create maxonrow events for scanner base on unfreeze token after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String()}
eventSignature := "UnfreezeFungibleToken(string,string)"

accountSequence := signerAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, signer.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using UnfreezeFungibleToken(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


5.This tutorial describes how to create maxonrow events for scanner base on approve transfer token-ownership after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String(), token.NewOwner.String()}
eventSignature := "ApprovedTransferTokenOwnership(string,string,string)"

accountSequence := fromWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using ApprovedTransferTokenOwnership(string,string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |
| newOwner | string | New token owner| |


6.This tutorial describes how to create maxonrow events for scanner base on reject transfer token-ownership after emitted by a network.

```
eventParam := []string{symbol, token.Owner.String(), token.NewOwner.String()}
eventSignature := "RejectedTransferTokenOwnership(string,string,string)"

accountSequence := fromWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using RejectedTransferTokenOwnership(string,string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |
| newOwner | string | New token owner| |


### MsgTransferFungibleToken
This is the message type used to transfer the fungible token.


`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| |
| from | string | true   | Token owner| |
| to | string | true   | New token owner| |
| value | int | true   | Value| |


Example
```
{
    "type": "token/transferFungibleToken",
    "value": {
        "symbol": "TT-6",
        "value": "0",
        "from": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "to": "mxw1w0m8xqy0fpgkf6luwu666f5hhl3tm0sq53snw5"
    }
}
```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgTransferFungibleToken` message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateFungibleToken:
			return handleMsgCreateFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenStatus:
			return handleMsgSetFungibleTokenStatus(ctx, keeper, msg)
		case MsgMintFungibleToken:
			return handleMsgMintFungibleToken(ctx, keeper, msg)
		'case MsgTransferFungibleToken:
			return handleMsgTransferFungibleToken(ctx, keeper, msg)'
		case MsgBurnFungibleToken:
			return handleMsgBurnFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenAccountStatus:
			return handleMsgSetFungibleTokenAccountStatus(ctx, keeper, msg)
		case MsgTransferFungibleTokenOwnership:
			return handleMsgTransferTokenOwnership(ctx, keeper, msg)
		case MsgAcceptFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)
		default:
			errMsg := fmt.Sprintf("Unrecognized fungible Msg type: %v", msg.Type())
			return sdkTypes.ErrUnknownRequest(errMsg).Result()
		}
	}
}
```


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgTypeTransferFungibleToken message in `handleMsgTransferFungibleToken`:

```
func (k *Keeper) TransferFungibleToken(ctx sdkTypes.Context, symbol string, from, to sdkTypes.AccAddress, value sdkTypes.Uint) sdkTypes.Result {
	var token = new(Token)
	if exists := k.getTokenData(ctx, symbol, token); !exists {
		return types.ErrTokenInvalid().Result()
	}

	fromAccount := k.accountKeeper.GetAccount(ctx, from)
	if fromAccount == nil {
		return types.ErrInvalidTokenAccount().Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	ownerAccount := k.getFungibleAccount(ctx, symbol, from)
	if ownerAccount == nil {
		return sdkTypes.ErrUnknownRequest("Owner doesn't have such token").Result()
	}

	if ownerAccount.Frozen {
		return types.ErrTokenAccountFrozen().Result()
	}

	if ownerAccount.Balance.LT(value) {
		return types.ErrInvalidTokenAccountBalance(fmt.Sprintf("Not enough tokens. Have only %v", ownerAccount.Balance.String())).Result()
	}

	newOwnerAccount := k.getFungibleAccount(ctx, symbol, to)
	if newOwnerAccount == nil {
		newOwnerAccount = k.createFungibleAccount(ctx, symbol, to)
	}

	if newOwnerAccount.Frozen {
		return types.ErrTokenAccountFrozen().Result()
	}

	subFungibleTokenErr := k.subFungibleToken(ctx, symbol, from, value)
	if subFungibleTokenErr != nil {
		return subFungibleTokenErr.Result()
	}

	addFungibleTokenErr := k.addFungibleToken(ctx, symbol, to, value)
	if addFungibleTokenErr != nil {
		return addFungibleTokenErr.Result()
	}

	eventParam := []string{symbol, from.String(), to.String(), value.String()}
	eventSignature := "TransferredFungibleToken(string,string,string,bignumber)"

	accountSequence := fromAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Signer must be a valid token owner.
* Current token owner's account is not in freeze condition.
* New token owner's account is not in freeze condition.
* Current token owner must have enough balance in order to do transfer amount to new token owner
* Token with FixedSupply equals to TRUE is not allowed to mint-token but allowed to Transfer.
* Dynamic-supply is where FixedSupply equals to FALSE, is allowed to mint-token until the MaxSupply Limit been reached. Dynamic-supply also allowed for to do Transfer.
* Action of Re-transfer is allowed if have enough balance.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{symbol, from.String(), to.String(), value.String()}
eventSignature := "TransferredFungibleToken(string,string,string,bignumber)"

accountSequence := fromAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using TransferredFungibleToken(string,string,string,bignumber)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| from | string | Token owner| |
| to | string | New token owner| |
| value | string | Value| |


### MsgMintFungibleToken
This is the message type used to mint the amount of fungible token to an account owner.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| |
| owner | string | true   | Token owner| |
| to | string | true   | Token account owner| |
| value | int | true   | Mint value| |


Example
```
{
    "type": "token/mintFungibleToken",
    "value": {
        "symbol": "TT-2b",
        "value": "100000",
        "owner": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
        "to": "mxw14vl7sua9jkhu0vd66eur35kzgesj5tj8pmhdjw"
    }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgMintFungibleToken` message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateFungibleToken:
			return handleMsgCreateFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenStatus:
			return handleMsgSetFungibleTokenStatus(ctx, keeper, msg)
		'case MsgMintFungibleToken:
			return handleMsgMintFungibleToken(ctx, keeper, msg)'
		case MsgTransferFungibleToken:
			return handleMsgTransferFungibleToken(ctx, keeper, msg)
		case MsgBurnFungibleToken:
			return handleMsgBurnFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenAccountStatus:
			return handleMsgSetFungibleTokenAccountStatus(ctx, keeper, msg)
		case MsgTransferFungibleTokenOwnership:
			return handleMsgTransferTokenOwnership(ctx, keeper, msg)
		case MsgAcceptFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)
		default:
			errMsg := fmt.Sprintf("Unrecognized fungible Msg type: %v", msg.Type())
			return sdkTypes.ErrUnknownRequest(errMsg).Result()
		}
	}
}
```


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgTypeMintFungibleToken message in `handleMsgMintFungibleToken`:

```
func (k *Keeper) MintFungibleToken(ctx sdkTypes.Context, symbol string, from sdkTypes.AccAddress, to sdkTypes.AccAddress, value sdkTypes.Uint) sdkTypes.Result {

	var token = new(Token)
	if exists := k.getTokenData(ctx, symbol, token); !exists {
		return types.ErrInvalidTokenSymbol(symbol).Result()
	}

	if !token.Flags.HasFlag(MintFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	// Wallet account
	minterAccount := k.accountKeeper.GetAccount(ctx, from)
	if minterAccount == nil {
		return types.ErrInvalidTokenMinter().Result()
	}

	// minter can only be the owner of the token
	tokenOwnerAccount := k.getFungibleAccount(ctx, symbol, from)

	// token account
	if tokenOwnerAccount == nil {
		return types.ErrInvalidTokenMinter().Result()
	}

	if tokenOwnerAccount.Frozen {
		return types.ErrTokenAccountFrozen().Result()
	}

	if !token.Owner.Equals(from) {
		return types.ErrInvalidTokenMinter().Result()
	}

	if !token.Flags.HasFlag(ApprovedFlag) {
		return types.ErrTokenInvalid().Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	account := k.getFungibleAccount(ctx, symbol, to)
	if account == nil {
		account = k.createFungibleAccount(ctx, symbol, to)
	}

	if account.Frozen {
		return types.ErrTokenAccountFrozen().Result()
	}

	token.TotalSupply = token.TotalSupply.Add(value)

	// max supply 0 means is dynamic supply
	if !token.MaxSupply.IsZero() {
		if token.TotalSupply.GT(token.MaxSupply) {
			return types.ErrInvalidTokenSupply().Result()
		}
	}

	addFungibleTokenErr := k.addFungibleToken(ctx, symbol, to, value)
	if addFungibleTokenErr != nil {
		return addFungibleTokenErr.Result()
	}

	k.storeToken(ctx, symbol, token)

	eventParam := []string{symbol, "mxw000000000000000000000000000000000000000", to.String(), value.String()}
	eventSignature := "MintedFungibleToken(string,string,string,bignumber)"

	accountSequence := minterAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}
}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token which Mint Flag equals to true and must be approved, and not yet be freeze.
* Token can only be minted by token-creator.
* Token with FixedSupply equals to TRUE is not allowed to mint-token but allowed to Transfer, Burn, Freeze.
* Dynamic-supply is where FixedSupply equals to FALSE, is allowed to mint-token until the MaxSupply Limit been reached. Dynamic-supply also allowed for to Transfer, Burn, Freeze and Transfer-ownership.
* Action of Re-mint is allowed if not yet reached the total-supply-limit (as Fixed-Supply).


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{symbol, "mxw000000000000000000000000000000000000000", to.String(), value.String()}
eventSignature := "MintedFungibleToken(string,string,string,bignumber)"

accountSequence := minterAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using MintedFungibleToken(string,string,string,bignumber)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token account owner| |
| to | string | Token account owner| |
| value | string | Mint value| |


### MsgBurnFungibleToken
This is the message type used to burn the amount of fungible token of an account owner.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| |
| value | string | true   | Burn value| |
| from | string | true   | Token account owner| |


Example
```
{
    "type": "token/burnFungibleToken",
    "value": {
        "symbol": "TT-2",
        "value": "200",
        "from": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73"
    }
}

```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgBurnFungibleToken` message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateFungibleToken:
			return handleMsgCreateFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenStatus:
			return handleMsgSetFungibleTokenStatus(ctx, keeper, msg)
		case MsgMintFungibleToken:
			return handleMsgMintFungibleToken(ctx, keeper, msg)
		case MsgTransferFungibleToken:
			return handleMsgTransferFungibleToken(ctx, keeper, msg)
		'case MsgBurnFungibleToken:
			return handleMsgBurnFungibleToken(ctx, keeper, msg)'
		case MsgSetFungibleTokenAccountStatus:
			return handleMsgSetFungibleTokenAccountStatus(ctx, keeper, msg)
		case MsgTransferFungibleTokenOwnership:
			return handleMsgTransferTokenOwnership(ctx, keeper, msg)
		case MsgAcceptFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)
		default:
			errMsg := fmt.Sprintf("Unrecognized fungible Msg type: %v", msg.Type())
			return sdkTypes.ErrUnknownRequest(errMsg).Result()
		}
	}
}
```


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgTypeBurnFungibleToken message in `handleMsgBurnFungibleItem`:

```
func (k *Keeper) BurnFungibleToken(ctx sdkTypes.Context, symbol string, owner sdkTypes.AccAddress, value sdkTypes.Uint) sdkTypes.Result {
	var token = new(Token)
	if exists := k.getTokenData(ctx, symbol, token); !exists {
		return types.ErrInvalidTokenSymbol(symbol).Result()
	}

	if !token.Flags.HasFlag(BurnFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	ownerAccount := k.accountKeeper.GetAccount(ctx, owner)
	if ownerAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid account to burn from.").Result()
	}

	if !token.Flags.HasFlag(ApprovedFlag) {
		return types.ErrTokenInvalid().Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	account := k.getFungibleAccount(ctx, symbol, owner)
	if account == nil {
		return types.ErrInvalidTokenAccount().Result()
	}

	if account.Frozen {
		return types.ErrTokenAccountFrozen().Result()
	}

	if account.Balance.LT(value) {
		return types.ErrInvalidTokenAccountBalance(fmt.Sprintf("Not enough tokens. Have only %v", account.Balance.String())).Result()
	}

	token.TotalSupply = token.TotalSupply.Sub(value)
	k.storeToken(ctx, symbol, token)

	subFungibleTokenErr := k.subFungibleToken(ctx, symbol, owner, value)
	if subFungibleTokenErr != nil {
		return subFungibleTokenErr.Result()
	}

	eventParam := []string{symbol, owner.String(), "mxw000000000000000000000000000000000000000", value.String()}
	eventSignature := "BurnedFungibleToken(string,string,string,bignumber)"

	accountSequence := ownerAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* A valid Token account.
* Token must be approved before this, and not be freeze. Also burnable flag must equals to true.
* Signer who is the Token owner need to be authorised to do this process.
* Token with FixedSupply equals to TRUE is not allowed to mint-token but allowed to Burn.
* Dynamic-supply is where FixedSupply equals to FALSE, is allowed to mint-token until the MaxSupply Limit been reached. Dynamic-supply also allowed for to do Burn.
* Action of Re-burn is allowed if the balance amount is enough to do burning.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{symbol, owner.String(), "mxw000000000000000000000000000000000000000", value.String()}
eventSignature := "BurnedFungibleToken(string,string,string,bignumber)"

accountSequence := ownerAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using BurnedFungibleToken(string,string,string,bignumber)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token account owner| |
| account | string | Token account owner| |
| value | string | Burn value| |


### MsgTransferFungibleTokenOwnership
This is the message type used to transfer the ownership of a fungible token.


`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| |
| from | string | true   | Token owner| |
| to | string | true   | New token owner| |


Example
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

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgTransferFungibleTokenOwnership` message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateFungibleToken:
			return handleMsgCreateFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenStatus:
			return handleMsgSetFungibleTokenStatus(ctx, keeper, msg)
		case MsgMintFungibleToken:
			return handleMsgMintFungibleToken(ctx, keeper, msg)
		case MsgTransferFungibleToken:
			return handleMsgTransferFungibleToken(ctx, keeper, msg)
		case MsgBurnFungibleToken:
			return handleMsgBurnFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenAccountStatus:
			return handleMsgSetFungibleTokenAccountStatus(ctx, keeper, msg)
		'case MsgTransferFungibleTokenOwnership:
			return handleMsgTransferTokenOwnership(ctx, keeper, msg)'
		case MsgAcceptFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)
		default:
			errMsg := fmt.Sprintf("Unrecognized fungible Msg type: %v", msg.Type())
			return sdkTypes.ErrUnknownRequest(errMsg).Result()
		}
	}
}
```


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgTypeTransferFungibleTokenOwnership message in `handleMsgTransferFungibleTokenOwnership`:

```
func (k *Keeper) transferFungibleTokenOwnership(ctx sdkTypes.Context, from sdkTypes.AccAddress, to sdkTypes.AccAddress, token *Token, metadata string) sdkTypes.Result {

	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, token.Owner)
	if ownerWalletAccount == nil {
		return types.ErrInvalidTokenOwner().Result()
	}

	if ownerWalletAccount != nil && !token.Owner.Equals(from) {
		return types.ErrInvalidTokenOwner().Result()
	}

	if !token.IsApproved() {
		// TODO: Please define an error code
		return sdkTypes.ErrUnknownRequest("Token is not approved.").Result()
	}

	if token.IsFrozen() {
		return types.ErrTokenFrozen().Result()
	}

	newOwnerAccount := k.getFungibleAccount(ctx, token.Symbol, to)
	if newOwnerAccount == nil {
		newOwnerAccount = k.createFungibleAccount(ctx, token.Symbol, to)
	}

	if newOwnerAccount.Frozen {
		return sdkTypes.ErrUnknownRequest("New owner is frozen").Result()
	}

	// set token newowner to new owner, pending for accepting by new owner
	token.NewOwner = to
	token.Metadata = metadata
	token.Flags.AddFlag(TransferTokenOwnershipFlag)

	k.storeToken(ctx, token.Symbol, token)

	eventParam := []string{token.Symbol, from.String(), to.String()}
	eventSignature := "TransferredFungibleTokenOwnership(string,string,string)"

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}
}

```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* Token must be approved, and not yet be freeze.
* Signer must be valid token owner
* Token with FixedSupply equals to TRUE is not allowed to mint-token but allowed to Transfer-ownership.
* Dynamic-supply is where FixedSupply equals to FALSE, is allowed to mint-token until the MaxSupply Limit been reached. Dynamic-supply also allowed for to do Transfer-ownership.
* Action of Re-transfer-ownership is not allowed.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{token.Symbol, from.String(), to.String()}
eventSignature := "TransferredFungibleTokenOwnership(string,string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using TransferredFungibleTokenOwnership(string,string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |
| newOwner | string | New token owner| |


### MsgAcceptFungibleTokenOwnership
This is the message type used to accept the ownership of a fungible token.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| symbol | string | true   | Token symbol, which must be unique| |
| from | string | true   | Token owner| |


Example

```
{
    "type": "token/acceptFungibleTokenOwnership",
    "value": {
        "symbol": "TT-6",
        "from": "mxw14vl7sua9jkhu0vd66eur35kzgesj5tj8pmhdjw"
    }
}
```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgAcceptFungibleTokenOwnership` message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateFungibleToken:
			return handleMsgCreateFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenStatus:
			return handleMsgSetFungibleTokenStatus(ctx, keeper, msg)
		case MsgMintFungibleToken:
			return handleMsgMintFungibleToken(ctx, keeper, msg)
		case MsgTransferFungibleToken:
			return handleMsgTransferFungibleToken(ctx, keeper, msg)
		case MsgBurnFungibleToken:
			return handleMsgBurnFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenAccountStatus:
			return handleMsgSetFungibleTokenAccountStatus(ctx, keeper, msg)
		case MsgTransferFungibleTokenOwnership:
			return handleMsgTransferTokenOwnership(ctx, keeper, msg)
		'case MsgAcceptFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)'
		default:
			errMsg := fmt.Sprintf("Unrecognized fungible Msg type: %v", msg.Type())
			return sdkTypes.ErrUnknownRequest(errMsg).Result()
		}
	}
}
```


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.
Now, you need to define the actual logic for handling the MsgTypeAcceptFungibleTokenOwnership message in `handleMsgTypeAcceptFungibleTokenOwnership`:

```
func (k *Keeper) acceptFungibleTokenOwnership(ctx sdkTypes.Context, from sdkTypes.AccAddress, token *Token, metadata string) sdkTypes.Result {

	if !token.Flags.HasFlag(AcceptTokenOwnershipFlag) && !token.Flags.HasFlag(ApproveTransferTokenOwnershipFlag) && !token.Flags.HasFlag(TransferTokenOwnershipFlag) {
		return types.ErrInvalidTokenAction().Result()
	}

	// validation of exisisting owner account
	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, token.Owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid token owner.").Result()
	}

	// validation of new owner account
	newOwnerWalletAccount := k.accountKeeper.GetAccount(ctx, token.Owner)
	if newOwnerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid token owner.").Result()
	}

	newOwnerAccount := k.getFungibleAccount(ctx, token.Symbol, from)
	if newOwnerAccount == nil {
		return sdkTypes.ErrUnknownRequest("New owner account is not found.").Result()
	}

	if newOwnerAccount.Frozen {
		return sdkTypes.ErrUnknownRequest("New owner is frozen").Result()
	}

	if newOwnerWalletAccount != nil && token.NewOwner.String() != from.String() {
		return types.ErrInvalidTokenNewOwner().Result()
	}

	if !token.Flags.HasFlag(ApprovedFlag) {
		return sdkTypes.ErrUnknownRequest("Token is not approved.").Result()
	}

	if token.Flags.HasFlag(FrozenFlag) {
		return types.ErrTokenFrozen().Result()
	}

	//TO-DO: if there is need to set token.NewOwner to empty
	// accepting token ownership, remove newowner move the newowner into owner.
	var emptyAccAddr sdkTypes.AccAddress
	token.Owner = from
	token.NewOwner = emptyAccAddr
	token.Metadata = metadata

	token.Flags.RemoveFlag(ApproveTransferTokenOwnershipFlag)
	token.Flags.RemoveFlag(AcceptTokenOwnershipFlag)
	token.Flags.RemoveFlag(TransferTokenOwnershipFlag)
	k.storeToken(ctx, token.Symbol, token)

	eventParam := []string{token.Symbol, from.String()}
	eventSignature := "AcceptedFungibleTokenOwnership(string,string)"

	accountSequence := newOwnerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
		Log:    resultLog.String(),
	}

}

```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* A valid Token owner.
* A valid New token owner.
* Token must be approved and not be freeze.
* Token with FixedSupply equals to TRUE is not allowed to mint-token but allowed to Transfer-ownership.
* Dynamic-supply is where FixedSupply equals to FALSE, is allowed to mint-token until the MaxSupply Limit been reached. Dynamic-supply also allowed for to do Transfer-ownership.
* Action of Re-accept-ownership is not allowed.


`Events`

This tutorial describes how to create maxonrow events for scanner on this after emitted by a network.

```
eventParam := []string{token.Symbol, from.String()}
eventSignature := "AcceptedFungibleTokenOwnership(string,string)"

accountSequence := newOwnerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, from.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using AcceptedFungibleTokenOwnership(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| from | string | New token owner| |


### MsgSetFungibleTokenAccountStatus
This is the message type used to set the account status of a fungible token.

`Parameters`

The message type contains the following parameters:

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| owner | string | true   | Item owner| |
| payload | TokenAccountPayload | true   | Account Payload information| |
| signatures | []Signature | true   | Array of Signature| |


Account Payload Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| tokenAccount | TokenAccount | true   | Token account| |
| pub_key | nil | true   | crypto.PubKey| |
| signature | []byte | true   | Signature| |


Token Account Information

| Name | Type | Required | Description                 |
| ---- | ---- | -------- | --------------------------- |
| from | string | true   | Token owner| |
| nonce | string | true   | nonce signature| |
| status | string | true   | There are different type of status, which include FREEZE_ACCOUNT, UNFREEZE_ACCOUNT. All this keywords must be matched while come to this message type | |
| symbol | string | true   | Token-symbol| |
| to | string | true   | Token account address| |

`status`

| status | Details                 |
| -------- | --------------------------- |
| FREEZE_ACCOUNT  | A valid token which already been approved must be signed by authorised KYC Signer with valid signature will be proceed. Token which already been frozen is not allowed to do re-submit.| | 
| UNFREEZE_ACCOUNT  | A valid token which already been approved and frozen must be signed by authorised KYC Signer with valid signature will be proceed. Token which already been unfreeze is not allowed to do re-submit.| | 


Example
```
{
    "type": "token/setFungibleTokenAccountStatus",
    "value": {
        "owner": "mxw1j4duwuaqdj2na054rmlg3pdzncpmwzdjwtfqht",
        "payload": {
            "tokenAccount": {
                "from": "mxw1lmym5599yja76d2s463390he22pcpng3zzpx4p",
                "nonce": "0",
                "status": "FREEZE_ACCOUNT",
                "symbol": "TFT-Acc1",
                "to": "mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73"
            },
            "pub_key": {
                "type": "tendermint/PubKeySecp256k1",
                "value": "A8I4AHjOPpkQhEFy24L8SZkwd9UxQyEyudZYcYl9f4jK"
            },
            "signature": "XgDB/IWOX2M2gwi+1Voi4YaNhCJzAZdG00hbUTtTAFhEtnl+7i1BHi70XX5pwH9s+74y5gZunT2BFZoJXm/xzA=="
        },
        "signatures": [
            {
                "pub_key": {
                    "type": "tendermint/PubKeySecp256k1",
                    "value": "A4PwoBS8fl/2W+V1HXrWQBn5jXci5gnYUTQzDZFqD0vl"
                },
                "signature": "STAuPiC0512i3HHYm6pK+k0LXwCMLr1DgAjO+bV3UAct068WnvoWNh1Jd7xcWrnaghLfdWZT8yknhEkKd9XFRA=="
            }
        ]
    }
}
```

`Handler`

The role of the handler is to define what action(s) needs to be taken when this `MsgSetFungibleTokenAccountStatus` message is received.

In the file (./x/token/fungible/handler.go) start with the following code:

```
func NewHandler(keeper *Keeper) sdkTypes.Handler {
	return func(ctx sdkTypes.Context, msg sdkTypes.Msg) sdkTypes.Result {
		switch msg := msg.(type) {
		case MsgCreateFungibleToken:
			return handleMsgCreateFungibleToken(ctx, keeper, msg)
		case MsgSetFungibleTokenStatus:
			return handleMsgSetFungibleTokenStatus(ctx, keeper, msg)
		case MsgMintFungibleToken:
			return handleMsgMintFungibleToken(ctx, keeper, msg)
		case MsgTransferFungibleToken:
			return handleMsgTransferFungibleToken(ctx, keeper, msg)
		case MsgBurnFungibleToken:
			return handleMsgBurnFungibleToken(ctx, keeper, msg)
		'case MsgSetFungibleTokenAccountStatus:
			return handleMsgSetFungibleTokenAccountStatus(ctx, keeper, msg)'
		case MsgTransferFungibleTokenOwnership:
			return handleMsgTransferTokenOwnership(ctx, keeper, msg)
		case MsgAcceptFungibleTokenOwnership:
			return handleMsgAcceptTokenOwnership(ctx, keeper, msg)
		default:
			errMsg := fmt.Sprintf("Unrecognized fungible Msg type: %v", msg.Type())
			return sdkTypes.ErrUnknownRequest(errMsg).Result()
		}
	}
}
```


NewHandler is essentially a sub-router that directs messages coming into this module to the proper handler.

First, you define the actual logic for handling the MsgTypeSetFungibleTokenAccountStatus-FreezeTokenAccount message in `handleMsgSetFungibleTokenAccountStatus`:

```
func (k *Keeper) FreezeFungibleTokenAccount(ctx sdkTypes.Context, symbol string, owner sdkTypes.AccAddress, tokenAccount sdkTypes.AccAddress, metadata string) sdkTypes.Result {
	var token = new(Token)
	if exists := k.getTokenData(ctx, symbol, token); !exists {
		return sdkTypes.ErrUnknownRequest("No such fungible token.").Result()
	}

	if !k.IsAuthorised(ctx, owner) {
		return sdkTypes.ErrUnauthorized("Not authorised to freeze token account.").Result()
	}

	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	fungibleAccount := k.getFungibleAccount(ctx, symbol, tokenAccount)
	if fungibleAccount == nil {
		return sdkTypes.ErrUnknownRequest("No such token account to freeze.").Result()
	}

	if fungibleAccount.Frozen {
		return sdkTypes.ErrUnknownRequest("Fungible token account already frozen.").Result()
	}

	fungibleAccount.Frozen = true
	fungibleAccount.Metadata = metadata

	k.storeFungibleAccount(ctx, symbol, fungibleAccount)

	eventParam := []string{symbol, tokenAccount.String()}
	eventSignature := "FrozenFungibleTokenAccount(string,string)"

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
		Log:    resultLog.String(),
	}
}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* A valid Token account which must not be freeze.
* Signer must be KYC authorised.
* Action of Re-freeze-item is not allowed.


Last, you define the actual logic for handling the MsgTypeSetFungibleTokenAccountStatus-UnfreezeTokenAccount message in `handleMsgSetFungibleTokenAccountStatus`:

```
func (k *Keeper) UnfreezeFungibleTokenAccount(ctx sdkTypes.Context, symbol string, owner sdkTypes.AccAddress, tokenAccount sdkTypes.AccAddress, metadata string) sdkTypes.Result {
	if !k.IsAuthorised(ctx, owner) {
		return sdkTypes.ErrUnauthorized("Not authorised to unfreeze token account.").Result()
	}

	ownerWalletAccount := k.accountKeeper.GetAccount(ctx, owner)
	if ownerWalletAccount == nil {
		return sdkTypes.ErrInvalidSequence("Invalid signer.").Result()
	}

	var token = new(Token)
	if exists := k.getTokenData(ctx, symbol, token); !exists {
		return sdkTypes.ErrUnknownRequest("No such fungible token.").Result()
	}

	fungibleAccount := k.getFungibleAccount(ctx, symbol, tokenAccount)
	if fungibleAccount == nil {
		return sdkTypes.ErrUnknownRequest("No such fungible token account to unfreeze.").Result()
	}

	if !fungibleAccount.Frozen {
		return sdkTypes.ErrUnknownRequest("Fungible token account not frozen.").Result()
	}

	fungibleAccount.Frozen = false
	fungibleAccount.Metadata = metadata

	k.storeFungibleAccount(ctx, symbol, fungibleAccount)

	eventParam := []string{symbol, tokenAccount.String()}
	eventSignature := "UnfreezeFungibleTokenAccount(string,string)"

	accountSequence := ownerWalletAccount.GetSequence()
	resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

	return sdkTypes.Result{
		Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
		Log:    resultLog.String(),
	}

}
```

In this function, requirements need to be met before emitted by the network.

* A valid Token.
* A valid Token account which must be freeze.
* Signer must be KYC authorised.
* Action of Re-unfreeze-item is not allowed.



`Events`

1.This tutorial describes how to create maxonrow events for scanner base on freeze token account
after emitted by a network.

```
eventParam := []string{symbol, tokenAccount.String()}
eventSignature := "FrozenFungibleTokenAccount(string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using FrozenFungibleTokenAccount(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |



2.This tutorial describes how to create maxonrow events for scanner base on unfreeze token account after emitted by a network.

```
eventParam := []string{symbol, tokenAccount.String()}
eventSignature := "UnfreezeFungibleTokenAccount(string,string)"

accountSequence := ownerWalletAccount.GetSequence()
resultLog := types.NewResultLog(accountSequence, ctx.TxBytes())

return sdkTypes.Result{
    Events: types.MakeMxwEvents(eventSignature, owner.String(), eventParam),
    Log:    resultLog.String(),
}
```

Usage

This MakeMxwEvents create maxonrow events, by accepting :

* Custom Event Signature : using UnfreezeFungibleTokenAccount(string,string)
* Token owner
* Event Parameters as below:

| Name | Type | Description                 |
| ---- | ---- | --------------------------- |
| symbol | string | Token symbol, which must be unique| |
| owner | string | Token owner| |


### Querier

Now you can navigate to the ./x/token/fungible/querier.go file. 
This is the place to define which queries against application state users will be able to make. 
 
Here, you will see NewQuerier been defined, and it acts as a sub-router for queries to this module (similar the NewHandler function). Note that because there isn't an interface similar to Msg for queries, we need to manually define switch statement cases (they can't be pulled off of the query .Route() function):

```
func NewQuerier(cdc *codec.Codec, keeper *Keeper, feeKeeper *fee.Keeper) sdkTypes.Querier {
	return func(ctx sdkTypes.Context, path []string, req abci.RequestQuery) ([]byte, sdkTypes.Error) {
		switch path[0] {
		case QueryListTokenSymbol:
			return queryListTokenSymbol(cdc, ctx, path[1:], req, keeper)
		case QueryTokenData:
			return queryTokenData(cdc, ctx, path[1:], req, keeper)
		case QueryAccount:
			return queryAccount(cdc, ctx, path[1:], req, keeper)
		default:
			return nil, sdkTypes.ErrUnknownRequest("unknown token query endpoint")
		}
	}
}
```

This module will expose few queries:

### ListTokenSymbol
This query the available tokens list.

After the router is defined, define the inputs and responses for this queryListTokenSymbol:

```
func queryListTokenSymbol(cdc *codec.Codec, ctx sdkTypes.Context, _ []string, _ abci.RequestQuery, keeper *Keeper) ([]byte, sdkTypes.Error) {
	tokens := keeper.ListTokens(ctx)

	var symbols []string
	for _, t := range tokens {
		symbols = append(symbols, t.Symbol)
	}

	respData := cdc.MustMarshalJSON(symbols)

	return respData, nil
}
```

Notes on the above code:

The output type should be something that is both JSON marshalable and stringable (implements the Golang fmt.Stringer interface). The returned bytes should be the JSON encoding of the output result.

For the output of ListTokenSymbol, the normal ListTokenSymbol struct is already JSON marshalable, but we need to add a .String() method on it.


`Parameters`

| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| path | string | false | false    | Path to the data (eg. "/a/b/c") |
| data | []byte | false | true     | Data |
| height | int64 | 0 | false    | Height (0 means latest) |
| prove | bool | false | false    | Include proofs of the transactions inclusion in the block, if true |


`Example`

In this example, we will explain how to query token list with abci_query. 

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "abci_query",
    "params": [
    	"/custom/token/list_token_symbol",
    	"",
    	"0",
    	false
    	],
    "id": 0,
    "jsonrpc": "2.0"
}

```

The above command returns JSON structured like this: 
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": {
    "response": {
      "code": 0,
      "log": "",
      "info": "",
      "index": "0",
      "key": null,
      "value": "WyJUVCIsIlRULTEiLCJUVC0yIiwiVFQtMmIiLCJUVC00IiwiVFQtNSIsIlRULTYiLCJUb1QiXQ==",
      "proof": null,
      "height": "562",
      "codespace": ""
    }
  }
}
```

### TokenData
This takes a symbol and returns token data base on it.

After the router is defined, define the inputs and responses for this queryTokenData:

```
func queryTokenData(cdc *codec.Codec, ctx sdkTypes.Context, path []string, _ abci.RequestQuery, keeper *Keeper) ([]byte, sdkTypes.Error) {
	if len(path) != 1 {
		return nil, sdkTypes.ErrUnknownRequest(fmt.Sprintf("Invalid path %s", strings.Join(path, "/")))
	}

	symbol := path[0]

	tokenData, err := keeper.GetTokenData(ctx, symbol)
	if err != nil {
		return nil, err
	}

	tokenInfo := cdc.MustMarshalJSON(tokenData)

	return tokenInfo, nil
}
```

Notes on the above code:

This query request ONE path-parameter which refer to token-symbol. 
The output type should be something that is both JSON marshalable and stringable (implements the Golang fmt.Stringer interface). The returned bytes should be the JSON encoding of the output result.

For the output of TokenData, the normal TokenData struct is already JSON marshalable, but we need to add a .String() method on it.


`Parameters`

| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| path | string | false | false    | Path to the data (eg. "/a/b/c") |
| data | []byte | false | true     | Data |
| height | int64 | 0 | false    | Height (0 means latest) |
| prove | bool | false | false    | Include proofs of the transactions inclusion in the block, if true |


`Example`

In this example, we will explain how to query token data with abci_query. 

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "abci_query",
    "params": [
    	"/custom/token/token_data/TT-2b",
    	"",
    	"0",
    	false
    	],
    "id": 0,
    "jsonrpc": "2.0"
}

```

The above command returns JSON structured like this: 
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": {
    "response": {
      "code": 0,
      "log": "",
      "info": "",
      "index": "0",
      "key": null,
      "value": "eyJGbGFncyI6MzEsIk5hbWUiOiJUZXN0VG9rZW4tMmIiLCJTeW1ib2wiOiJUVC0yYiIsIkRlY2ltYWxzIjoiOCIsIk93bmVyIjoibXh3MXg1Y2Y4eTk5bnRqYzhjam0wMHo2MDN5ZnF3enh3Mm1hd2VtZjczIiwiTmV3T3duZXIiOiIiLCJNZXRhZGF0YSI6IiIsIlRvdGFsU3VwcGx5IjoiMTAwMDAwIiwiTWF4U3VwcGx5IjoiMTAwMDAwIn0=",
      "proof": null,
      "height": "639",
      "codespace": ""
    }
  }
}
```



### Account
This takes a account and symbol then returns account data.

After the router is defined, define the inputs and responses for this queryAccount:

```
func queryAccount(cdc *codec.Codec, ctx sdkTypes.Context, path []string, _ abci.RequestQuery, keeper *Keeper) ([]byte, sdkTypes.Error) {
	if len(path) != 2 {
		return nil, sdkTypes.ErrUnknownRequest(fmt.Sprintf("Invalid path %s", strings.Join(path, "/")))
	}

	symbol := path[0]
	accountBech := path[1]

	account, err := sdkTypes.AccAddressFromBech32(accountBech)
	if err != nil {
		return nil, sdkTypes.ErrInvalidAddress("Invalid account address")
	}

	acc, err := keeper.GetAccount(ctx, symbol, account)
	if err != nil {
		return nil, sdkTypes.ErrInternal(err.Error())
	}

	if acc == nil {
		return nil, nil
	}

	accountData := cdc.MustMarshalJSON(acc)

	return accountData, nil
}
```

Notes on the above code:

This query request TWO path-parameters which refer to token-symbol and account. 
The output type should be something that is both JSON marshalable and stringable (implements the Golang fmt.Stringer interface). The returned bytes should be the JSON encoding of the output result.

For the output of Account, the normal Account struct is already JSON marshalable, but we need to add a .String() method on it.

`Parameters`

| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| path | string | false | false    | Path to the data (eg. "/a/b/c") |
| data | []byte | false | true     | Data |
| height | int64 | 0 | false    | Height (0 means latest) |
| prove | bool | false | false    | Include proofs of the transactions inclusion in the block, if true |


`Example`

In this example, we will explain how to query account data with abci_query. 

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "abci_query",
    "params": [
    	"/custom/token/account/TT-2b/mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
    	"",
    	"0",
    	false
    	],
    "id": 0,
    "jsonrpc": "2.0"
}

```

The above command returns JSON structured like this: 
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": {
    "response": {
      "code": 0,
      "log": "",
      "info": "",
      "index": "0",
      "key": null,
      "value": "eyJPd25lciI6Im14dzF4NWNmOHk5OW50amM4Y2ptMDB6NjAzeWZxd3p4dzJtYXdlbWY3MyIsIkZyb3plbiI6ZmFsc2UsIk1ldGFkYXRhIjoiIiwiQmFsYW5jZSI6IjAifQ==",
      "proof": null,
      "height": "844",
      "codespace": ""
    }
  }
}
```


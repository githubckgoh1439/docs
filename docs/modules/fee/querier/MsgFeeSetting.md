This takes a fee setting and returns the fee setting info.

After the router is defined, define the inputs and responses for this queryMsgFeeSetting:

![Image-2](../pic/queryMsgFeeSetting.png)


Notes on the above code:

This query request ONE path-parameter which refer to token symbol. 
The output type should be something that is both JSON marshalable and stringable (implements the Golang fmt.Stringer interface). The returned bytes should be the JSON encoding of the output result.

For the output of MsgFeeSetting, the normal MsgFeeSetting struct is already JSON marshalable, but we need to add a .String() method on it.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| path | string | false | false    | Path to the data (eg. "/a/b/c") |
| data | []byte | false | true     | Data |
| height | int64 | 0 | false    | Height (0 means latest) |
| prove | bool | false | false    | Include proofs of the transactions inclusion in the block, if true |


#### Example
In this example, we will explain how to query MsgFeeSetting with abci_query. 

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "abci_query",
    "params": [
    	"/custom/fee/get_msg_fee_setting/zero",
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
            "value": "eyJuYW1lIjoiZGVmYXVsdCIsIm1pbiI6W3siZGVub20iOiJjaW4iLCJhbW91bnQiOiIxMDAwMDAwMDAifV0sIm1heCI6W3siZGVub20iOiJjaW4iLCJhbW91bnQiOiIxMDAwMDAwMDAwIn1dLCJwZXJjZW50YWdlIjoiMC4wMDEiLCJpc3N1ZXIiOiJteHcxazl0cjJjdWtoZnZsaGozNTZlNWV1cjI4a3V3M3A2YTRsOTNoNTkifQ==",
            "proof": null,
            "height": "345",
            "codespace": ""
        }
    }
}
```


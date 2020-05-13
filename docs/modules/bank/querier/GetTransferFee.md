This takes fee setting and amount then returns Transfer Fee.

After the router is defined, define the inputs and responses for this queryGetTransferFee:

![Image-2](../pic/queryGetTransferFee.png)


Notes on the above code:

This query request TWO path-parameters which refer to fee setting and amount. 
The output type should be something that is both JSON marshalable and stringable (implements the Golang fmt.Stringer interface). The returned bytes should be the JSON encoding of the output result.

For the output of Item, the normal Item struct is already JSON marshalable, but we need to add a .String() method on it.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| path | string | false | false    | Path to the data (eg. "/a/b/c") |
| data | []byte | false | true     | Data |
| height | int64 | 0 | false    | Height (0 means latest) |
| prove | bool | false | false    | Include proofs of the transactions inclusion in the block, if true |


#### Example
In this example, we will explain how to query GetTransferFee with abci_query. 

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "abci_query",
    "params": [
    	"/custom/bank/get_fee/double/1",
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
      "code": 1,
      "log": "{\"codespace\":\"sdk\",\"code\":1,\"message\":\"Invalid percentage: 0.05\"}",
      "info": "",
      "index": "0",
      "key": null,
      "value": null,
      "proof": null,
      "height": "64",
      "codespace": "sdk"
    }
  }
}

```



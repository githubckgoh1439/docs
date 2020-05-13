This returns all the names stored in the nameservice store which yet to processed.

After the router is defined, define the inputs and responses for this queryPendingAlias:

![Image-2](../pic/queryPendingAlias.png)


Notes on the above code:

This query request ONE path-parameter which refer to account address. 
The output type should be something that is both JSON marshalable and stringable (implements the Golang fmt.Stringer interface). The returned bytes should be the JSON encoding of the output result.

For the output of Token, the normal Token struct is already JSON marshalable, but we need to add a .String() method on it.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| path | string | false | false    | Path to the data (eg. "/a/b/c") |
| data | []byte | false | true     | Data |
| height | int64 | 0 | false    | Height (0 means latest) |
| prove | bool | false | false    | Include proofs of the transactions inclusion in the block, if true |


#### Example
In this example, we will explain how to query token data with abci_query. 

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "abci_query",
    "params": [
    	"/custom/nameservice/pending/mxw1x5cf8y99ntjc8cjm00z603yfqwzxw2mawemf73",
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
            "code": 4002,
            "log": "{\"codespace\":\"mxw\",\"code\":4002,\"message\":\"No such pending alias.\"}",
            "info": "",
            "index": "0",
            "key": null,
            "value": null,
            "proof": null,
            "height": "345",
            "codespace": "mxw"
        }
    }
}

```


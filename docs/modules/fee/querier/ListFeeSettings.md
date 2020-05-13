This returns all fee setting info.

After the router is defined, define the inputs and responses for this queryListFeeSettings:

![Image-2](../pic/queryListFeeSettings.png)


Notes on the above code:

This query request ZERO path-parameter. 
The output type should be something that is both JSON marshalable and stringable (implements the Golang fmt.Stringer interface). The returned bytes should be the JSON encoding of the output result.

For the output of ListFeeSettings, the normal ListFeeSettings struct is already JSON marshalable, but we need to add a .String() method on it.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| path | string | false | false    | Path to the data (eg. "/a/b/c") |
| data | []byte | false | true     | Data |
| height | int64 | 0 | false    | Height (0 means latest) |
| prove | bool | false | false    | Include proofs of the transactions inclusion in the block, if true |


#### Example
In this example, we will explain how to query of ListFeeSettings with abci_query. 

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "abci_query",
    "params": [
    	"/custom/fee/list_all_fee_settings",
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
            "value": "W3sibmFtZSI6ImRlZmF1bHQiLCJtaW4iOlt7ImRlbm9tIjoiY2luIiwiYW1vdW50IjoiMTAwMDAwMDAwIn1dLCJtYXgiOlt7ImRlbm9tIjoiY2luIiwiYW1vdW50IjoiMTAwMDAwMDAwMCJ9XSwicGVyY2VudGFnZSI6IjAuMDAxIiwiaXNzdWVyIjoibXh3MWs5dHIyY3VraGZ2bGhqMzU2ZTVldXIyOGt1dzNwNmE0bDkzaDU5In0seyJuYW1lIjoiZG91YmxlIiwibWluIjpbeyJkZW5vbSI6ImNpbiIsImFtb3VudCI6IjgwMDAwMDAwMCJ9XSwibWF4IjpbeyJkZW5vbSI6ImNpbiIsImFtb3VudCI6IjQwMDAwMDAwMDAifV0sInBlcmNlbnRhZ2UiOiIwLjAwMiIsImlzc3VlciI6Im14dzFrOXRyMmN1a2hmdmxoajM1NmU1ZXVyMjhrdXczcDZhNGw5M2g1OSJ9LHsibmFtZSI6ImRvdWJsZTEiLCJtaW4iOlt7ImRlbm9tIjoiY2luIiwiYW1vdW50IjoiODAwMDAwMDAwIn1dLCJtYXgiOlt7ImRlbm9tIjoiY2luIiwiYW1vdW50IjoiNDAwMDAwMDAwMCJ9XSwicGVyY2VudGFnZSI6IjAuMDAyIiwiaXNzdWVyIjoibXh3MWs5dHIyY3VraGZ2bGhqMzU2ZTVldXIyOGt1dzNwNmE0bDkzaDU5In0seyJuYW1lIjoiemVybyIsIm1pbiI6W3siZGVub20iOiJjaW4iLCJhbW91bnQiOiIwIn1dLCJtYXgiOlt7ImRlbm9tIjoiY2luIiwiYW1vdW50IjoiMCJ9XSwicGVyY2VudGFnZSI6IjAiLCJpc3N1ZXIiOiJteHcxazl0cjJjdWtoZnZsaGozNTZlNWV1cjI4a3V3M3A2YTRsOTNoNTkifV0=",
            "proof": null,
            "height": "345",
            "codespace": ""
        }
    }
}
```


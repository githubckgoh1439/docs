This takes a symbol and returns token data base on it.

After the router is defined, define the inputs and responses for this queryTokenData:

![Image-2](../pic/queryTokenData.png)


Notes on the above code:

This query request ONE path-parameter which refer to token-symbol. 
The output type should be something that is both JSON marshalable and stringable (implements the Golang fmt.Stringer interface). The returned bytes should be the JSON encoding of the output result.

For the output of TokenData, the normal TokenData struct is already JSON marshalable, but we need to add a .String() method on it.


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


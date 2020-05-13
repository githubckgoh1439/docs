This takes group-account address and returns the group-account info.

After the router is defined, define the inputs and responses for this queryMultiSigAcc:

![Image-2](../pic/queryMultiSigAcc.png)


Notes on the above code:

This query request ONE path-parameter which refer to group-account address. 
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
    	"/custom/auth/get_multisig_acc/mxw1jngujsjjwlzj4stjneyh3ecv6n35s8hrpdzru5",
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
            "value": "eyJ0eXBlIjoiY29zbW9zLXNkay9BY2NvdW50IiwidmFsdWUiOnsiYWRkcmVzcyI6Im14dzFqbmd1anNqandsemo0c3RqbmV5aDNlY3Y2bjM1czhocnBkenJ1NSIsImNvaW5zIjpbeyJkZW5vbSI6ImNpbiIsImFtb3VudCI6IjEwMDAwMDAwMDAwIn1dLCJwdWJsaWNfa2V5IjoiIiwiYWNjb3VudF9udW1iZXIiOiIyMDkzIiwic2VxdWVuY2UiOiIwIiwibXVsdGlzaWciOnsib3duZXIiOiJteHcxeWR2emFjeGowd3M5amFkeGttZHphbWM4OTdqbWxuNWRkOTBmemgiLCJ0aHJlc2hvbGQiOiIyIiwiY291bnRlciI6IjMiLCJzaWduZXJzIjpbIm14dzEyYTI1eXUwNmo0ZG00eTY0YW51NGpueGdnODZ4dTAzZnVwc3YwZCIsIm14dzF5ZHZ6YWN4ajB3czlqYWR4a21kemFtYzg5N2ptbG41ZGQ5MGZ6aCJdLCJwZW5kaW5nVHhzIjpudWxsfX19",
            "proof": null,
            "height": "125",
            "codespace": ""
        }
    }
}
```


# Node RPC
* 1 Connecting use your own local node
* 2 Protocols
* 3 Configuration
* 4 Arguments
    * 4.1 URI/HTTP
    * 4.2 JSONRPC/HTTP
* 5 RPC Endpoint List
* 6 APIs


### 1. Connecting use your own local node
This page assumes that you have your own node running locally, so examples here use localhost:26657 to represent using RPC commands on a local node.

### 2. Protocols
The following RPC protocols are supported:

* URI over HTTP
* JSONRPC over HTTP

### 3. Configuration
RPC can be configured by tuning parameters under [rpc] table in the $HOME/config/config.toml file or by using the --rpc.X command-line flags.

Default rpc listen address is tcp://0.0.0.0:26657. To set another address, set the laddr config parameter to desired value. CORS (Cross-Origin Resource Sharing) can be enabled by setting cors_allowed_origins, cors_allowed_methods, cors_allowed_headers config parameters.

### 4. Arguments
Arguments which expect strings or byte arrays may be passed as quoted strings, like "abc" or as 0x-prefixed strings, like 0x616263.

#### 4.1 URI/HTTP
```
curl 'localhost:26657/broadcast_tx_sync?tx=0xdb01f0625dee0a63ce6dc0430a14813e4939f1567b219704ffc2ad4df58bde010879122b383133453439333946313536374232313937303446464332414434444635384244453031303837392d34331a0d5a454252412d3136445f424e422002280130c0843d38904e400112700a26eb5ae9872102139bdd95de72c22ac2a2b0f87853b1cca2e8adf9c58a4a689c75d3263013441a1240598c3a74dc08d82d97668ed3523a105a2afb752c5be34d09fb5f3158d55db2f545a2466263cf80f02d1184dd50efc4d8a636a262909a632ebeddeaa426c092b218d2e518202a'
```
Response:
```
{
    "jsonrpc": "2.0",
    "id": "",
    "result": {
        "code": 0,
        "data": "7B226F726465725F6964223A22383133453439333946313536374232313937303446464332414434444635384244453031303837392D3433227D",
        "log": "Msg 0: ",
        "hash": "AB1B84C7C0B0B195941DCE9CFE1A54214B72D5DB54AD388D8B146A6B62911E8E"
    }
}
```

#### 4.2 JSONRPC/HTTP
JSONRPC requests can be POST'd to the root RPC endpoint via HTTP (e.g. http://localhost:26657/).
```
{
  "method": "broadcast_tx_sync",
  "jsonrpc": "2.0",
  "params": [
    "abc"
  ],
  "id": "xyz"
}
```

### 5. RPC Endpoint List
An HTTP Get request to the root RPC endpoint shows a list of available endpoints.
```
curl 'localhost:26657'
```
Response:

#### Available endpoints that don't require arguments:
```
/abci_info
/broadcast_evidence
/consensus_state
/dump_consensus_state
/genesis
/health
/net_info
/num_unconfirmed_txs
/status
/version
```

#### Endpoints that require arguments:
```
/abci_query?path=_&data=_&prove=_
/account?address=_
/block?height=_
/block_result?height=_
/blockchain?minHeight=_&maxHeight=_
/broadcast_tx_async?tx=_
/broadcast_tx_commit?tx=_
/broadcast_tx_sync?tx=_
/commit?height=_
/consensus_params?height=_
/decode_tx?tx=_
/encode_and_broadcast_tx_async?json=_
/encode_and_broadcast_tx_commit?json=_
/encode_and_broadcast_tx_sync?json=_
/encode_tx?json=_
/is_whitelisted?address=_
/query_fee?tx=_
/subscribe?query=_
/tx?hash=_&prove=_
/tx_search?query=_&prove=_&page=_&per_page=_
/unconfirmed_txs?limit=_
/unsubscribe?query=_
/unsubscribe_all?
/validator?address=_
/validators?height=_
```
### 6. APIs

#### 6.1 Query APIs

#### 6.1.1 Query ABCIInfo
Get some info about the application.

#### Return Type
```
type ResponseInfo struct {
    Data                 string
    Version              string
    AppVersion           uint64
    LastBlockHeight      int64
    LastBlockAppHash     []byte
}
```

#### Example
Run the command:
```
curl 'localhost:26657/abci_info'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "response": {
      "data": "maxonrow-go",
      "last_block_height": "2674",
      "last_block_app_hash": "f4pzDKuVaGzWk1hCCJseLteuE/PiDhNncSD+JqTpGw4="
    }
  }
}
```

#### 6.1.2 ABCIQuery
Query the application for some information.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| path | string | false | false    | Path to the data ("/a/b/c") |
| data | []byte | false | true     | Data |
| height | int64 | 0 | false    | Height (0 means latest) |
| prove | bool | false | false    | Include proofs of the transactions inclusion in the block, if true |


#### Available Query Path
- /store/acc/key
- /tokens/info
- /tokens/list
- /param/fees


#### Return Type
```
type ResponseQuery struct {
    Code                 uint32
    Log                  string
    Info                 string
    Index                int64
    Key                  []byte
    Value                []byte
    Proof                *merkle.Proof
    Height               int64
    Codespace            string
}
```

#### Example
In this example, we will explain how to query fee-setting info with abci_query. 

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "abci_query",
    "params": [
    	"custom/token/get_fee/default",
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
      "value": "W3siYW1vdW50IjoiMjAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0="
    }
  }
}
```

#### 6.1.3 Query Account
Get the account information on blockchain.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| address | string | false | true    | Bech32 OperatorAddress of account |

#### Return Type
```
type ResultAccount struct {
    Type  string
    Value *state.AccountInfoResponses
}
```

#### Example
Query the account number and sequence of the account-address

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "account",
    "params": ["mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "
    { 
   "type":"cosmos-sdk/Account",
   "value":{ 
      "address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
      "coins":[ 
         { 
            "denom":"cin",
            "amount":"1000000000000000000000"
         }
      ],
      "public_key":{ 
         "type":"tendermint/PubKeySecp256k1",
         "value":"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX"
      },
      "account_number":"8",
      "sequence":"0"
   }}"
}
```

#### 6.1.4 Query BlockResults
BlockResults gets ABCIResults at a given height. If no height is provided, it will fetch results for the latest block. Results are for the height of the block containing the txs. Thus response.results[5] is the results of executing getBlock(h).Txs[5]

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| height | int64 | false | false   | height to return. If no height is provided, it will fetch informations regarding the latest block. 0 means latest |



#### Return Type
```
type ResultBlockResults struct {
    Height  int64
    Results *state.ABCIResponses
}
```

#### Example
Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "block_results",
    "params": ["250"],
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
    "height": "250",
    "results": {
      "deliver_tx": [
        {
          "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
          "gasUsed": "78131",
          "events": [
            {
              "type": "message",
              "attributes": [
                {
                  "key": "YWN0aW9u",
                  "value": "c2VuZA=="
                }
              ]
            },
            {
              "type": "message",
              "attributes": [
                {
                  "key": "RnJvbQ==",
                  "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
                },
                {
                  "key": "VG8=",
                  "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
                },
                {
                  "key": "QW1vdW50",
                  "value": "MTE="
                }
              ]
            },
            {
              "type": "system",
              "attributes": [
                {
                  "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
                  "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
                }
              ]
            }
          ]
        }
      ],
      "end_block": {
        "validator_updates": null
      },
      "begin_block": {
        "events": [
          {
            "type": "transfer",
            "attributes": [
              {
                "key": "cmVjaXBpZW50",
                "value": "bXh3MWp2NjVzM2dycWY2djZqbDNkcDR0NmM5dDlyazk5Y2Q4dXk5MjNu"
              },
              {
                "key": "YW1vdW50"
              }
            ]
          },
          {
            "type": "message",
            "attributes": [
              {
                "key": "c2VuZGVy",
                "value": "bXh3MTd4cGZ2YWttMmFtZzk2MnlsczZmODR6M2tlbGw4YzVsdHp6a24z"
              }
            ]
          },
          {
            "type": "commission",
            "attributes": [
              {
                "key": "YW1vdW50"
              },
              {
                "key": "dmFsaWRhdG9y",
                "value": "bXh3dmFsb3BlcjF1cmF6ejZ2dWxkanAwOWd0emZ0MG53Mmt1djI1bDA3OWZoOTlnNA=="
              }
            ]
          },
          {
            "type": "rewards",
            "attributes": [
              {
                "key": "YW1vdW50"
              },
              {
                "key": "dmFsaWRhdG9y",
                "value": "bXh3dmFsb3BlcjF1cmF6ejZ2dWxkanAwOWd0emZ0MG53Mmt1djI1bDA3OWZoOTlnNA=="
              }
            ]
          }
        ]
      }
    }
  }
}
```
#### 6.1.5 Query Block
Get block at a given height. If no height is provided, it will fetch the latest block.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| height | int64 | false | false    | height of blockchain |

#### Return Type
```
type ResultBlock struct {
    BlockMeta *types.BlockMeta
    Block     *types.Block
}
// BlockMeta contains meta information about a block - namely, it's ID and Header.
type BlockMeta struct {
    BlockID BlockID
    Header  Header
}
// Block defines the atomic unit of a Tendermint blockchain.
type Block struct {
    mtx        sync.Mutex
    Header     `json:"header"`
    Data       `json:"data"`
    Evidence   EvidenceData
    LastCommit *Commit
}
```

#### Example
Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "block",
    "params": ["250"],
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
    "block_meta": {
      "block_id": {
        "hash": "1C615DE531EF5A1942E4615F78D9F35DE84364BA4BBAE69CD6079B102D4DBDFD",
        "parts": {
          "total": "1",
          "hash": "D74ED6766BF8636CFC3254CFC24555F1110A2B83909FBD8E7D91E43464EFC51D"
        }
      },
      "header": {
        "version": {
          "block": "10",
          "app": "0"
        },
        "chain_id": "maxonrow-chain",
        "height": "250",
        "time": "2019-10-09T04:02:16.19724Z",
        "num_txs": "1",
        "total_txs": "1",
        "last_block_id": {
          "hash": "5729518E8822DBAC0EE8CFAB8611DA7B8AE869E07291ECCB0BED9708771E2975",
          "parts": {
            "total": "1",
            "hash": "2AD3BA4B2EFE7DA324DBF1C32FC9068C03991293C5E04D6188988532F94E683A"
          }
        },
        "last_commit_hash": "EACEB08EAC29641F6D1E1FC839266CEDE8FEBDF6A79377EA85C840660A4C8719",
        "data_hash": "C5FFE7245DDDB3B80637863EFD86AB1E35F790DA9A38C703F84FF6760F448860",
        "validators_hash": "8FA5DDBB4588748676C7D0F4D61EDD06424CE9C6E16BFD6604E6B25807501EC7",
        "next_validators_hash": "8FA5DDBB4588748676C7D0F4D61EDD06424CE9C6E16BFD6604E6B25807501EC7",
        "consensus_hash": "048091BC7DDC283F77BFBF91D73C44DA58C3DF8A9CBC867405D8B7F3DAADA22F",
        "app_hash": "83CE21DF09121CA295A0B82BFD391CAAB61BF692B0C728FFF7DAFF1906463436",
        "last_results_hash": "",
        "evidence_hash": "",
        "proposer_address": "CE9CA702BE3125D62B062BB925A552882CBC08CA"
      }
    },
    "block": {
      "header": {
        "version": {
          "block": "10",
          "app": "0"
        },
        "chain_id": "maxonrow-chain",
        "height": "250",
        "time": "2019-10-09T04:02:16.19724Z",
        "num_txs": "1",
        "total_txs": "1",
        "last_block_id": {
          "hash": "5729518E8822DBAC0EE8CFAB8611DA7B8AE869E07291ECCB0BED9708771E2975",
          "parts": {
            "total": "1",
            "hash": "2AD3BA4B2EFE7DA324DBF1C32FC9068C03991293C5E04D6188988532F94E683A"
          }
        },
        "last_commit_hash": "EACEB08EAC29641F6D1E1FC839266CEDE8FEBDF6A79377EA85C840660A4C8719",
        "data_hash": "C5FFE7245DDDB3B80637863EFD86AB1E35F790DA9A38C703F84FF6760F448860",
        "validators_hash": "8FA5DDBB4588748676C7D0F4D61EDD06424CE9C6E16BFD6604E6B25807501EC7",
        "next_validators_hash": "8FA5DDBB4588748676C7D0F4D61EDD06424CE9C6E16BFD6604E6B25807501EC7",
        "consensus_hash": "048091BC7DDC283F77BFBF91D73C44DA58C3DF8A9CBC867405D8B7F3DAADA22F",
        "app_hash": "83CE21DF09121CA295A0B82BFD391CAAB61BF692B0C728FFF7DAFF1906463436",
        "last_results_hash": "",
        "evidence_hash": "",
        "proposer_address": "CE9CA702BE3125D62B062BB925A552882CBC08CA"
      },
      "data": {
        "txs": [
          "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04"
        ]
      },
      "evidence": {
        "evidence": null
      },
      "last_commit": {
        "block_id": {
          "hash": "5729518E8822DBAC0EE8CFAB8611DA7B8AE869E07291ECCB0BED9708771E2975",
          "parts": {
            "total": "1",
            "hash": "2AD3BA4B2EFE7DA324DBF1C32FC9068C03991293C5E04D6188988532F94E683A"
          }
        },
        "precommits": [
          {
            "type": 2,
            "height": "249",
            "round": "0",
            "block_id": {
              "hash": "5729518E8822DBAC0EE8CFAB8611DA7B8AE869E07291ECCB0BED9708771E2975",
              "parts": {
                "total": "1",
                "hash": "2AD3BA4B2EFE7DA324DBF1C32FC9068C03991293C5E04D6188988532F94E683A"
              }
            },
            "timestamp": "2019-10-09T04:02:16.19724Z",
            "validator_address": "CE9CA702BE3125D62B062BB925A552882CBC08CA",
            "validator_index": "0",
            "signature": "6Lc2zEJ5+mIesdr1LFCyV1lFtuePRUswSkxMzJCI57cku1iLqVKZo4SrKfqPiFUP3sI6EOD30nu0QfMXpZv8BA=="
          }
        ]
      }
    }
  }
}
```

#### 6.1.6 Query BlockchainInfo
Get block headers for minHeight <= height <= maxHeight. Block headers are returned in descending order (highest first).

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| minHeight | int64 | false | true   | Minimum block height of blockchain |
| maxHeight | int64 | false | true   | Maximum block height of blockchain |

#### Return Type
List of blocks
```
type ResultBlockchainInfo struct {
    LastHeight int64
    BlockMetas []*types.BlockMeta
}
```

#### Example
Run the command:
```
curl 'http://localhost:26657/blockchain?minHeight=250&maxHeight=252'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "last_height": "3027",
    "block_metas": [
      {
        "block_id": {
          "hash": "4E6A262CE71B6104FE6B2033672617DB824C6F9CB0F3758B629B8D2D1FED5105",
          "parts": {
            "total": "1",
            "hash": "C1B7A9105EEE1E6120021DBAF1EE25EF4EA05235EC7758DEDEA9E8390A41FF1A"
          }
        },
        "header": {
          "version": {
            "block": "10",
            "app": "0"
          },
          "chain_id": "maxonrow-chain",
          "height": "252",
          "time": "2019-10-09T04:02:26.360922Z",
          "num_txs": "0",
          "total_txs": "1",
          "last_block_id": {
            "hash": "275734B220949E522948D2D9D6E6A39899044CA141BFE03B993F643227EE7FF8",
            "parts": {
              "total": "1",
              "hash": "495A182D6B984EE937C02DDBD0A0A07C197022670B37CBADDA266C1BD10E3DED"
            }
          },
          "last_commit_hash": "AD8DAAEC538BEE9A5F1488AE415B333A9A2607643081974BE8CBEE7A442BBF5B",
          "data_hash": "",
          "validators_hash": "8FA5DDBB4588748676C7D0F4D61EDD06424CE9C6E16BFD6604E6B25807501EC7",
          "next_validators_hash": "8FA5DDBB4588748676C7D0F4D61EDD06424CE9C6E16BFD6604E6B25807501EC7",
          "consensus_hash": "048091BC7DDC283F77BFBF91D73C44DA58C3DF8A9CBC867405D8B7F3DAADA22F",
          "app_hash": "980D7CF4FF5920212329B4D0AE319565F70840AB225B65502790F04660F23CA9",
          "last_results_hash": "",
          "evidence_hash": "",
          "proposer_address": "CE9CA702BE3125D62B062BB925A552882CBC08CA"
        }
      },
      {
        "block_id": {
          "hash": "275734B220949E522948D2D9D6E6A39899044CA141BFE03B993F643227EE7FF8",
          "parts": {
            "total": "1",
            "hash": "495A182D6B984EE937C02DDBD0A0A07C197022670B37CBADDA266C1BD10E3DED"
          }
        },
        "header": {
          "version": {
            "block": "10",
            "app": "0"
          },
          "chain_id": "maxonrow-chain",
          "height": "251",
          "time": "2019-10-09T04:02:21.276513Z",
          "num_txs": "0",
          "total_txs": "1",
          "last_block_id": {
            "hash": "1C615DE531EF5A1942E4615F78D9F35DE84364BA4BBAE69CD6079B102D4DBDFD",
            "parts": {
              "total": "1",
              "hash": "D74ED6766BF8636CFC3254CFC24555F1110A2B83909FBD8E7D91E43464EFC51D"
            }
          },
          "last_commit_hash": "AC9ABF612D35EE807BF1FA0A950883B7830456138D045420FDACC20AD8C701CD",
          "data_hash": "",
          "validators_hash": "8FA5DDBB4588748676C7D0F4D61EDD06424CE9C6E16BFD6604E6B25807501EC7",
          "next_validators_hash": "8FA5DDBB4588748676C7D0F4D61EDD06424CE9C6E16BFD6604E6B25807501EC7",
          "consensus_hash": "048091BC7DDC283F77BFBF91D73C44DA58C3DF8A9CBC867405D8B7F3DAADA22F",
          "app_hash": "D1D69AE358EEFC277B99C504E92FB7D777C49AFF45A7283CB88D34089CA07CB6",
          "last_results_hash": "6E340B9CFFB37A989CA544E6BB780A2C78901D3FB33738768511A30617AFA01D",
          "evidence_hash": "",
          "proposer_address": "CE9CA702BE3125D62B062BB925A552882CBC08CA"
        }
      },
      {
        "block_id": {
          "hash": "1C615DE531EF5A1942E4615F78D9F35DE84364BA4BBAE69CD6079B102D4DBDFD",
          "parts": {
            "total": "1",
            "hash": "D74ED6766BF8636CFC3254CFC24555F1110A2B83909FBD8E7D91E43464EFC51D"
          }
        },
        "header": {
          "version": {
            "block": "10",
            "app": "0"
          },
          "chain_id": "maxonrow-chain",
          "height": "250",
          "time": "2019-10-09T04:02:16.19724Z",
          "num_txs": "1",
          "total_txs": "1",
          "last_block_id": {
            "hash": "5729518E8822DBAC0EE8CFAB8611DA7B8AE869E07291ECCB0BED9708771E2975",
            "parts": {
              "total": "1",
              "hash": "2AD3BA4B2EFE7DA324DBF1C32FC9068C03991293C5E04D6188988532F94E683A"
            }
          },
          "last_commit_hash": "EACEB08EAC29641F6D1E1FC839266CEDE8FEBDF6A79377EA85C840660A4C8719",
          "data_hash": "C5FFE7245DDDB3B80637863EFD86AB1E35F790DA9A38C703F84FF6760F448860",
          "validators_hash": "8FA5DDBB4588748676C7D0F4D61EDD06424CE9C6E16BFD6604E6B25807501EC7",
          "next_validators_hash": "8FA5DDBB4588748676C7D0F4D61EDD06424CE9C6E16BFD6604E6B25807501EC7",
          "consensus_hash": "048091BC7DDC283F77BFBF91D73C44DA58C3DF8A9CBC867405D8B7F3DAADA22F",
          "app_hash": "83CE21DF09121CA295A0B82BFD391CAAB61BF692B0C728FFF7DAFF1906463436",
          "last_results_hash": "",
          "evidence_hash": "",
          "proposer_address": "CE9CA702BE3125D62B062BB925A552882CBC08CA"
        }
      }
    ]
  }
}
```

#### 6.1.7 Query BroadcastEvidence
Broadcast evidence of the misbehavior.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| evidence | string | false | true    | Amino-encoded JSON evidence |

#### Return Type
```
type ResultBroadcastEvidence struct {
    Results *state.Responses
}
```

#### Example

Run the command:
```
curl 'http://localhost:26657/broadcast_evidence'
```

The above command returns JSON structured like this:
```
{
  "error": "",
  "result": "",
  "id": "",
  "jsonrpc": "2.0"
}
```
#### 6.1.8 BroadcastTxAsync
This method just return transaction hash right away and there is no return from CheckTx or DeliverTx.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| tx | Tx | nil | true  | The transaction info bytes in hex |



#### Return Type
CheckTx Result
```
type ResultBroadcastTx struct {
    Code uint32
    Data cmn.HexBytes
    Log  string
    Hash cmn.HexBytes
}
```

#### Example

Step-1. Query the account number and sequence of the TWO account-address 

Run the command with the JSON request body:
```
http://localhost:26657/
```

```
{
    "method": "account",
    "params": ["mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "
    { 
   "type":"cosmos-sdk/Account",
   "value":{ 
      "address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
      "coins":[ 
         { 
            "denom":"cin",
            "amount":"1000000000000000000000"
         }
      ],
      "public_key":{ 
         "type":"tendermint/PubKeySecp256k1",
         "value":"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX"
      },
      "account_number":"8",
      "sequence":"0"
   }}"
}
```

Run the command with the JSON request body:
```
http://localhost:26657/
```

```
{
    "method": "account",
    "params": ["mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "
    { 
   "type":"cosmos-sdk/Account",
   "value":{ 
      "address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e",
      "coins":[ 
         { 
            "denom":"cin",
            "amount":"1000000000000000000000"
         }
      ],
      "public_key":{ 
         "type":"tendermint/PubKeySecp256k1",
         "value":"A9Nmqbf2cKAOs9hzGTRyS2Wd6uLBWrwkZ0S7q06N0/3i"
      },
      "account_number":"7",
      "sequence":"0"
   }}"
}
```


Step-2. Generate the transaction output as json format

```
mxwcli tx send mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e 11cin --fees 50000000000000000cin --gas 0 --memo "testing transfer with 11cin, from acc-9 to acc-8" --home ~/.mxw_20191009_local01 --chain-id=maxonrow-chain
```

```
{ 
   "chain_id":"maxonrow-chain",
   "account_number":"8",
   "sequence":"0",
   "fee":{ 
      "amount":[ 
         { 
            "denom":"cin",
            "amount":"50000000000000000"
         }
      ],
      "gas":"0"
   },
   "msgs":[ 
      { 
         "type":"mxw/msgSend",
         "value":{ 
            "from_address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
            "to_address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e",
            "amount":[ 
               { 
                  "denom":"cin",
                  "amount":"11"
               }
            ]
         }
      }
   ],
   "memo":"testing transfer with 11cin, from acc-9 to acc-8"
}
```

```
confirm transaction before signing and broadcasting [y/N]: y
Password to sign with 'acc-9':
```

```
height: 0
txhash: 79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9
code: 0
data: ""
rawlog: '[{"msg_index":0,"success":true,"log":""}]'
logs:
- msgindex: 0
  success: true
  log: ""
info: ""
gaswanted: 0
gasused: 0
events: []
codespace: ""
tx: null
timestamp: ""
```

Step-3. You could decode using the decode_tx, base on the value of txhash 

Run the command:
```
curl 'http://localhost:26657/decoded_tx?hash=0x79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9&prove=true'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250",
    "index": 0,
    "tx_result": {
      "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
      "gasUsed": "78131",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "YWN0aW9u",
              "value": "c2VuZA=="
            }
          ]
        },
        {
          "type": "message",
          "attributes": [
            {
              "key": "RnJvbQ==",
              "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
            },
            {
              "key": "VG8=",
              "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
            },
            {
              "key": "QW1vdW50",
              "value": "MTE="
            }
          ]
        },
        {
          "type": "system",
          "attributes": [
            {
              "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
              "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
            }
          ]
        }
      ]
    },
    "tx": "{\"type\":\"cosmos-sdk/StdTx\",\"value\":{\"fee\":{\"amount\":[{\"amount\":\"50000000000000000\",\"denom\":\"cin\"}],\"gas\":\"0\"},\"memo\":\"testing transfer with 11cin, from acc-9 to acc-8\",\"msg\":[{\"type\":\"mxw/msgSend\",\"value\":{\"amount\":[{\"amount\":\"11\",\"denom\":\"cin\"}],\"from_address\":\"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p\",\"to_address\":\"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e\"}}],\"signatures\":[{\"pub_key\":{\"type\":\"tendermint/PubKeySecp256k1\",\"value\":\"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX\"},\"signature\":\"1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4g==\"}]}}",
    "proof": {
      "RootHash": "C5FFE7245DDDB3B80637863EFD86AB1E35F790DA9A38C703F84FF6760F448860",
      "Data": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04",
      "Proof": {
        "total": "1",
        "index": "0",
        "leaf_hash": "xf/nJF3ds7gGN4Y+/YarHjX3kNqaOMcD+E/2dg9EiGA=",
        "aunts": []
      }
    }
  }
}
```

Step-4. You proceed below : 

4.1 Do the filtering by remove symbol of '\', base on the value of tx like below
```
{ 
   "type":"cosmos-sdk/StdTx",
   "value":{ 
      "fee":{ 
         "amount":[ 
            { 
               "amount":"50000000000000000",
               "denom":"cin"
            }
         ],
         "gas":"0"
      },
      "memo":"testing transfer with 11cin, from acc-9 to acc-8",
      "msg":[ 
         { 
            "type":"mxw/msgSend",
            "value":{ 
               "amount":[ 
                  { 
                     "amount":"11",
                     "denom":"cin"
                  }
               ],
               "from_address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
               "to_address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e"
            }
         }
      ],
      "signatures":[ 
         { 
            "pub_key":{ 
               "type":"tendermint/PubKeySecp256k1",
               "value":"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX"
            },
            "signature":"1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4g=="
         }
      ]
   }
}
```

4.2 Then convert the value into base-64 format
```
eyJ0eXBlIjoiY29zbW9zLXNkay9TdGRUeCIsInZhbHVlIjp7ImZlZSI6eyJhbW91bnQiOlt7ImFtb3VudCI6IjUwMDAwMDAwMDAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0sImdhcyI6IjAifSwibWVtbyI6InRlc3RpbmcgdHJhbnNmZXIgd2l0aCAxMWNpbiwgZnJvbSBhY2MtOSB0byBhY2MtOCIsIm1zZyI6W3sidHlwZSI6Im14dy9tc2dTZW5kIiwidmFsdWUiOnsiYW1vdW50IjpbeyJhbW91bnQiOiIxMSIsImRlbm9tIjoiY2luIn1dLCJmcm9tX2FkZHJlc3MiOiJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJ0b19hZGRyZXNzIjoibXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRlIn19XSwic2lnbmF0dXJlcyI6W3sicHViX2tleSI6eyJ0eXBlIjoidGVuZGVybWludC9QdWJLZXlTZWNwMjU2azEiLCJ2YWx1ZSI6IkE1SWE4RUFvTllZbFhVNXk0aHV6c3czRm1FSXZLZ3pxd2lDOXhqZjFWekRYIn0sInNpZ25hdHVyZSI6IjFIZHZodkFNTkJGVlpYbTRMam1DMkJFeTQxSC9HQnFjUHJsbnZFRTVYT1VRZVNVTXJuZkd1NkhmeGhqOHNlKzNlZUZaTXVQSFZaZzFEdm9iV3pBRTRnPT0ifV19fQ==
```

Step-5. You could encode using the encode_tx, base on the value of base-64 as above.

Run the command with the JSON request body:
```
http://localhost:26657/
```

```
{
    "method": "encode_tx",
    "params": ["eyJ0eXBlIjoiY29zbW9zLXNkay9TdGRUeCIsInZhbHVlIjp7ImZlZSI6eyJhbW91bnQiOlt7ImFtb3VudCI6IjUwMDAwMDAwMDAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0sImdhcyI6IjAifSwibWVtbyI6InRlc3RpbmcgdHJhbnNmZXIgd2l0aCAxMWNpbiwgZnJvbSBhY2MtOSB0byBhY2MtOCIsIm1zZyI6W3sidHlwZSI6Im14dy9tc2dTZW5kIiwidmFsdWUiOnsiYW1vdW50IjpbeyJhbW91bnQiOiIxMSIsImRlbm9tIjoiY2luIn1dLCJmcm9tX2FkZHJlc3MiOiJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJ0b19hZGRyZXNzIjoibXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRlIn19XSwic2lnbmF0dXJlcyI6W3sicHViX2tleSI6eyJ0eXBlIjoidGVuZGVybWludC9QdWJLZXlTZWNwMjU2azEiLCJ2YWx1ZSI6IkE1SWE4RUFvTllZbFhVNXk0aHV6c3czRm1FSXZLZ3pxd2lDOXhqZjFWekRYIn0sInNpZ25hdHVyZSI6IjFIZHZodkFNTkJGVlpYbTRMam1DMkJFeTQxSC9HQnFjUHJsbnZFRTVYT1VRZVNVTXJuZkd1NkhmeGhqOHNlKzNlZUZaTXVQSFZaZzFEdm9iV3pBRTRnPT0ifV19fQ=="],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04"
}
```

Step-6. You could submit the transaction using the BroadcastTxAsync, base on the result value.

Run the command with the JSON request body:
```
http://localhost:26657/
```

```
{
    "method": "broadcast_tx_async",
    "params": ["/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04"],
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
    "code": 0,
    "data": "",
    "log": "",
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9"
  }
}
```

Step-7. You can verify the current transaction using the Tx, base on the hash value.

Run the command:
```
curl 'http://localhost:26657/tx?hash=0x79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250",
    "index": 0,
    "tx_result": {
      "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
      "gasUsed": "78131",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "YWN0aW9u",
              "value": "c2VuZA=="
            }
          ]
        },
        {
          "type": "message",
          "attributes": [
            {
              "key": "RnJvbQ==",
              "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
            },
            {
              "key": "VG8=",
              "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
            },
            {
              "key": "QW1vdW50",
              "value": "MTE="
            }
          ]
        },
        {
          "type": "system",
          "attributes": [
            {
              "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
              "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
            }
          ]
        }
      ]
    },
    "tx": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04"
  }
}
```
#### 6.1.9 BroadcastTxCommit
The transaction will be broadcasted and returns with the response from CheckTx and DeliverTx.
SThis method will wait for both CheckTx and DeliverTx, so it is the slowest way to broadcast through RPC but offers the most accurate success/failure response.

CONTRACT: only returns error if mempool.CheckTx() errs or if we timeout waiting for tx to commit.

If CheckTx or DeliverTx fail, no error will be returned, but the returned result will contain a non-OK ABCI code.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| tx | Tx | nil | true  | The transaction info bytes in hex |

#### Return Type
CheckTx and DeliverTx results
```
type ResultBroadcastTxCommit struct {
    CheckTx   abci.ResponseCheckTx
    DeliverTx abci.ResponseDeliverTx
    Hash      cmn.HexBytes
    Height    int64
}
```

#### Example

Step-1. Query the account number and sequence of the TWO account-address 

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "account",
    "params": ["mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "
    { 
   "type":"cosmos-sdk/Account",
   "value":{ 
      "address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
      "coins":[ 
         { 
            "denom":"cin",
            "amount":"1000000000000000000000"
         }
      ],
      "public_key":{ 
         "type":"tendermint/PubKeySecp256k1",
         "value":"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX"
      },
      "account_number":"8",
      "sequence":"0"
   }}"
}
```

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "account",
    "params": ["mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "
    { 
   "type":"cosmos-sdk/Account",
   "value":{ 
      "address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e",
      "coins":[ 
         { 
            "denom":"cin",
            "amount":"1000000000000000000000"
         }
      ],
      "public_key":{ 
         "type":"tendermint/PubKeySecp256k1",
         "value":"A9Nmqbf2cKAOs9hzGTRyS2Wd6uLBWrwkZ0S7q06N0/3i"
      },
      "account_number":"7",
      "sequence":"0"
   }}"
}
```


Step-2. Generate the transaction output as json format

```
mxwcli tx send mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e 11cin --fees 50000000000000000cin --gas 0 --memo "testing transfer with 11cin, from acc-9 to acc-8" --home ~/.mxw_20191009_local01 --chain-id=maxonrow-chain
```

```
{ 
   "chain_id":"maxonrow-chain",
   "account_number":"8",
   "sequence":"0",
   "fee":{ 
      "amount":[ 
         { 
            "denom":"cin",
            "amount":"50000000000000000"
         }
      ],
      "gas":"0"
   },
   "msgs":[ 
      { 
         "type":"mxw/msgSend",
         "value":{ 
            "from_address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
            "to_address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e",
            "amount":[ 
               { 
                  "denom":"cin",
                  "amount":"11"
               }
            ]
         }
      }
   ],
   "memo":"testing transfer with 11cin, from acc-9 to acc-8"
}
```

```
confirm transaction before signing and broadcasting [y/N]: y
Password to sign with 'acc-9':
```

```
height: 0
txhash: 79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9
code: 0
data: ""
rawlog: '[{"msg_index":0,"success":true,"log":""}]'
logs:
- msgindex: 0
  success: true
  log: ""
info: ""
gaswanted: 0
gasused: 0
events: []
codespace: ""
tx: null
timestamp: ""
```

Step-3. You could decode using the decode_tx, base on the value of txhash 

Run the command:
```
curl 'http://localhost:26657/decoded_tx?hash=0x79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9&prove=true'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250",
    "index": 0,
    "tx_result": {
      "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
      "gasUsed": "78131",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "YWN0aW9u",
              "value": "c2VuZA=="
            }
          ]
        },
        {
          "type": "message",
          "attributes": [
            {
              "key": "RnJvbQ==",
              "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
            },
            {
              "key": "VG8=",
              "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
            },
            {
              "key": "QW1vdW50",
              "value": "MTE="
            }
          ]
        },
        {
          "type": "system",
          "attributes": [
            {
              "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
              "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
            }
          ]
        }
      ]
    },
    "tx": "{\"type\":\"cosmos-sdk/StdTx\",\"value\":{\"fee\":{\"amount\":[{\"amount\":\"50000000000000000\",\"denom\":\"cin\"}],\"gas\":\"0\"},\"memo\":\"testing transfer with 11cin, from acc-9 to acc-8\",\"msg\":[{\"type\":\"mxw/msgSend\",\"value\":{\"amount\":[{\"amount\":\"11\",\"denom\":\"cin\"}],\"from_address\":\"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p\",\"to_address\":\"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e\"}}],\"signatures\":[{\"pub_key\":{\"type\":\"tendermint/PubKeySecp256k1\",\"value\":\"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX\"},\"signature\":\"1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4g==\"}]}}",
    "proof": {
      "RootHash": "C5FFE7245DDDB3B80637863EFD86AB1E35F790DA9A38C703F84FF6760F448860",
      "Data": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04",
      "Proof": {
        "total": "1",
        "index": "0",
        "leaf_hash": "xf/nJF3ds7gGN4Y+/YarHjX3kNqaOMcD+E/2dg9EiGA=",
        "aunts": []
      }
    }
  }
}
```

Step-4. You proceed below : 

4.1 Do the filtering by remove symbol of '\', base on the value of tx like below
```
{ 
   "type":"cosmos-sdk/StdTx",
   "value":{ 
      "fee":{ 
         "amount":[ 
            { 
               "amount":"50000000000000000",
               "denom":"cin"
            }
         ],
         "gas":"0"
      },
      "memo":"testing transfer with 11cin, from acc-9 to acc-8",
      "msg":[ 
         { 
            "type":"mxw/msgSend",
            "value":{ 
               "amount":[ 
                  { 
                     "amount":"11",
                     "denom":"cin"
                  }
               ],
               "from_address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
               "to_address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e"
            }
         }
      ],
      "signatures":[ 
         { 
            "pub_key":{ 
               "type":"tendermint/PubKeySecp256k1",
               "value":"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX"
            },
            "signature":"1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4g=="
         }
      ]
   }
}
```

4.2 Then convert the value into base-64 format
```
eyJ0eXBlIjoiY29zbW9zLXNkay9TdGRUeCIsInZhbHVlIjp7ImZlZSI6eyJhbW91bnQiOlt7ImFtb3VudCI6IjUwMDAwMDAwMDAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0sImdhcyI6IjAifSwibWVtbyI6InRlc3RpbmcgdHJhbnNmZXIgd2l0aCAxMWNpbiwgZnJvbSBhY2MtOSB0byBhY2MtOCIsIm1zZyI6W3sidHlwZSI6Im14dy9tc2dTZW5kIiwidmFsdWUiOnsiYW1vdW50IjpbeyJhbW91bnQiOiIxMSIsImRlbm9tIjoiY2luIn1dLCJmcm9tX2FkZHJlc3MiOiJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJ0b19hZGRyZXNzIjoibXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRlIn19XSwic2lnbmF0dXJlcyI6W3sicHViX2tleSI6eyJ0eXBlIjoidGVuZGVybWludC9QdWJLZXlTZWNwMjU2azEiLCJ2YWx1ZSI6IkE1SWE4RUFvTllZbFhVNXk0aHV6c3czRm1FSXZLZ3pxd2lDOXhqZjFWekRYIn0sInNpZ25hdHVyZSI6IjFIZHZodkFNTkJGVlpYbTRMam1DMkJFeTQxSC9HQnFjUHJsbnZFRTVYT1VRZVNVTXJuZkd1NkhmeGhqOHNlKzNlZUZaTXVQSFZaZzFEdm9iV3pBRTRnPT0ifV19fQ==
```

Step-5. You could encode using the encode_tx, base on the value of base-64 as above.

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "encode_tx",
    "params": ["eyJ0eXBlIjoiY29zbW9zLXNkay9TdGRUeCIsInZhbHVlIjp7ImZlZSI6eyJhbW91bnQiOlt7ImFtb3VudCI6IjUwMDAwMDAwMDAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0sImdhcyI6IjAifSwibWVtbyI6InRlc3RpbmcgdHJhbnNmZXIgd2l0aCAxMWNpbiwgZnJvbSBhY2MtOSB0byBhY2MtOCIsIm1zZyI6W3sidHlwZSI6Im14dy9tc2dTZW5kIiwidmFsdWUiOnsiYW1vdW50IjpbeyJhbW91bnQiOiIxMSIsImRlbm9tIjoiY2luIn1dLCJmcm9tX2FkZHJlc3MiOiJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJ0b19hZGRyZXNzIjoibXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRlIn19XSwic2lnbmF0dXJlcyI6W3sicHViX2tleSI6eyJ0eXBlIjoidGVuZGVybWludC9QdWJLZXlTZWNwMjU2azEiLCJ2YWx1ZSI6IkE1SWE4RUFvTllZbFhVNXk0aHV6c3czRm1FSXZLZ3pxd2lDOXhqZjFWekRYIn0sInNpZ25hdHVyZSI6IjFIZHZodkFNTkJGVlpYbTRMam1DMkJFeTQxSC9HQnFjUHJsbnZFRTVYT1VRZVNVTXJuZkd1NkhmeGhqOHNlKzNlZUZaTXVQSFZaZzFEdm9iV3pBRTRnPT0ifV19fQ=="],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04"
}
```

Step-6. You could submit the transaction using the BroadcastTxCommit, base on the result value.

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "broadcast_tx_commit",
    "params": ["/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this. Please note that the returned data contains both CheckTx and DeliverTx output.

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": {
    "check_tx": {
      "code": 0,
      "log": "",
      "gasUsed": "28083",
      "events": []
    },
    "deliver_tx": {
         "code": 0,
         "log": "",
         "gasUsed": "28083",
         "events": []
    },
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250"
  }
}

```

Step-7. You can verify the current transaction using the Tx, base on the hash value.

Run the command:
```
curl 'http://localhost:26657/tx?hash=0x79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250",
    "index": 0,
    "tx_result": {
      "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
      "gasUsed": "78131",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "YWN0aW9u",
              "value": "c2VuZA=="
            }
          ]
        },
        {
          "type": "message",
          "attributes": [
            {
              "key": "RnJvbQ==",
              "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
            },
            {
              "key": "VG8=",
              "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
            },
            {
              "key": "QW1vdW50",
              "value": "MTE="
            }
          ]
        },
        {
          "type": "system",
          "attributes": [
            {
              "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
              "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
            }
          ]
        }
      ]
    },
    "tx": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04"
  }
}
```
#### 6.1.10 Query Commit
Get block commit at a given height. If no height is provided, it will fetch the commit for the latest block.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| Height | int64 | false | true  | height of blockchain |


#### Return Type
```
type ResultCommit struct {
    types.SignedHeader
    CanonicalCommit    bool
}
```

#### Example

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "commit",
    "params": ["250"],
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
    "signed_header": {
      "header": {
        "version": {
          "block": "10",
          "app": "0"
        },
        "chain_id": "maxonrow-chain",
        "height": "250",
        "time": "2019-10-09T04:02:16.19724Z",
        "num_txs": "1",
        "total_txs": "1",
        "last_block_id": {
          "hash": "5729518E8822DBAC0EE8CFAB8611DA7B8AE869E07291ECCB0BED9708771E2975",
          "parts": {
            "total": "1",
            "hash": "2AD3BA4B2EFE7DA324DBF1C32FC9068C03991293C5E04D6188988532F94E683A"
          }
        },
        "last_commit_hash": "EACEB08EAC29641F6D1E1FC839266CEDE8FEBDF6A79377EA85C840660A4C8719",
        "data_hash": "C5FFE7245DDDB3B80637863EFD86AB1E35F790DA9A38C703F84FF6760F448860",
        "validators_hash": "8FA5DDBB4588748676C7D0F4D61EDD06424CE9C6E16BFD6604E6B25807501EC7",
        "next_validators_hash": "8FA5DDBB4588748676C7D0F4D61EDD06424CE9C6E16BFD6604E6B25807501EC7",
        "consensus_hash": "048091BC7DDC283F77BFBF91D73C44DA58C3DF8A9CBC867405D8B7F3DAADA22F",
        "app_hash": "83CE21DF09121CA295A0B82BFD391CAAB61BF692B0C728FFF7DAFF1906463436",
        "last_results_hash": "",
        "evidence_hash": "",
        "proposer_address": "CE9CA702BE3125D62B062BB925A552882CBC08CA"
      },
      "commit": {
        "block_id": {
          "hash": "1C615DE531EF5A1942E4615F78D9F35DE84364BA4BBAE69CD6079B102D4DBDFD",
          "parts": {
            "total": "1",
            "hash": "D74ED6766BF8636CFC3254CFC24555F1110A2B83909FBD8E7D91E43464EFC51D"
          }
        },
        "precommits": [
          {
            "type": 2,
            "height": "250",
            "round": "0",
            "block_id": {
              "hash": "1C615DE531EF5A1942E4615F78D9F35DE84364BA4BBAE69CD6079B102D4DBDFD",
              "parts": {
                "total": "1",
                "hash": "D74ED6766BF8636CFC3254CFC24555F1110A2B83909FBD8E7D91E43464EFC51D"
              }
            },
            "timestamp": "2019-10-09T04:02:21.276513Z",
            "validator_address": "CE9CA702BE3125D62B062BB925A552882CBC08CA",
            "validator_index": "0",
            "signature": "WLwe3JutiuSwM/BCbRh1ZbMVCF8y8OvRemCGC1hNx8WwxyQckwq5nAWaTdwNNVHIOmYTQOk6CzycFQ6y3RAjBg=="
          }
        ]
      }
    },
    "canonical": true
  }
}
```

#### 6.1.11 Consensus Parameters
Get consensus parameters.


#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| height | int | 0 | false    | height to return. If no height is provided, it will fetch commit informations regarding the latest block. 0 means latest |


#### Return Type
Consensus parameters results.
```
type ResultConsensusParameters struct {
    Results *state.Responses
}
```

#### Example

Run the command:
```
curl 'http://localhost:26657/consensus_params?height=250'
```

The above command returns JSON structured like this: 
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "block_height": "250",
    "consensus_params": {
      "block": {
        "max_bytes": "22020096",
        "max_gas": "-1",
        "time_iota_ms": "1000"
      },
      "evidence": {
        "max_age": "100000"
      },
      "validator": {
        "pub_key_types": [
          "ed25519"
        ]
      }
    }
  }
}
```

#### 6.1.12 Query ConsensusState
ConsensusState returns a concise summary of the consensus state. This is just a snapshot of consensus state, and it will not persist.

#### Return Type
Return round states
```
type ResultConsensusState struct {
    RoundState json.RawMessage `
}
```

#### Example

Run the command:
```
curl 'localhost:26657/consensus_state'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "round_state": {
      "height/round/step": "3227/0/1",
      "start_time": "2019-10-10T03:31:57.438164Z",
      "proposal_block_hash": "",
      "locked_block_hash": "",
      "valid_block_hash": "",
      "height_vote_set": [
        {
          "round": "0",
          "prevotes": [
            "nil-Vote"
          ],
          "prevotes_bit_array": "BA{1:_} 0/100 = 0.00",
          "precommits": [
            "nil-Vote"
          ],
          "precommits_bit_array": "BA{1:_} 0/100 = 0.00"
        }
      ]
    }
  }
}
```

#### 6.1.13 Query DumpConsensusState
DumpConsensusState dumps consensus state. This is just a snapshot of consensus state, and it will not persist.

#### Return Type
Return round states
```
type ResultConsensusState struct {
    RoundState json.RawMessage `
}
```

#### Example

Run the command:
```
curl 'http://localhost:26657/dump_consensus_state'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "round_state": {
      "height": "3331",
      "round": "0",
      "step": 1,
      "start_time": "2019-10-10T03:40:45.881035Z",
      "commit_time": "2019-10-10T03:40:40.881035Z",
      "validators": {
        "validators": [
          {
            "address": "CE9CA702BE3125D62B062BB925A552882CBC08CA",
            "pub_key": {
              "type": "tendermint/PubKeyEd25519",
              "value": "l6Zl8owP5UHMm8Cx3wCHqFdeJMSrz7KgSs9YSQW3c88="
            },
            "voting_power": "100",
            "proposer_priority": "0"
          }
        ],
        "proposer": {
          "address": "CE9CA702BE3125D62B062BB925A552882CBC08CA",
          "pub_key": {
            "type": "tendermint/PubKeyEd25519",
            "value": "l6Zl8owP5UHMm8Cx3wCHqFdeJMSrz7KgSs9YSQW3c88="
          },
          "voting_power": "100",
          "proposer_priority": "0"
        }
      },
      "proposal": null,
      "proposal_block": null,
      "proposal_block_parts": null,
      "locked_round": "-1",
      "locked_block": null,
      "locked_block_parts": null,
      "valid_round": "-1",
      "valid_block": null,
      "valid_block_parts": null,
      "votes": [
        {
          "round": "0",
          "prevotes": [
            "nil-Vote"
          ],
          "prevotes_bit_array": "BA{1:_} 0/100 = 0.00",
          "precommits": [
            "nil-Vote"
          ],
          "precommits_bit_array": "BA{1:_} 0/100 = 0.00"
        }
      ],
      "commit_round": "-1",
      "last_commit": {
        "votes": [
          "Vote{0:CE9CA702BE31 3330/00/2(Precommit) 55C9AD369D14 72D33688A200 @ 2019-10-10T03:40:40.862611Z}"
        ],
        "votes_bit_array": "BA{1:x} 100/100 = 1.00",
        "peer_maj_23s": {}
      },
      "last_validators": {
        "validators": [
          {
            "address": "CE9CA702BE3125D62B062BB925A552882CBC08CA",
            "pub_key": {
              "type": "tendermint/PubKeyEd25519",
              "value": "l6Zl8owP5UHMm8Cx3wCHqFdeJMSrz7KgSs9YSQW3c88="
            },
            "voting_power": "100",
            "proposer_priority": "0"
          }
        ],
        "proposer": {
          "address": "CE9CA702BE3125D62B062BB925A552882CBC08CA",
          "pub_key": {
            "type": "tendermint/PubKeyEd25519",
            "value": "l6Zl8owP5UHMm8Cx3wCHqFdeJMSrz7KgSs9YSQW3c88="
          },
          "voting_power": "100",
          "proposer_priority": "0"
        }
      },
      "triggered_timeout_precommit": false
    },
    "peers": []
  }
}
```


#### 6.1.14 Query GenesisFile
Get genesis file.

#### Return Type
Return round states
```
// Genesis file
type ResultGenesis struct {
    Genesis *types.GenesisDoc
}
// GenesisDoc defines the initial conditions for a tendermint blockchain, in particular its validator set.
type GenesisDoc struct {
    GenesisTime     time.Time
    ChainID         string
    ConsensusParams *ConsensusParams
    Validators      []GenesisValidator
    AppHash         cmn.HexBytes
    AppState        json.RawMessage
}
```

#### Example

Run the command:
```
curl 'http://localhost:26657/genesis'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "genesis": {
      "genesis_time": "2019-10-09T03:35:53.661582Z",
      "chain_id": "maxonrow-chain",
      "consensus_params": {
        "block": {
          "max_bytes": "22020096",
          "max_gas": "-1",
          "time_iota_ms": "1000"
        },
        "evidence": {
          "max_age": "100000"
        },
        "validator": {
          "pub_key_types": [
            "ed25519"
          ]
        }
      },
      "app_hash": "",
      "app_state": {
        "auth": {
          "params": {
            "max_memo_characters": "256",
            "tx_sig_limit": "7",
            "tx_size_cost_per_byte": "0",
            "sig_verify_cost_ed25519": "0",
            "sig_verify_cost_secp256k1": "0"
          }
        },
        "accounts": [
          {
            "address": "mxw1urazz6vuldjp09gtzft0nw2kuv25l0793c6m9w",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1u7wn0ewe53csak2xl5nxaj45gangy3a4wq7xy7",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1xl7awvx7e7yjm256hx676vcqavx5c232rrtqpx",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1fkw4vdzgqd4jsvrt0whqjp6kr7mlt0rzl89lkv",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1vhz42dpefjhfrufurh2wwkpdt0837s5hxamt0u",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw146xepk070h8h2znggp8rdffn3ym3fda5gfjwxm",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw17tx5936pw0xfcqe4g76wxefwdl3j4rpdygt5wf",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1vulu4lra8nd2aswz60xkj5uxnp9kw0fuq3e65g",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1rnwy4utedvl7p2x49squm2a3a0d6gne4pe8h5w",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1kvc7pvyu65sg0hgqc6tnu0rd40g5qwayrwtqnh",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1vtqkfs7trns8jepp3mu8skqkcga90ky2zp7v7p",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1rq9txn872hchzdau2c44t464fqdvn3ad4drkp8",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1wc6dp6qrp48x3hjy6049l3f8phtkr9hqd2z5wh",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw19dgfsqpqnatfv298zna6wnlauv07kcvw386vew",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1smcavtfea3mpsplsual2jyx3jcps8vjv8yh43v",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1uzhu2au5r5gaafx6uj2hw87af44udw3wuehvvy",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw1v36a5t98pgywy59vhmwkr527u330uweqvxcj3p",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          },
          {
            "address": "mxw16g5gj7slym5a3amnyajycc9zw8yqmd74agludz",
            "coins": [
              {
                "denom": "cin",
                "amount": "1000000000000000000000"
              }
            ],
            "public_key": null,
            "account_number": "0",
            "sequence": "0"
          }
        ],
        "bank": {
          "send_enabled": true
        },
        "staking": {
          "params": {
            "unbonding_time": "1814400000000000",
            "max_validators": 100,
            "max_entries": 7,
            "bond_denom": "cin"
          },
          "last_total_power": "0",
          "last_validator_powers": null,
          "validators": null,
          "delegations": null,
          "unbonding_delegations": null,
          "redelegations": null,
          "exported": false
        },
        "distribution": {
          "fee_pool": {
            "community_pool": null
          },
          "community_tax": "0.000000000000000000",
          "base_proposer_reward": "0.000000000000000000",
          "bonus_proposer_reward": "0.000000000000000000",
          "withdraw_addr_enabled": true,
          "delegator_withdraw_infos": null,
          "previous_proposer": "",
          "outstanding_rewards": null,
          "validator_accumulated_commissions": null,
          "validator_historical_rewards": null,
          "validator_current_rewards": null,
          "delegator_starting_infos": null,
          "validator_slash_events": null
        },
        "kyc": {
          "authorised_addresses": [
            "mxw1urazz6vuldjp09gtzft0nw2kuv25l0793c6m9w",
            "mxw1u7wn0ewe53csak2xl5nxaj45gangy3a4wq7xy7",
            "mxw1xl7awvx7e7yjm256hx676vcqavx5c232rrtqpx",
            "mxw1fkw4vdzgqd4jsvrt0whqjp6kr7mlt0rzl89lkv",
            "mxw1vhz42dpefjhfrufurh2wwkpdt0837s5hxamt0u",
            "mxw146xepk070h8h2znggp8rdffn3ym3fda5gfjwxm",
            "mxw17tx5936pw0xfcqe4g76wxefwdl3j4rpdygt5wf",
            "mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e",
            "mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
            "mxw1vulu4lra8nd2aswz60xkj5uxnp9kw0fuq3e65g",
            "mxw1rnwy4utedvl7p2x49squm2a3a0d6gne4pe8h5w",
            "mxw1kvc7pvyu65sg0hgqc6tnu0rd40g5qwayrwtqnh",
            "mxw1vtqkfs7trns8jepp3mu8skqkcga90ky2zp7v7p",
            "mxw1rq9txn872hchzdau2c44t464fqdvn3ad4drkp8",
            "mxw1wc6dp6qrp48x3hjy6049l3f8phtkr9hqd2z5wh",
            "mxw19dgfsqpqnatfv298zna6wnlauv07kcvw386vew",
            "mxw1smcavtfea3mpsplsual2jyx3jcps8vjv8yh43v",
            "mxw1uzhu2au5r5gaafx6uj2hw87af44udw3wuehvvy",
            "mxw1v36a5t98pgywy59vhmwkr527u330uweqvxcj3p",
            "mxw16g5gj7slym5a3amnyajycc9zw8yqmd74agludz"
          ],
          "issuer_addresses": null,
          "provider_addresses": null,
          "whitelisted_addresses": null
        },
        "token": {
          "authorised_addresses": null,
          "issuer_addresses": null,
          "provider_addresses": null
        },
        "nameservice": {
          "authorised_addresses": null,
          "issuer_addresses": null,
          "provider_addresses": null,
          "genesis_alias_owners": null
        },
        "fee": {
          "fee_settings": [
            {
              "name": "default",
              "min": [
                {
                  "denom": "cin",
                  "amount": "10000000000000000"
                }
              ],
              "max": [
                {
                  "denom": "cin",
                  "amount": "1000000000000000000000000"
                }
              ],
              "percentage": "0.05"
            }
          ],
          "authorised_addresses": [
            "mxw1urazz6vuldjp09gtzft0nw2kuv25l0793c6m9w"
          ],
          "multiplier": "1",
          "assigned_fee": [
            {
              "name": "zero",
              "msg_type": "kyc-whitelist"
            },
            {
              "name": "zero",
              "msg_type": "kyc-whitelist"
            },
            {
              "name": "zero",
              "msg_type": "kyc-revokeWhitelist"
            },
            {
              "name": "zero",
              "msg_type": "fee-createFeeSetting"
            },
            {
              "name": "zero",
              "msg_type": "fee-createTxFeeSetting"
            },
            {
              "name": "zero",
              "msg_type": "fee-createMultiplier"
            },
            {
              "name": "zero",
              "msg_type": "fee-createAccFeeSetting"
            }
          ]
        },
        "maintenance": {
          "maintainers": null,
          "starting_proposal_id": "1",
          "validator_set": [
            "mxwvalconspub1zcjduepqj7nxtu5vplj5rnymczca7qy84pt4ufxy408m9gz2eavyjpdhw08s8gcu3p"
          ]
        },
        "gentxs": [
          {
            "type": "cosmos-sdk/StdTx",
            "value": {
              "msg": [
                {
                  "type": "cosmos-sdk/MsgCreateValidator",
                  "value": {
                    "description": {
                      "moniker": "gohck.local",
                      "identity": "",
                      "website": "",
                      "security_contact": "",
                      "details": ""
                    },
                    "commission": {
                      "rate": "0.100000000000000000",
                      "max_rate": "0.200000000000000000",
                      "max_change_rate": "0.010000000000000000"
                    },
                    "min_self_delegation": "1",
                    "delegator_address": "mxw1urazz6vuldjp09gtzft0nw2kuv25l0793c6m9w",
                    "validator_address": "mxwvaloper1urazz6vuldjp09gtzft0nw2kuv25l079fh99g4",
                    "pubkey": "mxwvalconspub1zcjduepqj7nxtu5vplj5rnymczca7qy84pt4ufxy408m9gz2eavyjpdhw08s8gcu3p",
                    "value": {
                      "denom": "cin",
                      "amount": "100000000000000000000"
                    }
                  }
                }
              ],
              "fee": {
                "amount": [
                  {
                    "denom": "cin",
                    "amount": "0"
                  }
                ],
                "gas": "0"
              },
              "signatures": [
                {
                  "pub_key": {
                    "type": "tendermint/PubKeySecp256k1",
                    "value": "AqlVNA8KELZX2GgQ7dTCFIcmrDtZgBb1ajKrT9/AQ7Re"
                  },
                  "signature": "MA4EcO21wFbSzWy1LFSZyXiFNf+7nIEWcy6oHIEQiSUDfGZvlZQLFqxQyigVCaZXAlHaYwoZQPkdH60tqmMXqQ=="
                }
              ],
              "memo": "13d45324d870a0bcb93affb09ea58a774338aafd@192.168.20.42:26656"
            }
          }
        ]
      }
    }
  }
}
```
#### 6.1.15 Query Health
Get node health. Returns empty result (200 OK) on success, no response - in case of an error.

#### Return Type
```
ResultHealth{}
```

#### Example
Run the command:
```
curl 'http://localhost:26657/health'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {}
}
```

#### 6.1.16 IsWhitelisted
Returns status of whitelisted.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| address | string | false | true  | Bech32 OperatorAddress of account |


#### Return Type
```
type ResultIsWhitelisted struct {
    Results boolean
}
```


#### Example

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "is_whitelisted",
    "params": ["mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": true
}
```

#### 6.1.17 Query LatestBlockHeight
Get latest block height. 

#### Return Type
```
type ResultLatestBlockHeight struct {
    Results  int64
}
```


#### Example

Run the command:
```
curl 'http://localhost:26657/latest_block_height'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": "2366"
}
```
#### 6.1.18 Query NetInfo
Get network info.

#### Return Type
```
type ResultNetInfo struct {
    Listening bool     `json:"listening"`
    Listeners []string `json:"listeners"`
    NPeers    int      `json:"n_peers"`
    Peers     []Peer   `json:"peers"`
}
```

#### Example

Run the command:
```
curl 'http://localhost:26657/net_info'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "listening": true,
    "listeners": [
      "Listener(@)"
    ],
    "n_peers": "0",
    "peers": []
  }
}
```
#### 6.1.19 Query NumUnconfirmedTxs
Get number of unconfirmed transactions.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| limit | int | 30 | false    | Maximum number of entries (max: 100) |

#### Return Type
```
// List of mempool txs
type ResultUnconfirmedTxs struct {
    N   int
    Txs []types.Tx
}
```

#### Example

Run the command:
```
curl 'http://localhost:26657/num_unconfirmed_txs'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "n_txs": "0",
    "total": "0",
    "total_bytes": "0",
    "txs": null
  }
}
```

#### 6.1.20 Query Fee
Get the Fee Setting of a particular Tx.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| hash | []byte | nil | true    | transaction Hash to retrive |


#### Return Type
```
type ResultQueryFee struct {
    Results *state.Responses
}
```


#### Example
Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "query_fee",
    "params": ["eyJ0eXBlIjoiY29zbW9zLXNkay9TdGRUeCIsInZhbHVlIjp7ImZlZSI6eyJhbW91bnQiOlt7ImFtb3VudCI6IjUwMDAwMDAwMDAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0sImdhcyI6IjAifSwibWVtbyI6InRlc3RpbmcgdHJhbnNmZXIgd2l0aCAxMWNpbiwgZnJvbSBhY2MtOSB0byBhY2MtOCIsIm1zZyI6W3sidHlwZSI6Im14dy9tc2dTZW5kIiwidmFsdWUiOnsiYW1vdW50IjpbeyJhbW91bnQiOiIxMSIsImRlbm9tIjoiY2luIn1dLCJmcm9tX2FkZHJlc3MiOiJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJ0b19hZGRyZXNzIjoibXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRlIn19XSwic2lnbmF0dXJlcyI6W3sicHViX2tleSI6eyJ0eXBlIjoidGVuZGVybWludC9QdWJLZXlTZWNwMjU2azEiLCJ2YWx1ZSI6IkE1SWE4RUFvTllZbFhVNXk0aHV6c3czRm1FSXZLZ3pxd2lDOXhqZjFWekRYIn0sInNpZ25hdHVyZSI6IjFIZHZodkFNTkJGVlpYbTRMam1DMkJFeTQxSC9HQnFjUHJsbnZFRTVYT1VRZVNVTXJuZkd1NkhmeGhqOHNlKzNlZUZaTXVQSFZaZzFEdm9iV3pBRTRnPT0ifV19fQ=="],
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
    "amount": [
      {
        "denom": "cin",
        "amount": "10000000000000000"
      }
    ],
    "gas": "0"
  }
}
```
#### 6.1.21 Query Status
Get Tendermint status including node info, pubkey, latest block hash, app hash, block height and time.

#### Return Type
```
type ResultStatus struct {
    NodeInfo      p2p.DefaultNodeInfo
    SyncInfo      SyncInfo
    ValidatorInfo ValidatorInfo
}
```

#### Example
Run the command:
```
curl 'localhost:26657/status'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "node_info": {
      "protocol_version": {
        "p2p": "7",
        "block": "10",
        "app": "0"
      },
      "id": "13d45324d870a0bcb93affb09ea58a774338aafd",
      "listen_addr": "tcp://0.0.0.0:26656",
      "network": "maxonrow-chain",
      "version": "0.32.2",
      "channels": "4020212223303800",
      "moniker": "gohck.local",
      "other": {
        "tx_index": "on",
        "rpc_address": "tcp://127.0.0.1:26657"
      }
    },
    "sync_info": {
      "latest_block_hash": "9882052FF69F928B1697FCE6C8F0BAB9DABC374C93EF56820E864E4126AF1590",
      "latest_app_hash": "7F8A730CAB95686CD6935842089B1E2ED7AE13F3E20E13677120FE26A4E91B0E",
      "latest_block_height": "2290",
      "latest_block_time": "2019-10-09T10:32:17.999343Z",
      "catching_up": false
    },
    "validator_info": {
      "address": "CE9CA702BE3125D62B062BB925A552882CBC08CA",
      "pub_key": {
        "type": "tendermint/PubKeyEd25519",
        "value": "l6Zl8owP5UHMm8Cx3wCHqFdeJMSrz7KgSs9YSQW3c88="
      },
      "voting_power": "100"
    }
  }
}
```

#### 6.1.22 Query Tx
Get transactions by hash. Tx allows you to query the transaction results. nil could mean the transaction is in the mempool, invalidated, or was not sent in the first place.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| hash | []byte | nil | true    | transaction Hash to retrive |
| prove | bool | false | false     | Include proofs of the transactions inclusion in the block |



#### Return Type
Returns a transaction matching the given transaction hash.
```
type ResultTx struct {
    Hash     cmn.HexBytes          //hash of the transaction
    Height   int64                  //height of the block where this transaction was in
    Index    uint32                 //index of the transaction
    TxResult abci.ResponseDeliverTx //the `abci.Result` object
    Tx       types.Tx               //the transaction
    Proof    types.TxProof          //the `types.TxProof` object
}
```

#### Example

Run the command:
```
curl 'http://localhost:26657/tx?hash=0x79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250",
    "index": 0,
    "tx_result": {
      "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
      "gasUsed": "78131",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "YWN0aW9u",
              "value": "c2VuZA=="
            }
          ]
        },
        {
          "type": "message",
          "attributes": [
            {
              "key": "RnJvbQ==",
              "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
            },
            {
              "key": "VG8=",
              "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
            },
            {
              "key": "QW1vdW50",
              "value": "MTE="
            }
          ]
        },
        {
          "type": "system",
          "attributes": [
            {
              "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
              "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
            }
          ]
        }
      ]
    },
    "tx": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04"
  }
}
```

#### 6.1.23 Query TxSearch
TxSearch allows you to query for multiple transactions results.You could search transaction by its index. It returns a list of transactions (maximum ?per_page entries) and the total count.

#### Enable Indexer

You need to enable indexer in config.tml. You can modify the index_tags to tx.height, which is the only tag we support now. In this way, you can index transactions by height by adding "tx.height" tag here.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| query | string | " " | true   | Query |
| prove | bool | false | false   | Include proofs of the transactions inclusion in the block |
| page | int | 1 | false   | Page number (1-based) |
| per_page | int | 30 | false   | Number of entries per page (max: 100) |

#### Return Type
- proof: the types.TxProof object
- tx: []byte - the transaction
- tx_result: the abci.Result object
- index: int - index of the transaction
- height: int - height of the block where this transaction was in
- hash: []byte - hash of the transaction

#### Example
For example, if you want to search all the transations happened on height 250.

Run the command:
```
curl 'localhost:26657/tx_search?query="tx.height=250"&prove=true'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "txs": [
      {
        "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
        "height": "250",
        "index": 0,
        "tx_result": {
          "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
          "gasUsed": "78131",
          "events": [
            {
              "type": "message",
              "attributes": [
                {
                  "key": "YWN0aW9u",
                  "value": "c2VuZA=="
                }
              ]
            },
            {
              "type": "message",
              "attributes": [
                {
                  "key": "RnJvbQ==",
                  "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
                },
                {
                  "key": "VG8=",
                  "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
                },
                {
                  "key": "QW1vdW50",
                  "value": "MTE="
                }
              ]
            },
            {
              "type": "system",
              "attributes": [
                {
                  "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
                  "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
                }
              ]
            }
          ]
        },
        "tx": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04",
        "proof": {
          "RootHash": "C5FFE7245DDDB3B80637863EFD86AB1E35F790DA9A38C703F84FF6760F448860",
          "Data": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04",
          "Proof": {
            "total": "1",
            "index": "0",
            "leaf_hash": "xf/nJF3ds7gGN4Y+/YarHjX3kNqaOMcD+E/2dg9EiGA=",
            "aunts": []
          }
        }
      }
    ],
    "total_count": "1"
  }
}
```

#### 6.1.24 Query UnconfirmedTxs
Get list of unconfirmed transactions

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| limit | int | 30 | false    | Maximum number of unconfirmed transactions to return |

#### Return Type
List of unconfirmed transactions
```
type ResultQueryUnconfirmedTxs struct {
    Results *state.Responses
}
```

#### Example
Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "unconfirmed_txs",
    "params": ["1"],
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
    "n_txs": "0",
    "total": "0",
    "total_bytes": "0",
    "txs": []
  }
}
```

#### 6.1.25 Query ValidatorsSetHeight 
Get validators set a certain height

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| height  | int | false | true    | Block height |


#### Return Type
```
type ResultValidatorsetHeight struct {
    Results *state.Responses
}
```

#### Example

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "validators",
    "params": ["250"],
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
    "block_height": "250",
    "validators": [
      {
        "address": "CE9CA702BE3125D62B062BB925A552882CBC08CA",
        "pub_key": {
          "type": "tendermint/PubKeyEd25519",
          "value": "l6Zl8owP5UHMm8Cx3wCHqFdeJMSrz7KgSs9YSQW3c88="
        },
        "voting_power": "100",
        "proposer_priority": "0"
      }
    ]
  }
}
```

#### 6.1.26 Query Validator
Query the information from a single validator

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| address | string | false | true    | Bech32 OperatorAddress of validator |


#### Return Type
```
type ResultValidator struct {
    Results *state.Responses
}
```


#### Example

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "validator",
    "params": ["mxwvaloper1urazz6vuldjp09gtzft0nw2kuv25l079fh99g4"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "
    { 
    "operator_address":"mxwvaloper1urazz6vuldjp09gtzft0nw2kuv25l079fh99g4",
    "consensus_pubkey":"mxwvalconspub1zcjduepqj7nxtu5vplj5rnymczca7qy84pt4ufxy408m9gz2eavyjpdhw08s8gcu3p",
    "jailed":false,
    "status":2,
    "tokens":"100000000000000000000",
    "delegator_shares":"100000000000000000000.000000000000000000",
    "description":{ 
        "moniker":"local",
        "identity":"",
        "website":"",
        "security_contact":"",
        "details":""
    },
    "unbonding_height":"0",
    "unbonding_time":"1970-01-01T00:00:00Z",
    "commission":{ 
        "commission_rates":{ 
            "rate":"0.100000000000000000",
            "max_rate":"0.200000000000000000",
            "max_change_rate":"0.010000000000000000"
        },
        "update_time":"2019-10-09T03:35:53.661582Z"
    },
    "min_self_delegation":"1"
    }"
}
```
#### 6.1.27 Query Version
Get the version of maxonrow running locally to compare against expected.


#### Return Type
```
type ResultVersion struct {
    Results *state.Responses
}
```


#### Example
Run the command:
```
curl 'http://localhost:26657/version'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "tendermintversion": "0.32.2",
    "maxonrowversion": "0.6.3",
    "cosmosversion": ""
  }
}
```

#### 6.2 Tx APIs

#### 6.2.1 DecodeTx
Return readable Transaction Data using tx hash value.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| hash | string | false | true    | transaction Hash to retrive |
| prove | bool | false | false    | include a proof of the transaction inclusion in the block in the result (default: false). |


#### Return Type
```
type ResultDecodeTx struct {
    Results *state.Responses
}
```

#### Example
Run the command:
```
curl 'http://localhost:26657/decoded_tx?hash=0x79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9&prove=true'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250",
    "index": 0,
    "tx_result": {
      "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
      "gasUsed": "78131",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "YWN0aW9u",
              "value": "c2VuZA=="
            }
          ]
        },
        {
          "type": "message",
          "attributes": [
            {
              "key": "RnJvbQ==",
              "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
            },
            {
              "key": "VG8=",
              "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
            },
            {
              "key": "QW1vdW50",
              "value": "MTE="
            }
          ]
        },
        {
          "type": "system",
          "attributes": [
            {
              "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
              "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
            }
          ]
        }
      ]
    },
    "tx": "{\"type\":\"cosmos-sdk/StdTx\",\"value\":{\"fee\":{\"amount\":[{\"amount\":\"50000000000000000\",\"denom\":\"cin\"}],\"gas\":\"0\"},\"memo\":\"testing transfer with 11cin, from acc-9 to acc-8\",\"msg\":[{\"type\":\"mxw/msgSend\",\"value\":{\"amount\":[{\"amount\":\"11\",\"denom\":\"cin\"}],\"from_address\":\"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p\",\"to_address\":\"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e\"}}],\"signatures\":[{\"pub_key\":{\"type\":\"tendermint/PubKeySecp256k1\",\"value\":\"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX\"},\"signature\":\"1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4g==\"}]}}",
    "proof": {
      "RootHash": "C5FFE7245DDDB3B80637863EFD86AB1E35F790DA9A38C703F84FF6760F448860",
      "Data": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04",
      "Proof": {
        "total": "1",
        "index": "0",
        "leaf_hash": "xf/nJF3ds7gGN4Y+/YarHjX3kNqaOMcD+E/2dg9EiGA=",
        "aunts": []
      }
    }
  }
}
```
#### 6.2.2 EncodeAndBroadcastTxAsync
This method just return transaction hash right away and there is no return from CheckTx or DeliverTx.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| tx | Tx | nil | true  | The transaction info bytes in hex |



#### Return Type
CheckTx Result
```
type ResultEncodeAndBroadcastTxAsync struct {
    Code uint32
    Data cmn.HexBytes
    Log  string
    Hash cmn.HexBytes
}
```

#### Example

Step-1. Query the account number and sequence of the TWO account-address 

Run the command with the JSON request body:
```
http://localhost:26657/
```

```
{
    "method": "account",
    "params": ["mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "
    { 
   "type":"cosmos-sdk/Account",
   "value":{ 
      "address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
      "coins":[ 
         { 
            "denom":"cin",
            "amount":"1000000000000000000000"
         }
      ],
      "public_key":{ 
         "type":"tendermint/PubKeySecp256k1",
         "value":"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX"
      },
      "account_number":"8",
      "sequence":"0"
   }}"
}
```

Run the command with the JSON request body:
```
http://localhost:26657/
```

```
{
    "method": "account",
    "params": ["mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "
    { 
   "type":"cosmos-sdk/Account",
   "value":{ 
      "address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e",
      "coins":[ 
         { 
            "denom":"cin",
            "amount":"1000000000000000000000"
         }
      ],
      "public_key":{ 
         "type":"tendermint/PubKeySecp256k1",
         "value":"A9Nmqbf2cKAOs9hzGTRyS2Wd6uLBWrwkZ0S7q06N0/3i"
      },
      "account_number":"7",
      "sequence":"0"
   }}"
}
```


Step-2. Generate the transaction output as json format

```
mxwcli tx send mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e 11cin --fees 50000000000000000cin --gas 0 --memo "testing transfer with 11cin, from acc-9 to acc-8" --home ~/.mxw_20191009_local01 --chain-id=maxonrow-chain
```

```
{ 
   "chain_id":"maxonrow-chain",
   "account_number":"8",
   "sequence":"0",
   "fee":{ 
      "amount":[ 
         { 
            "denom":"cin",
            "amount":"50000000000000000"
         }
      ],
      "gas":"0"
   },
   "msgs":[ 
      { 
         "type":"mxw/msgSend",
         "value":{ 
            "from_address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
            "to_address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e",
            "amount":[ 
               { 
                  "denom":"cin",
                  "amount":"11"
               }
            ]
         }
      }
   ],
   "memo":"testing transfer with 11cin, from acc-9 to acc-8"
}
```

```
confirm transaction before signing and broadcasting [y/N]: y
Password to sign with 'acc-9':
```

```
height: 0
txhash: 79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9
code: 0
data: ""
rawlog: '[{"msg_index":0,"success":true,"log":""}]'
logs:
- msgindex: 0
  success: true
  log: ""
info: ""
gaswanted: 0
gasused: 0
events: []
codespace: ""
tx: null
timestamp: ""
```

Step-3. You could decode using the decode_tx, base on the value of txhash 

Run the command:
```
curl 'http://localhost:26657/decoded_tx?hash=0x79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9&prove=true'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250",
    "index": 0,
    "tx_result": {
      "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
      "gasUsed": "78131",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "YWN0aW9u",
              "value": "c2VuZA=="
            }
          ]
        },
        {
          "type": "message",
          "attributes": [
            {
              "key": "RnJvbQ==",
              "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
            },
            {
              "key": "VG8=",
              "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
            },
            {
              "key": "QW1vdW50",
              "value": "MTE="
            }
          ]
        },
        {
          "type": "system",
          "attributes": [
            {
              "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
              "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
            }
          ]
        }
      ]
    },
    "tx": "{\"type\":\"cosmos-sdk/StdTx\",\"value\":{\"fee\":{\"amount\":[{\"amount\":\"50000000000000000\",\"denom\":\"cin\"}],\"gas\":\"0\"},\"memo\":\"testing transfer with 11cin, from acc-9 to acc-8\",\"msg\":[{\"type\":\"mxw/msgSend\",\"value\":{\"amount\":[{\"amount\":\"11\",\"denom\":\"cin\"}],\"from_address\":\"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p\",\"to_address\":\"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e\"}}],\"signatures\":[{\"pub_key\":{\"type\":\"tendermint/PubKeySecp256k1\",\"value\":\"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX\"},\"signature\":\"1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4g==\"}]}}",
    "proof": {
      "RootHash": "C5FFE7245DDDB3B80637863EFD86AB1E35F790DA9A38C703F84FF6760F448860",
      "Data": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04",
      "Proof": {
        "total": "1",
        "index": "0",
        "leaf_hash": "xf/nJF3ds7gGN4Y+/YarHjX3kNqaOMcD+E/2dg9EiGA=",
        "aunts": []
      }
    }
  }
}
```

Step-4. You proceed below : 

4.1 Do the filtering by remove symbol of '\', base on the value of tx like below
```
{ 
   "type":"cosmos-sdk/StdTx",
   "value":{ 
      "fee":{ 
         "amount":[ 
            { 
               "amount":"50000000000000000",
               "denom":"cin"
            }
         ],
         "gas":"0"
      },
      "memo":"testing transfer with 11cin, from acc-9 to acc-8",
      "msg":[ 
         { 
            "type":"mxw/msgSend",
            "value":{ 
               "amount":[ 
                  { 
                     "amount":"11",
                     "denom":"cin"
                  }
               ],
               "from_address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
               "to_address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e"
            }
         }
      ],
      "signatures":[ 
         { 
            "pub_key":{ 
               "type":"tendermint/PubKeySecp256k1",
               "value":"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX"
            },
            "signature":"1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4g=="
         }
      ]
   }
}
```

4.2 Then convert the value into base-64 format
```
eyJ0eXBlIjoiY29zbW9zLXNkay9TdGRUeCIsInZhbHVlIjp7ImZlZSI6eyJhbW91bnQiOlt7ImFtb3VudCI6IjUwMDAwMDAwMDAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0sImdhcyI6IjAifSwibWVtbyI6InRlc3RpbmcgdHJhbnNmZXIgd2l0aCAxMWNpbiwgZnJvbSBhY2MtOSB0byBhY2MtOCIsIm1zZyI6W3sidHlwZSI6Im14dy9tc2dTZW5kIiwidmFsdWUiOnsiYW1vdW50IjpbeyJhbW91bnQiOiIxMSIsImRlbm9tIjoiY2luIn1dLCJmcm9tX2FkZHJlc3MiOiJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJ0b19hZGRyZXNzIjoibXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRlIn19XSwic2lnbmF0dXJlcyI6W3sicHViX2tleSI6eyJ0eXBlIjoidGVuZGVybWludC9QdWJLZXlTZWNwMjU2azEiLCJ2YWx1ZSI6IkE1SWE4RUFvTllZbFhVNXk0aHV6c3czRm1FSXZLZ3pxd2lDOXhqZjFWekRYIn0sInNpZ25hdHVyZSI6IjFIZHZodkFNTkJGVlpYbTRMam1DMkJFeTQxSC9HQnFjUHJsbnZFRTVYT1VRZVNVTXJuZkd1NkhmeGhqOHNlKzNlZUZaTXVQSFZaZzFEdm9iV3pBRTRnPT0ifV19fQ==
```

Step-5. You could submit the transaction using the EncodeAndBroadcastTxAsync, using the above base-64 value.

Run the command with the JSON request body:
```
http://localhost:26657/
```

```
{
    "method": "encode_and_broadcast_tx_async",
    "params": ["eyJ0eXBlIjoiY29zbW9zLXNkay9TdGRUeCIsInZhbHVlIjp7ImZlZSI6eyJhbW91bnQiOlt7ImFtb3VudCI6IjUwMDAwMDAwMDAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0sImdhcyI6IjAifSwibWVtbyI6InRlc3RpbmcgdHJhbnNmZXIgd2l0aCAxMWNpbiwgZnJvbSBhY2MtOSB0byBhY2MtOCIsIm1zZyI6W3sidHlwZSI6Im14dy9tc2dTZW5kIiwidmFsdWUiOnsiYW1vdW50IjpbeyJhbW91bnQiOiIxMSIsImRlbm9tIjoiY2luIn1dLCJmcm9tX2FkZHJlc3MiOiJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJ0b19hZGRyZXNzIjoibXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRlIn19XSwic2lnbmF0dXJlcyI6W3sicHViX2tleSI6eyJ0eXBlIjoidGVuZGVybWludC9QdWJLZXlTZWNwMjU2azEiLCJ2YWx1ZSI6IkE1SWE4RUFvTllZbFhVNXk0aHV6c3czRm1FSXZLZ3pxd2lDOXhqZjFWekRYIn0sInNpZ25hdHVyZSI6IjFIZHZodkFNTkJGVlpYbTRMam1DMkJFeTQxSC9HQnFjUHJsbnZFRTVYT1VRZVNVTXJuZkd1NkhmeGhqOHNlKzNlZUZaTXVQSFZaZzFEdm9iV3pBRTRnPT0ifV19fQ="],
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
    "code": 0,
    "data": "",
    "log": "",
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9"
  }
}
```

Step-6. You can verify the current transaction using the Tx, base on the hash value.

Run the command:
```
curl 'http://localhost:26657/tx?hash=0x79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250",
    "index": 0,
    "tx_result": {
      "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
      "gasUsed": "78131",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "YWN0aW9u",
              "value": "c2VuZA=="
            }
          ]
        },
        {
          "type": "message",
          "attributes": [
            {
              "key": "RnJvbQ==",
              "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
            },
            {
              "key": "VG8=",
              "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
            },
            {
              "key": "QW1vdW50",
              "value": "MTE="
            }
          ]
        },
        {
          "type": "system",
          "attributes": [
            {
              "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
              "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
            }
          ]
        }
      ]
    },
    "tx": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04"
  }
}
```

#### 6.2.3 EncodeAndBroadcastTxcommit
The transaction will be encoded and broadcasted before returns with the response from CheckTx and DeliverTx.

This method will wait for both CheckTx and DeliverTx, so it is the slowest way to broadcast through RPC but offers the most accurate success/failure response.

CONTRACT: only returns error if mempool.CheckTx() errs or if we timeout waiting for tx to commit.

If CheckTx or DeliverTx fail, no error will be returned, but the returned result will contain a non-OK ABCI code.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| tx | Tx | nil | true  | The transaction info bytes in hex |

#### Return Type
CheckTx and DeliverTx results
```
type ResultEncodeAndBroadcastTxCommit struct {
    CheckTx   abci.ResponseCheckTx
    DeliverTx abci.ResponseDeliverTx
    Hash      cmn.HexBytes
    Height    int64
}
```

#### Example

Step-1. Query the account number and sequence of the TWO account-address 

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "account",
    "params": ["mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "
    { 
   "type":"cosmos-sdk/Account",
   "value":{ 
      "address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
      "coins":[ 
         { 
            "denom":"cin",
            "amount":"1000000000000000000000"
         }
      ],
      "public_key":{ 
         "type":"tendermint/PubKeySecp256k1",
         "value":"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX"
      },
      "account_number":"8",
      "sequence":"0"
   }}"
}
```

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "account",
    "params": ["mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "
    { 
   "type":"cosmos-sdk/Account",
   "value":{ 
      "address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e",
      "coins":[ 
         { 
            "denom":"cin",
            "amount":"1000000000000000000000"
         }
      ],
      "public_key":{ 
         "type":"tendermint/PubKeySecp256k1",
         "value":"A9Nmqbf2cKAOs9hzGTRyS2Wd6uLBWrwkZ0S7q06N0/3i"
      },
      "account_number":"7",
      "sequence":"0"
   }}"
}
```


Step-2. Generate the transaction output as json format

```
mxwcli tx send mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e 11cin --fees 50000000000000000cin --gas 0 --memo "testing transfer with 11cin, from acc-9 to acc-8" --home ~/.mxw_20191009_local01 --chain-id=maxonrow-chain
```

```
{ 
   "chain_id":"maxonrow-chain",
   "account_number":"8",
   "sequence":"0",
   "fee":{ 
      "amount":[ 
         { 
            "denom":"cin",
            "amount":"50000000000000000"
         }
      ],
      "gas":"0"
   },
   "msgs":[ 
      { 
         "type":"mxw/msgSend",
         "value":{ 
            "from_address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
            "to_address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e",
            "amount":[ 
               { 
                  "denom":"cin",
                  "amount":"11"
               }
            ]
         }
      }
   ],
   "memo":"testing transfer with 11cin, from acc-9 to acc-8"
}
```

```
confirm transaction before signing and broadcasting [y/N]: y
Password to sign with 'acc-9':
```

```
height: 0
txhash: 79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9
code: 0
data: ""
rawlog: '[{"msg_index":0,"success":true,"log":""}]'
logs:
- msgindex: 0
  success: true
  log: ""
info: ""
gaswanted: 0
gasused: 0
events: []
codespace: ""
tx: null
timestamp: ""
```

Step-3. You could decode using the decode_tx, base on the value of txhash 

Run the command:
```
curl 'http://localhost:26657/decoded_tx?hash=0x79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9&prove=true'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250",
    "index": 0,
    "tx_result": {
      "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
      "gasUsed": "78131",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "YWN0aW9u",
              "value": "c2VuZA=="
            }
          ]
        },
        {
          "type": "message",
          "attributes": [
            {
              "key": "RnJvbQ==",
              "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
            },
            {
              "key": "VG8=",
              "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
            },
            {
              "key": "QW1vdW50",
              "value": "MTE="
            }
          ]
        },
        {
          "type": "system",
          "attributes": [
            {
              "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
              "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
            }
          ]
        }
      ]
    },
    "tx": "{\"type\":\"cosmos-sdk/StdTx\",\"value\":{\"fee\":{\"amount\":[{\"amount\":\"50000000000000000\",\"denom\":\"cin\"}],\"gas\":\"0\"},\"memo\":\"testing transfer with 11cin, from acc-9 to acc-8\",\"msg\":[{\"type\":\"mxw/msgSend\",\"value\":{\"amount\":[{\"amount\":\"11\",\"denom\":\"cin\"}],\"from_address\":\"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p\",\"to_address\":\"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e\"}}],\"signatures\":[{\"pub_key\":{\"type\":\"tendermint/PubKeySecp256k1\",\"value\":\"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX\"},\"signature\":\"1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4g==\"}]}}",
    "proof": {
      "RootHash": "C5FFE7245DDDB3B80637863EFD86AB1E35F790DA9A38C703F84FF6760F448860",
      "Data": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04",
      "Proof": {
        "total": "1",
        "index": "0",
        "leaf_hash": "xf/nJF3ds7gGN4Y+/YarHjX3kNqaOMcD+E/2dg9EiGA=",
        "aunts": []
      }
    }
  }
}
```

Step-4. You proceed below : 

4.1 Do the filtering by remove symbol of '\', base on the value of tx like below
```
{ 
   "type":"cosmos-sdk/StdTx",
   "value":{ 
      "fee":{ 
         "amount":[ 
            { 
               "amount":"50000000000000000",
               "denom":"cin"
            }
         ],
         "gas":"0"
      },
      "memo":"testing transfer with 11cin, from acc-9 to acc-8",
      "msg":[ 
         { 
            "type":"mxw/msgSend",
            "value":{ 
               "amount":[ 
                  { 
                     "amount":"11",
                     "denom":"cin"
                  }
               ],
               "from_address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
               "to_address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e"
            }
         }
      ],
      "signatures":[ 
         { 
            "pub_key":{ 
               "type":"tendermint/PubKeySecp256k1",
               "value":"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX"
            },
            "signature":"1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4g=="
         }
      ]
   }
}
```

4.2 Then convert the value into base-64 format
```
eyJ0eXBlIjoiY29zbW9zLXNkay9TdGRUeCIsInZhbHVlIjp7ImZlZSI6eyJhbW91bnQiOlt7ImFtb3VudCI6IjUwMDAwMDAwMDAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0sImdhcyI6IjAifSwibWVtbyI6InRlc3RpbmcgdHJhbnNmZXIgd2l0aCAxMWNpbiwgZnJvbSBhY2MtOSB0byBhY2MtOCIsIm1zZyI6W3sidHlwZSI6Im14dy9tc2dTZW5kIiwidmFsdWUiOnsiYW1vdW50IjpbeyJhbW91bnQiOiIxMSIsImRlbm9tIjoiY2luIn1dLCJmcm9tX2FkZHJlc3MiOiJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJ0b19hZGRyZXNzIjoibXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRlIn19XSwic2lnbmF0dXJlcyI6W3sicHViX2tleSI6eyJ0eXBlIjoidGVuZGVybWludC9QdWJLZXlTZWNwMjU2azEiLCJ2YWx1ZSI6IkE1SWE4RUFvTllZbFhVNXk0aHV6c3czRm1FSXZLZ3pxd2lDOXhqZjFWekRYIn0sInNpZ25hdHVyZSI6IjFIZHZodkFNTkJGVlpYbTRMam1DMkJFeTQxSC9HQnFjUHJsbnZFRTVYT1VRZVNVTXJuZkd1NkhmeGhqOHNlKzNlZUZaTXVQSFZaZzFEdm9iV3pBRTRnPT0ifV19fQ==
```

Step-5. You could submit the transaction using the EncodeAndBroadcastTxCommit, using the above base-64 value.

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "encode_and_broadcast_tx_commit",
    "params": ["eyJ0eXBlIjoiY29zbW9zLXNkay9TdGRUeCIsInZhbHVlIjp7ImZlZSI6eyJhbW91bnQiOlt7ImFtb3VudCI6IjUwMDAwMDAwMDAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0sImdhcyI6IjAifSwibWVtbyI6InRlc3RpbmcgdHJhbnNmZXIgd2l0aCAxMWNpbiwgZnJvbSBhY2MtOSB0byBhY2MtOCIsIm1zZyI6W3sidHlwZSI6Im14dy9tc2dTZW5kIiwidmFsdWUiOnsiYW1vdW50IjpbeyJhbW91bnQiOiIxMSIsImRlbm9tIjoiY2luIn1dLCJmcm9tX2FkZHJlc3MiOiJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJ0b19hZGRyZXNzIjoibXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRlIn19XSwic2lnbmF0dXJlcyI6W3sicHViX2tleSI6eyJ0eXBlIjoidGVuZGVybWludC9QdWJLZXlTZWNwMjU2azEiLCJ2YWx1ZSI6IkE1SWE4RUFvTllZbFhVNXk0aHV6c3czRm1FSXZLZ3pxd2lDOXhqZjFWekRYIn0sInNpZ25hdHVyZSI6IjFIZHZodkFNTkJGVlpYbTRMam1DMkJFeTQxSC9HQnFjUHJsbnZFRTVYT1VRZVNVTXJuZkd1NkhmeGhqOHNlKzNlZUZaTXVQSFZaZzFEdm9iV3pBRTRnPT0ifV19fQ=="],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this. Please note that the returned data contains both CheckTx and DeliverTx output.

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": {
    "check_tx": {
      "code": 0,
      "log": "",
      "gasUsed": "28083",
      "events": []
    },
    "deliver_tx": {
         "code": 0,
         "log": "",
         "gasUsed": "28083",
         "events": []
    },
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250"
  }
}

```

Step-6. You can verify the current transaction using the Tx, base on the hash value.

Run the command:
```
curl 'http://localhost:26657/tx?hash=0x79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250",
    "index": 0,
    "tx_result": {
      "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
      "gasUsed": "78131",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "YWN0aW9u",
              "value": "c2VuZA=="
            }
          ]
        },
        {
          "type": "message",
          "attributes": [
            {
              "key": "RnJvbQ==",
              "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
            },
            {
              "key": "VG8=",
              "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
            },
            {
              "key": "QW1vdW50",
              "value": "MTE="
            }
          ]
        },
        {
          "type": "system",
          "attributes": [
            {
              "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
              "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
            }
          ]
        }
      ]
    },
    "tx": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04"
  }
}
```
#### 6.2.4 EncodeAndBroadcastTxsync
The transaction will be encoded and broadcasted before returns with the response from CheckTx.
Does not wait for DeliverTx result.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| tx | Tx | nil | true  | The transaction info bytes in hex |



#### Return Type
CheckTx Result
```
type ResultEncodeAndBroadcastTxSync struct {
    Code uint32
    Data cmn.HexBytes
    Log  string
    Hash cmn.HexBytes
}
```

#### Example

Step-1. Query the account number and sequence of the TWO account-address 

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "account",
    "params": ["mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "
    { 
   "type":"cosmos-sdk/Account",
   "value":{ 
      "address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
      "coins":[ 
         { 
            "denom":"cin",
            "amount":"1000000000000000000000"
         }
      ],
      "public_key":{ 
         "type":"tendermint/PubKeySecp256k1",
         "value":"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX"
      },
      "account_number":"8",
      "sequence":"0"
   }}"
}
```

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "account",
    "params": ["mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e"],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": 0,
  "result": "
    { 
   "type":"cosmos-sdk/Account",
   "value":{ 
      "address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e",
      "coins":[ 
         { 
            "denom":"cin",
            "amount":"1000000000000000000000"
         }
      ],
      "public_key":{ 
         "type":"tendermint/PubKeySecp256k1",
         "value":"A9Nmqbf2cKAOs9hzGTRyS2Wd6uLBWrwkZ0S7q06N0/3i"
      },
      "account_number":"7",
      "sequence":"0"
   }}"
}
```


Step-2. Generate the transaction output as json format

```
mxwcli tx send mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e 11cin --fees 50000000000000000cin --gas 0 --memo "testing transfer with 11cin, from acc-9 to acc-8" --home ~/.mxw_20191009_local01 --chain-id=maxonrow-chain
```

```
{ 
   "chain_id":"maxonrow-chain",
   "account_number":"8",
   "sequence":"0",
   "fee":{ 
      "amount":[ 
         { 
            "denom":"cin",
            "amount":"50000000000000000"
         }
      ],
      "gas":"0"
   },
   "msgs":[ 
      { 
         "type":"mxw/msgSend",
         "value":{ 
            "from_address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
            "to_address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e",
            "amount":[ 
               { 
                  "denom":"cin",
                  "amount":"11"
               }
            ]
         }
      }
   ],
   "memo":"testing transfer with 11cin, from acc-9 to acc-8"
}
```

```
confirm transaction before signing and broadcasting [y/N]: y
Password to sign with 'acc-9':
```

```
height: 0
txhash: 79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9
code: 0
data: ""
rawlog: '[{"msg_index":0,"success":true,"log":""}]'
logs:
- msgindex: 0
  success: true
  log: ""
info: ""
gaswanted: 0
gasused: 0
events: []
codespace: ""
tx: null
timestamp: ""
```

Step-3. You could decode using the decode_tx, base on the value of txhash 

Run the command:
```
curl 'http://localhost:26657/decoded_tx?hash=0x79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9&prove=true'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250",
    "index": 0,
    "tx_result": {
      "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
      "gasUsed": "78131",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "YWN0aW9u",
              "value": "c2VuZA=="
            }
          ]
        },
        {
          "type": "message",
          "attributes": [
            {
              "key": "RnJvbQ==",
              "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
            },
            {
              "key": "VG8=",
              "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
            },
            {
              "key": "QW1vdW50",
              "value": "MTE="
            }
          ]
        },
        {
          "type": "system",
          "attributes": [
            {
              "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
              "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
            }
          ]
        }
      ]
    },
    "tx": "{\"type\":\"cosmos-sdk/StdTx\",\"value\":{\"fee\":{\"amount\":[{\"amount\":\"50000000000000000\",\"denom\":\"cin\"}],\"gas\":\"0\"},\"memo\":\"testing transfer with 11cin, from acc-9 to acc-8\",\"msg\":[{\"type\":\"mxw/msgSend\",\"value\":{\"amount\":[{\"amount\":\"11\",\"denom\":\"cin\"}],\"from_address\":\"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p\",\"to_address\":\"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e\"}}],\"signatures\":[{\"pub_key\":{\"type\":\"tendermint/PubKeySecp256k1\",\"value\":\"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX\"},\"signature\":\"1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4g==\"}]}}",
    "proof": {
      "RootHash": "C5FFE7245DDDB3B80637863EFD86AB1E35F790DA9A38C703F84FF6760F448860",
      "Data": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04",
      "Proof": {
        "total": "1",
        "index": "0",
        "leaf_hash": "xf/nJF3ds7gGN4Y+/YarHjX3kNqaOMcD+E/2dg9EiGA=",
        "aunts": []
      }
    }
  }
}
```

Step-4. You proceed below : 

4.1 Do the filtering by remove symbol of '\', base on the value of tx like below
```
{ 
   "type":"cosmos-sdk/StdTx",
   "value":{ 
      "fee":{ 
         "amount":[ 
            { 
               "amount":"50000000000000000",
               "denom":"cin"
            }
         ],
         "gas":"0"
      },
      "memo":"testing transfer with 11cin, from acc-9 to acc-8",
      "msg":[ 
         { 
            "type":"mxw/msgSend",
            "value":{ 
               "amount":[ 
                  { 
                     "amount":"11",
                     "denom":"cin"
                  }
               ],
               "from_address":"mxw1xs5946fk53jnsxd2axjtkjmw6ks3h4ducs5a7p",
               "to_address":"mxw1aglwdehkrgan9p86ea8kl75lkag2h0xtwj2d4e"
            }
         }
      ],
      "signatures":[ 
         { 
            "pub_key":{ 
               "type":"tendermint/PubKeySecp256k1",
               "value":"A5Ia8EAoNYYlXU5y4huzsw3FmEIvKgzqwiC9xjf1VzDX"
            },
            "signature":"1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4g=="
         }
      ]
   }
}
```

4.2 Then convert the value into base-64 format
```
eyJ0eXBlIjoiY29zbW9zLXNkay9TdGRUeCIsInZhbHVlIjp7ImZlZSI6eyJhbW91bnQiOlt7ImFtb3VudCI6IjUwMDAwMDAwMDAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0sImdhcyI6IjAifSwibWVtbyI6InRlc3RpbmcgdHJhbnNmZXIgd2l0aCAxMWNpbiwgZnJvbSBhY2MtOSB0byBhY2MtOCIsIm1zZyI6W3sidHlwZSI6Im14dy9tc2dTZW5kIiwidmFsdWUiOnsiYW1vdW50IjpbeyJhbW91bnQiOiIxMSIsImRlbm9tIjoiY2luIn1dLCJmcm9tX2FkZHJlc3MiOiJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJ0b19hZGRyZXNzIjoibXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRlIn19XSwic2lnbmF0dXJlcyI6W3sicHViX2tleSI6eyJ0eXBlIjoidGVuZGVybWludC9QdWJLZXlTZWNwMjU2azEiLCJ2YWx1ZSI6IkE1SWE4RUFvTllZbFhVNXk0aHV6c3czRm1FSXZLZ3pxd2lDOXhqZjFWekRYIn0sInNpZ25hdHVyZSI6IjFIZHZodkFNTkJGVlpYbTRMam1DMkJFeTQxSC9HQnFjUHJsbnZFRTVYT1VRZVNVTXJuZkd1NkhmeGhqOHNlKzNlZUZaTXVQSFZaZzFEdm9iV3pBRTRnPT0ifV19fQ==
```


Step-5. You could submit the transaction using the EncodeAndBroadcastTxSync, using the above base-64 value.

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "encode_and_broadcast_tx_sync",
    "params": ["eyJ0eXBlIjoiY29zbW9zLXNkay9TdGRUeCIsInZhbHVlIjp7ImZlZSI6eyJhbW91bnQiOlt7ImFtb3VudCI6IjUwMDAwMDAwMDAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0sImdhcyI6IjAifSwibWVtbyI6InRlc3RpbmcgdHJhbnNmZXIgd2l0aCAxMWNpbiwgZnJvbSBhY2MtOSB0byBhY2MtOCIsIm1zZyI6W3sidHlwZSI6Im14dy9tc2dTZW5kIiwidmFsdWUiOnsiYW1vdW50IjpbeyJhbW91bnQiOiIxMSIsImRlbm9tIjoiY2luIn1dLCJmcm9tX2FkZHJlc3MiOiJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJ0b19hZGRyZXNzIjoibXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRlIn19XSwic2lnbmF0dXJlcyI6W3sicHViX2tleSI6eyJ0eXBlIjoidGVuZGVybWludC9QdWJLZXlTZWNwMjU2azEiLCJ2YWx1ZSI6IkE1SWE4RUFvTllZbFhVNXk0aHV6c3czRm1FSXZLZ3pxd2lDOXhqZjFWekRYIn0sInNpZ25hdHVyZSI6IjFIZHZodkFNTkJGVlpYbTRMam1DMkJFeTQxSC9HQnFjUHJsbnZFRTVYT1VRZVNVTXJuZkd1NkhmeGhqOHNlKzNlZUZaTXVQSFZaZzFEdm9iV3pBRTRnPT0ifV19fQ=="],
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
    "code": 0,
    "data": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04",
    "log": "",
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9"
  }
}

```

Step-6. You can verify the current transaction using the Tx, base on the hash value.

Run the command:
```
curl 'http://localhost:26657/tx?hash=0x79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {
    "hash": "79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9",
    "height": "250",
    "index": 0,
    "tx_result": {
      "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"{\\\"hash\\\":\\\"79215E24FEB4016F1CB70049D771800B7F622749C4602B6BC328142DCDD493C9\\\",\\\"nonce\\\":0}\"}]",
      "gasUsed": "78131",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "YWN0aW9u",
              "value": "c2VuZA=="
            }
          ]
        },
        {
          "type": "message",
          "attributes": [
            {
              "key": "RnJvbQ==",
              "value": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw"
            },
            {
              "key": "VG8=",
              "value": "bXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRl"
            },
            {
              "key": "QW1vdW50",
              "value": "MTE="
            }
          ]
        },
        {
          "type": "system",
          "attributes": [
            {
              "key": "bXh3MXhzNTk0NmZrNTNqbnN4ZDJheGp0a2ptdzZrczNoNGR1Y3M1YTdw",
              "value": "eyJoYXNoIjoiMmNhZGNmYjBjMzM2NzY5ZDUwM2Q1NTdiMjZmY2YxZTkxODE5ZTdlNSIsInBhcmFtcyI6WyJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJteHcxYWdsd2RlaGtyZ2FuOXA4NmVhOGtsNzVsa2FnMmgweHR3ajJkNGUiLCIxMSJdfQ=="
            }
          ]
        }
      ]
    },
    "tx": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04"
  }
}
```
#### 6.2.5 EncodeTx
Return encoded Transaction Data using tx hash value.

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| hash | string | false | true    | transaction Hash to retrive |
| prove | bool | false | false    | include a proof of the transaction inclusion in the block in the result (default: false). |


#### Return Type
```
type ResultEncodeTx struct {
    Results string
}
```


#### Example

Run the command with the JSON request body:
```
curl 'http://localhost:26657/'
```

```
{
    "method": "encode_tx",
    "params": ["eyJ0eXBlIjoiY29zbW9zLXNkay9TdGRUeCIsInZhbHVlIjp7ImZlZSI6eyJhbW91bnQiOlt7ImFtb3VudCI6IjUwMDAwMDAwMDAwMDAwMDAwIiwiZGVub20iOiJjaW4ifV0sImdhcyI6IjAifSwibWVtbyI6InRlc3RpbmcgdHJhbnNmZXIgd2l0aCAxMWNpbiwgZnJvbSBhY2MtOSB0byBhY2MtOCIsIm1zZyI6W3sidHlwZSI6Im14dy9tc2dTZW5kIiwidmFsdWUiOnsiYW1vdW50IjpbeyJhbW91bnQiOiIxMSIsImRlbm9tIjoiY2luIn1dLCJmcm9tX2FkZHJlc3MiOiJteHcxeHM1OTQ2Zms1M2puc3hkMmF4anRram13NmtzM2g0ZHVjczVhN3AiLCJ0b19hZGRyZXNzIjoibXh3MWFnbHdkZWhrcmdhbjlwODZlYThrbDc1bGthZzJoMHh0d2oyZDRlIn19XSwic2lnbmF0dXJlcyI6W3sicHViX2tleSI6eyJ0eXBlIjoidGVuZGVybWludC9QdWJLZXlTZWNwMjU2azEiLCJ2YWx1ZSI6IkE1SWE4RUFvTllZbFhVNXk0aHV6c3czRm1FSXZLZ3pxd2lDOXhqZjFWekRYIn0sInNpZ25hdHVyZSI6IjFIZHZodkFNTkJGVlpYbTRMam1DMkJFeTQxSC9HQnFjUHJsbnZFRTVYT1VRZVNVTXJuZkd1NkhmeGhqOHNlKzNlZUZaTXVQSFZaZzFEdm9iV3pBRTRnPT0ifV19fQ=="],
    "id": 0,
    "jsonrpc": "2.0"
}
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "result": "/QEoKBapCj12B33fCjcKFDQoWuk2pGU4GarppLtLbtWhG9W8EhTqPubm9ho7MoT6z09v+p+3UKu8yxoJCgNjaW4SAjExEhoKGAoDY2luEhE1MDAwMDAwMDAwMDAwMDAwMBpqCibrWumHIQOSGvBAKDWGJV1OcuIbs7MNxZhCLyoM6sIgvcY39Vcw1xJA1HdvhvAMNBFVZXm4LjmC2BEy41H/GBqcPrlnvEE5XOUQeSUMrnfGu6Hfxhj8se+3eeFZMuPHVZg1DvobWzAE4iIwdGVzdGluZyB0cmFuc2ZlciB3aXRoIDExY2luLCBmcm9tIGFjYy05IHRvIGFjYy04"
}
```
#### 6.2.6 Subscribe
Subscribe for events

To tell which events you want, you need to provide a query. query is a
string, which has a form: "condition AND condition ..." (no OR at the
moment). 

Examples:

- tm.event = 'NewBlock' # new blocks
- tm.event = 'CompleteProposal' # node got a complete proposal
- tm.event = 'Tx' AND tx.hash = 'XYZ' # single transaction
- tm.event = 'Tx' AND tx.height = 5 # all txs of the fifth block
- tx.height = 5 # all txs of the fifth block


#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| query  | string | false | true    | query is a string, which has a form: "condition AND condition ..." (no OR at the moment).   |


#### Return Type
```
type ResultSubscribe struct {
    Results *state.Responses
}
```

#### Example
Run the command:
```
curl 'http://localhost:26657/subscribe'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {}
}
```

#### 6.2.7 UnsubscribeAll
Unsubscribe from all events


#### Return Type
```
type ResultUnsubscribeAll struct {
    Results *state.Responses
}
```

#### Example
Run the command:
```
curl 'http://localhost:26657/unsubscribe_all'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {}
}
```
#### 6.2.8 Unsubscribe
Unsubscribe from event

#### Parameters
| Name | Type | Default | Required | Description                 |
| ---- | ---- | ------- | -------- | --------------------------- |
| query  | string | false | true    | query is a string, which has a form: "condition AND condition ..." (no OR at the moment).  |


#### Return Type
```
type ResultUnsubscribe struct {
    Results *state.Responses
}
```

#### Example
Run the command:
```
curl 'http://localhost:26657/unsubscribe'
```

The above command returns JSON structured like this:
```
{
  "jsonrpc": "2.0",
  "id": "",
  "result": {}
}
```








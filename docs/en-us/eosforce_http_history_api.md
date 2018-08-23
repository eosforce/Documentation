# history api

- Protocol: http
- Bash Path: /v1/history
- Request Method: POST

## /get_actions

BODY PARAMS	 :
- pos : int32
- offset : int32
- limit : int32 
- account_name : string

```json
{
    "actions": [
        {
            "global_action_seq": 1735208,
            "account_action_seq": 1733608,
            "block_num": 1151135,
            "block_time": "2018-08-01T06:21:55.000",
            "action_trace": {
                "receipt": {
                    "receiver": "eosio",
                    "act_digest": "c3a23dbfb318174c812a3f91be7280c33dbcd25bfb95c0f566d94deda2b9b422",
                    "global_sequence": 1735418,
                    "recv_sequence": 1733609,
                    "auth_sequence": [
                        [
                            "uuu",
                            156
                        ]
                    ],
                    "code_sequence": 1,
                    "abi_sequence": 1
                },
                "act": {
                    "account": "eosio",
                    "name": "newaccount",
                    "authorization": [
                        {
                            "actor": "uuu",
                            "permission": "active"
                        }
                    ],
                    "data": "0000000000001ac6000000404d1a293d0100000001000251c4453442961fbdc969fd4fa6aaaaaaaa31752cf7eaf5d86782803f4f4ca6010000000100000001000251c4453442961fbdc969fd4fa6a617369eb431752cf7eaf5d86782803f4f4ca601000000"
                },
                "elapsed": 177,
                "cpu_usage": 0,
                "console": "",
                "total_cpu_usage": 0,
                "trx_id": "d6cdb13b56f77313637b7e50efs83cdebd2c88420c1ecfa4de367f70e1a2929",
                "inline_traces": []
            }
        }
    ],
    "last_irreversible_block": 1124414
}
```

## /get_transaction
BODY PARAMS	
- id : int32

```json
{
    "id": "100004bf44d5cc60fe0697b37de830809bef3c2fa0438c38705992f649b97eb6",
    "trx": null,
    "block_time": "2018-07-01T08:32:09.000",
    "block_num": 264171,
    "last_irreversible_block": 1154459,
    "traces": []
}
```

## /get_key_accounts
get all accounts of this key

BODY PARAMS	
- public_key : string

```json
{
    "account_names":["god"]
}
```

## /get_controlled_accounts

BODY PARAMS	
- controlling_account : string

# Create account

## cleos command

```bash
cleos create account [OPTIONS] creator name OwnerKey [ActiveKey]
```

Positionals:
  - creator TEXT                The name of the account creating the new account (required)
  - name TEXT                   The name of the new account (required)
  - OwnerKey TEXT               The owner public key for the new account (required)
  - ActiveKey TEXT              The active public key for the new account

## abi data structure

```json
    {
      "name": "newaccount",
      "base": "",
      "fields": [
        {"name":"creator", "type":"account_name"},
        {"name":"name",    "type":"account_name"},
        {"name":"owner",   "type":"authority"},
        {"name":"active",  "type":"authority"}
      ]
    }

```
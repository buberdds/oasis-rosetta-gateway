[![CI badge](https://github.com/oasisprotocol/oasis-core-rosetta-gateway/workflows/Continuous%20integration/badge.svg)](https://github.com/oasisprotocol/oasis-core-rosetta-gateway/actions?query=workflow%3A%22Continuous+integration%22+branch%3Amaster)

# Oasis Gateway for Rosetta

This repository implements the [Rosetta][1] server for the [Oasis][0] network.
See the [Rosetta API docs][3] for information on how to use the API.
Oasis-specific Rosetta API information is given in the "Oasis-specific
information" subsection below.

## Building and testing

To build the server:

	make

To run tests:

	make test

To clean-up:

	make clean


`make test` will automatically download the [Oasis node][0] and [rosetta-cli][2],
set up a test Oasis network, make some sample transactions, then run the
gateway and validate it using `rosetta-cli`.

## Running the gateway

The gateway connects to an Oasis node, so make sure you have a running node
first.  If you don't have an Oasis node yet, follow the [instructions for running an Oasis node][4].

Set the `OASIS_NODE_GRPC_ADDR` environment variable to the node's gRPC socket
address (e.g. `unix:/path/to/node/internal.sock`).

Optionally, set the `OASIS_ROSETTA_GATEWAY_PORT` environment variable to the
port that you want the gateway to listen on (default is 8080).

Start the gateway simply by running the executable `oasis-core-rosetta-gateway`.


[0]: https://github.com/oasisprotocol/oasis-core
[1]: https://github.com/coinbase/rosetta-sdk-go
[2]: https://github.com/coinbase/rosetta-cli
[3]: https://www.rosetta-api.org/
[4]: https://docs.oasis.dev/general/operator-docs/running-a-node



## Oasis-specific information
This section describes how Oasis fits into the Rosetta APIs.

### Network identifier
Rosetta reference: https://www.rosetta-api.org/docs/api_identifiers.html#network-identifier

For Amber (at time of writing):

```js
{
    "blockchain": "Oasis",
    "network": "c014bda208f670539e8f03016b0dcfe16e0c2ad9a060419d1aad580f5c7ff447"
    /* no sub_network_identifier */
}
```

In general (e.g., for other testnets), the `.network` string is the lowercase hex encoded SHA-512/256 hash of the CBOR encoded genesis document.

### Account identifier
Rosetta reference: https://www.rosetta-api.org/docs/api_identifiers.html#account-identifier

#### General account
For an account `account_addr`'s (e.g. `oasis1qzzd6khm3acqskpxlk9vd5044cmmcce78y5l6000`) general account:

```js
{
    "address": account_addr
    /* no sub_account */
    /* no metadata */
}
```

#### Escrow account
For an account `account_addr`'s (e.g. `oasis1qzzd6khm3acqskpxlk9vd5044cmmcce78y5l6000`) escrow account:

```js
{
    "address": account_addr,
    "sub_account": {
        "address": "escrow"
        /* no metadata */
    }
    /* no metadata */
}
```

#### Common pool
For the common pool:

```js
{
    "address": "oasis1qrmufhkkyyf79s5za2r8yga9gnk4t446dcy3a5zm"
    /* no sub_account */
    /* no metadata */
}
```

#### Fee accumulator
For the fee accumulator:

```js
{
    "address": "oasis1qqnv3peudzvekhulf8v3ht29z4cthkhy7gkxmph5"
    /* no sub_account */
    /* no metadata */
}
```

### Currency
Rosetta reference: https://www.rosetta-api.org/docs/api_objects.html#currency

#### ROSE
For ROSE:

```js
{
    "symbol": "ROSE",
    "decimals": 9
    /* no metadata */
}
```

### Transaction intents
Rosetta reference: https://www.rosetta-api.org/docs/ConstructionApi.html#constructionpreprocess and https://www.rosetta-api.org/docs/ConstructionApi.html#constructionpayloads

The first two operations in the listings are the gas fee payment.
For zero-fee transactions, omit them and decrease the remaining operation identifier indices.

#### Staking transfer
For transfer, `amount_bu` base units from `signer_addr` to `to_addr` with gas limit `gas_limit` and fee `fee_bu` base units:

```js
[
    {
        "operation_identifier": {
            "index": 0
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": signer_addr
            /* no sub_account */
            /* no metadata */
        },
        "amount": {
            "value": "-" + fee_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        },
        /* no coin_change */
        "metadata": {
            "fee_gas": gas_limit.toString()
        }
    },
    {
        "operation_identifier": {
            "index": 1
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": "oasis1qqnv3peudzvekhulf8v3ht29z4cthkhy7gkxmph5" /* fee accumulator */
            /* no sub_account */
            /* no metadata */
        },
        "amount": {
            "value": fee_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        }
        /* no coin_change */
        /* no metadata */
    },
    {
        "operation_identifier": {
            "index": 2
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": signer_addr
            /* no sub_account */
            /* no metadata */
        },
        "amount": {
            "value": "-" + amount_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        }
        /* no coin_change */
        /* no metadata */
    },
    {
        "operation_identifier": {
            "index": 3
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": to_addr
            /* no sub_account */
            /* no metadata */
        },
        "amount": {
            "value": amount_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        }
        /* no coin_change */
        /* no metadata */
    }
]
```

#### Staking burn
For burn, `amount_bu` base units from `signer_addr` with gas limit `gas_limit` and fee `fee_bu` base units:

```js
[
    {
        "operation_identifier": {
            "index": 0
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": signer_addr
            /* no sub_account */
            /* no metadata */
        },
        "amount": {
            "value": "-" + fee_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        },
        /* no coin_change */
        "metadata": {
            "fee_gas": gas_limit.toString()
        }
    },
    {
        "operation_identifier": {
            "index": 1
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": "oasis1qqnv3peudzvekhulf8v3ht29z4cthkhy7gkxmph5" /* fee accumulator */
            /* no sub_account */
            /* no metadata */
        },
        "amount": {
            "value": fee_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        }
        /* no coin_change */
        /* no metadata */
    },
    {
        "operation_identifier": {
            "index": 2
            /* no network_index */
        },
        /* no related_operations */
        "type": "Burn",
        /* no status */
        "account": {
            "address": signer_addr
            /* no sub_account */
            /* no metadata */
        },
        "amount": {
            "value": "-" + amount_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        }
        /* no coin_change */
        /* no metadata */
    }
]
```

#### Staking add escrow
For add escrow, `amount_bu` base units from `signer_addr` to `escrow_addr` with gas limit `gas_limit` and fee `fee_bu` base units:

```js
[
    {
        "operation_identifier": {
            "index": 0
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": signer_addr
            /* no sub_account */
            /* no metadata */
        },
        "amount": {
            "value": "-" + fee_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        },
        /* no coin_change */
        "metadata": {
            "fee_gas": gas_limit.toString()
        }
    },
    {
        "operation_identifier": {
            "index": 1
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": "oasis1qqnv3peudzvekhulf8v3ht29z4cthkhy7gkxmph5" /* fee accumulator */
            /* no sub_account */
            /* no metadata */
        },
        "amount": {
            "value": fee_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        }
        /* no coin_change */
        /* no metadata */
    },
    {
        "operation_identifier": {
            "index": 2
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": signer_addr
            /* no sub_account */
            /* no metadata */
        },
        "amount": {
            "value": "-" + amount_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        }
        /* no coin_change */
        /* no metadata */
    },
    {
        "operation_identifier": {
            "index": 3
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": escrow_addr,
            "sub_account": {
                "address": "escrow"
                /* no metadata */
            }
            /* no metadata */
        },
        "amount": {
            "value": amount_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        }
        /* no coin_change */
        /* no metadata */
    }
]
```

#### Staking reclaim escrow
For transfer, `amount_sh` shares to `signer_addr` from `escrow_addr` with gas limit `gas_limit` and fee `fee_bu` base units:

```js
[
    {
        "operation_identifier": {
            "index": 0
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": signer_addr
            /* no sub_account */
            /* no metadata */
        },
        "amount": {
            "value": "-" + fee_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        },
        /* no coin_change */
        "metadata": {
            "fee_gas": gas_limit.toString()
        }
    },
    {
        "operation_identifier": {
            "index": 1
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": "oasis1qqnv3peudzvekhulf8v3ht29z4cthkhy7gkxmph5" /* fee accumulator */
            /* no sub_account */
            /* no metadata */
        },
        "amount": {
            "value": fee_bu.toString(),
            "currency": {
                "symbol": "ROSE",
                "decimals": 9
                /* no metadata */
            }
            /* no metadata */
        }
        /* no coin_change */
        /* no metadata */
    },
    {
        "operation_identifier": {
            "index": 2
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": signer_addr
            /* no sub_account */
            /* no metadata */
        },
        /* no amount */
        /* no coin_change */
        /* no metadata */
    },
    {
        "operation_identifier": {
            "index": 3
            /* no network_index */
        },
        /* no related_operations */
        "type": "Transfer",
        /* no status */
        "account": {
            "address": escrow_addr,
            "sub_account": {
                "address": "escrow"
                /* no metadata */
            }
            /* no metadata */
        },
        /* no amount */
        /* no coin_change */
        "metadata": {
            "reclaim_escrow_shares": amount_sh.toString()
        }
    }
]
```

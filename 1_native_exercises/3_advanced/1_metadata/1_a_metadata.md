
## Setup

To create a transaction metadata using the `cardano-cli`, you must first create a **payment key-pair** and a **wallet address** if you haven't already.

** Create Payment Keys **

```bash
cardano-cli address key-gen \
--verification-key-file payment.vkey \
--signing-key-file payment.skey
```

** Create Wallet Address **

```bash
cardano-cli address build \
--payment-verification-key-file payment.vkey \
--out-file payment.addr \
$TESTNET
```

Now that you have a **wallet address**, you can now request some `tAda` funds from the [testnet faucet](https://testnets.cardano.org/en/testnets/cardano/tools/faucet/). 

Once you have some funds, we can now create the sample metadata that we want to store into the blockchain.

We start by creating a `metadata.json` file with the following content:

```json
{
    "20220101": {
        "name": "hello world",
        "completed": 0
    }
}
```

:::note

Based on our theoretical **To-Do List** application, this `JSON` shape could be a way to insert / update entries into our list. We choose an arbitrary number (`20220101`) as the key; we are basically saying that all metadata that will be inserted with that key is related to the **To-Do List** application data. Although we don't have control over what will be inserted with that metadata key since **Cardano** is an open platform.

:::

Now that we have our `JSON` data, we can create a transaction and embed the metadata into the transaction. Ultimately storing it into the **Cardano** blockchain forever.

## Query UTXO

The next step is to query the available **UTXO** from our **wallet address**:

```bash
cardano-cli query utxo --address $(cat payment.addr) $TESTNET
```

You should see something like this:

```
                           TxHash                                 TxIx        Amount
--------------------------------------------------------------------------------------
dfb99f8f103e56a856e04e087255dbaf402f3801acb71a6baf423a1054d3ccd5     0        1749651926 lovelace
```

Here we can see that our **wallet address** contains some spendable `lovelace` with the `TxHash: dfb99f8f103e56a856e04e087255dbaf402f3801acb71a6baf423a1054d3ccd5` and `TxIndex: 0`. We can then use it to pay for the transaction fee when we store our data on the blockchain.

## Submit to blockchain

Next, we create a draft transaction with the metadata embedded into it using the `TxHash` and `TxIndex` result from our last query.

```bash {2}
cardano-cli transaction build-raw \
--babbage-era \
--tx-in dfb99f8f103e56a856e04e087255dbaf402f3801acb71a6baf423a1054d3ccd5#0 \
--tx-out $(cat payment.addr)+0 \
--metadata-json-file metadata.json \
--fee 0 \
--out-file tx.draft
```

Then we calculate the transaction fee like so:

```bash
cardano-cli transaction calculate-min-fee \
--tx-body-file tx.draft \
--tx-in-count 1 \
--tx-out-count 1 \
--witness-count 1 \
$TESTNET \
--protocol-params-file protocol.json
```

You should see something like this:

```bash
171793 Lovelace
```

With that, We build the final transaction with the total amount of our wallet minus the calculated fee as the total output amount. `1749651926 - 171793 = 1749480133`

```bash {3}
cardano-cli transaction build-raw \
--babbage-era \
--tx-in dfb99f8f103e56a856e04e087255dbaf402f3801acb71a6baf423a1054d3ccd5#0 \
--tx-out $(cat payment.addr)+1749480133 \
--metadata-json-file metadata.json \
--fee 171793 \
--out-file tx.draft
```

We then sign the transaction using our **payment signing key**:

```bash
cardano-cli transaction sign \             
--tx-body-file tx.draft \
--signing-key-file payment.skey \
$TESTNET \
--out-file tx.signed 
```

Finally, we submit the signed transaction to the blockchain:


```bash
cardano-cli transaction submit \
--tx-file tx.signed \    
$TESTNET
```

Congratulations, you are now able to submit **Cardano** transactions with metadata embedded into them. 

## Retrieving Metadata 

There are many ways to retrieve metadata stored in the Cardano blockchain. 


### Blockfrost

Blockfrost provides an API to access the Cardano blockchain fast and easily.

To retrieve metadata using Blockfrost, we call a specific endpoint for transaction metadata that they provide.

#### NOTE: the path that you use for the API query will change based upon the network as below
 - Cardano preprod	https://cardano-preprod.blockfrost.io/api/v0
 - Cardano preview	https://cardano-preview.blockfrost.io/api/v0

#### Query key 20220101 Metadata on Preview network 

    curl -H 'project_id: <PUT_API_KEY_HERE>' https://cardano-preview.blockfrost.io/api/v0/metadata/txs/labels/20220101 | jq
    
You should see something like this:

    [
        {
            "tx_hash": "96b888bb22f35ff2ac8fcef9bd0a1f25eefcb149813fc5aefef24461648d42b8",
            "json_metadata": {
                "name": "hello Bharat",
                "completed": 100
            }
        },
        {
            "tx_hash": "a5af97df108c654055a0379a0cb4f1c0df83f2c55d00227edd9e02b300fb9068",
            "json_metadata": {
                "name": "hello world, greetings from the British Inland Waterways",
                "completed": 0
            }
        },
        {
            "tx_hash": "8e13278d8e8c20074ad9cf70eeda361a67d9747df8376b70e75e9e2dbad46f24",
            "json_metadata": {
                "name": "hello world from batch58-43",
                "completed": 0
            }
        }
    ]
 
This has been taken from the below sites for ease of reference.
1. https://developers.cardano.org/docs/transaction-metadata/how-to-create-a-metadata-transaction-cli 
2. https://github.com/cardano-foundation/developer-portal/blob/staging/docs/transaction-metadata/retrieving-metadata.md

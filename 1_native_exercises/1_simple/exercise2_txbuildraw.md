# Building and signing transactions using build-raw

Transactions vary in complexity, depending on their intended outcomes, but all transactions share a number of attributes:

* Input - contains funds that are spent by the transaction. It is simply the output of an earlier transaction. A transaction can have multiple inputs.
* Output - determine where the funds go to. An output is given by a payment address and an amount. A transaction can have multiple outputs.
* Payment address - an address that can receive payments, This is the only type of addresses that can be specified in a transaction output.
* Payment and stake key pairs - sets of files containing a public verification key and a private signing key.
* Invalid-before - The slot that the transaction is valid from. This can only be specified in the Allegra era and onwards.
* Invalid-hereafter - represents a slot, or deadline by which a transaction must be submitted. This is an absolute slot number, rather than a relative one, which means that the `--invalid-hereafter` value should be greater than the current slot number. A transaction becomes invalid at the invalid-hereafter slot.
* Era - Transactions can differ between the eras (e.g have new features such as multi-assets) so we must specify the era we are currently in.

To create a transaction in the shelley era onwards, we need to follow this process:

* Get the protocol parameters
* Draft the transaction
* Calculate the fee
* Define the validity interval for the transaction
* Build the transaction
* Sign the transaction
* Submit the transaction


**TESTNET ID**
save the environment variable TESTNET=--testnet-magic 1 / OR 2 ( based upon network)

**Protocol parameters**
Query and save the parameters in **protocolparams.json**

    cardano-cli query protocol-parameters \
    $TESTNET \
    --out-file protocolparams.json
    
**FIND A UTXO for the given addr1 which can be consumed**
**Copy-paste the transaction hash concatenated with the # symbol and index of the UTXO in question**

    cardano-cli query utxo --address $(cat addr1.addr) $TESTNET
    
                              TxHash                                 TxIx        Amount
    --------------------------------------------------------------------------------------
    35a1e638d1c8a1bf74e79ea514adfb8d24113d1391baad2dceace737af30185d     0        10000000000 lovelace + TxOutDatumNone

**append and save to env. variable, appending a # followed by the the Index (#0 in this case)**

    UTXO1=35a1e638d1c8a1bf74e79ea514adfb8d24113d1391baad2dceace737af30185d#0

**Draft the transaction**

In the draft `tx-out`, `ttl` and `fee` can be zero. Later we use the `out-file` `tx.draft` to calculate the `fee`

    cardano-cli transaction build-raw \
    --babbage-era \
    --tx-in $UTXO1 \
    --tx-out $(cat addr2.addr)+2500000000 \
    --tx-out $(cat addr1.addr)+0 \
    --invalid-hereafter 0 \
    --fee 0 \
    --out-file tx.draft

**Calculate the fee**

Use `tx.draft` as `tx-body-file`. **Witnesses** are the amount of keys that must sign the transaction.

    cardano-cli transaction calculate-min-fee \
    --tx-body-file tx.draft \
    --tx-in-count 1 \
    --tx-out-count 2 \
    --witness-count 1 \
    $TESTNET \
    --protocol-params-file protocolparams.json

For example:

    > 167965

**Save the fee to a variable**

    FEE=167965

**Save the return balance to an environment variable**
This should be understood as (SUM OF INPUT UTXOs) - (SUM OF OUTPUT UTXOs) - Fee
In this case Input - sent amount - fee = 10000,000,000 lovelace - 2500,000,000 lovelace - 167965
```
    BALANCE=$(expr 10000000000 - 2500000000 - $FEE)
```
**Determine the validity interval**

When building and submitting a transaction in the shelley era you need to check the current tip of the blockchain, for example, if the tip is slot 4000, you should set the invalid-hereafter to (4000 + N slots), so that you have enough time to build and submit a transaction. Submitting a transaction with a validity interval set in the past would result in a tx error.
```
    cardano-cli query tip $TESTNET 
```
Look for the value of `SlotNo`

    {
        "epoch": 259,
        "hash": "dbf5104ab91a7a0b405353ad31760b52b2703098ec17185bdd7ff1800bb61aca",
        "slot": 26633911,
        "block": 5580350
    }

Therefore, if N = 600 slots (10 minutes)

**save this into an environment variable called VALIDTILL**
```
    VALIDTILL=$(expr 26633911 + 600)    
```
**echo $VALIDTILL    
26634511**

**Build the transaction**

This time we include all the parameters:
```
    cardano-cli transaction build-raw \
    --babbage-era \
    --tx-in $UTXO1 \
    --tx-out $(cat addr2.addr)+2500000000 \
    --tx-out $(cat addr1.addr)+$BALANCE \
    --invalid-hereafter $VALIDTILL \
    --fee $FEE \
    --out-file tx.raw
```
**Signing**

A transaction must prove that it has the right to spend its inputs. In the most common case, this means that a transaction must be signed by the signing keys belonging to the payment addresses of the inputs. If a transaction contains certificates, it must additionally be signed by somebody with the right to issue those certificates. For example, a stake address registration certificate must be signed by the signing key of the corresponding stake key pair.

```
    cardano-cli transaction sign \
    --tx-body-file tx.raw \
    --signing-key-file addr1.skey \
    $TESTNET \
    --out-file tx.signed
```

**Submit**
```
    cardano-cli transaction submit \
    --tx-file tx.signed \
    $TESTNET
```
**Get the transaction hash**
```
    cardano-cli transaction txid --tx-file tx.signed
```
**09e9d3223a30d4c45e9db26d2a97bb1afd0ffc36c5ed6888e2332bc9483a2a74**


## FUN ACTIVITY: Manually calculate the actual tx fee from the protocol parameters and the formula txFeeFixed + (txFeePerByte * TxSizeInBytes)
## and compare it to the fee calculated by the cardano-cli tool!
### Hint: Take the CBOR Hex, and calculate the size in bytes given that one hex digit is 4 bits, so two hex digits is 1 byte

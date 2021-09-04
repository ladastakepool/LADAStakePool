# How to mint your NFT on Cardano Foundation

# Introduction
   
   - NFT trends now is the most excited and effective on the world. Today, NFT is being widely applied in real life, namely the release of digital paintings, digital music, digital books and gaming, etc

   - The content of this post will teach you how to mint 1 NFT (1ADA) of an image (stored under IPFS) on Cardano Foundation

# Steps
As a pool owner, you can start to mint a NFT follow the steps below
1. Create the policy key by the command <br/>
    Note: if you already have the key at the air gap server, you can skip this step
    
    * cardano-cli address key-gen –verification-key-file policy.vkey –signing-key-file policy.skey <br/>
    
2. Create policy by commands<br/>
     touch policy.script && echo "" > policy.script<br/>
     echo "{" >> policy.script<br/>
     echo "  \"keyHash\": \"$(cardano-cli address key-hash --payment-verification-key-file policy.vkey)\"," >> policy.script 
    echo "  \"type\": \"sig\"" >> policy.script<br/>
    echo "}" >> policy.script<br/>
    
    * Then, you can verify your policy file<br/>
        cat policy.script<br/>
        The format should be<br/>
        {<br/>
          "keyHash": "2867788a868fb28231a736a3eb4420261bb42019dc3605dd83rrf88",<br/>
          "type": "sig"<br/>
        }
    cardano-cli address key-gen –verification-key-file policy.vkey –signing-key-file policy.skey

    * Get the policy id

      cardano-cli transaction policyid --script-file policy.script<br/>
        --> the policy id format string<br/>

    * The policy id used to mint the token by sample<br/>
    --mint="1 677d3bbe3e01eeab498cd8786f7d261d92bd6ecea12109a332e86374.AhaXuPic002"
    
3. Inquiry the transaction history<br/>
    cardano-cli query utxo –address xxxxxxx –mainnet<br/>
    xxxxxxx is the adress of your pool<br/>
    The result should be<br/>
    TxHash TxIx Amount5588c6b9011dbdf6e4139f56a252eadfa6744a21b221662d18dc08xxxxxxxxxx 0 385445197 lovelace + 1 8a7ab85ce0533d7ad7d6f8f120eb4486f9c69ab2261825xxxxxxxxxx.LADA01
    68da69657fa1d6c790e0b5001431e577c3310e50cc6441a3661a6exxxxxxxxxx 0 118000000 lovelace …

4. Upload the picture/ content that to mint NFT to the IPFS<br/>
        4.1 Login to blockfrost.io and create your project<br/>
        4.2 Use the API Key of the project to push the image file for your NFT,
        by the command<br/>
        curl "https://ipfs.blockfrost.io/api/v0/ipfs/add" -X POST -H "project_id: <API Key>" -F "file=@./theNFT.jpg"<br/>
        4.3
        Example: 
        https://ipfs.blockfrost.dev/ipfs/Qmb4XwM8qGXSfT6uXx4kAyeiWd9i2cQ4bUYaTimDoGfKCz<br/>
        4.4 Documents<br/>
        https://docs.blockfrost.io/#tag/IPFS-Add about 100 MB free<br/>
        https://pinata.cloud/documentation#GettingStarted about 1GB free<br/>
        

5. Prepare the json file for you NFT picture (mint) <br/>
    {
        “6868”: {
    “8a7ab85ce0533d7ad7d6f8f120eb4486f9c69ab2261825xxxxxxxxxx”: {
    “LADA”: {
    “image”: “ipfs://Qmb4XwM8qGXSfT6uXx4kAyeiWd9i2cQ4bUYaTimDoGfKCz”,
    “latitude”: “10.4888441”,
    “longitude”: “100.7920294”,
    “name”: “LADA”,
    “description”: “LADA NFT”
    }
    }
    }
    }

6. Build the transaction with the metadata<br/>

        cardano-cli query utxo --address $address --mainnet
    Your output should look something like this (fictional example):

                         TxHash                                      TxIx        Amount
--------------------------------------------------------------------------------------

    b35a4ba9ef3ce21adcd6879d08553642224304704d206c74d3ffb3e6eed3ca28     0        1000000000 lovelace
    
    Since we need each of those values in our transaction, we will store them individually in a corresponding variable.
    txhash="insert your txhash here"
    txix="insert your TxIx here"
    funds="insert Amount in lovelace here"
    policyid=$(cat policy/policyID)

7. Check if all of the other needed variables for the transaction are set:

        echo $fee
        echo $address
        echo $output
        echo $tokenamount
        echo $policyid
        echo $tokenname
        echo $slotnumber
        echo $script

8. Run this command to generate a raw transaction file.

        cardano-cli transaction build-raw \
        --fee $fee  \
        --tx-in $txhash#$txix  \
        --tx-out $address+$output+"$tokenamount $policyid.$tokenname" \
        --mint="$tokenamount $policyid.$tokenname" \
        --minting-script-file $script \
        --metadata-json-file metadata.json  \
        --invalid-hereafter $slotnumber \
        --out-file matx.raw <br/>
    As with every other transaction, we need to calculate the fee and the output and save them in the corresponding variables (currently set to zero).

9. Set the $fee variable.

        fee=$(cardano-cli transaction calculate-min-fee --tx-body-file matx.raw --tx-in-count 1 --tx-out-count 1 --witness-count 1 --mainnet --protocol-params-file protocol.json | cut -d " " -f1)<br/>
        And this command calculates the correct value for $output.<br/>
        output=$(expr $funds - $fee)<br/>
        
        With the newly set values, re-issue the building of the raw transaction.

        cardano-cli transaction build-raw \
        --fee $fee  \
        --tx-in $txhash#$txix  \
        --tx-out $address+$output+"$tokenamount $policyid.$tokenname" \
        --mint="$tokenamount $policyid.$tokenname" \
        --minting-script-file $script \
        --metadata-json-file metadata.json  \
        --invalid-hereafter $slotnumber \
        --out-file matx.raw
        Sign the transaction
        
        cardano-cli transaction sign  \
        --signing-key-file payment.skey  \
        --signing-key-file policy/policy.skey  \
        --mainnet --tx-body-file matx.raw  \
        --out-file matx.signed

10. Submit the transaction, therefore minting our native assets:

        cardano-cli transaction submit --tx-file matx.signed --mainnet
        
!! Congratulations, we have now successfully minted our own token. After a couple of seconds, we can check the output address

            cardano-cli query utxo --address $address --mainnet 😉

# Documents
https://github.com/ladastakepool/LADAStakePool/blob/main/NFT_Mint_Guide.md<br/>
https://developers.cardano.org/docs/native-tokens/minting-nfts<br/>

        
    
    
    
    
    

                  
                    
            
        
        
    
    
            
     
    
            
            
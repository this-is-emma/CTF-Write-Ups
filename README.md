# CTF-Write-Ups
A repo to stores write ups for CTFs challenges 

# [HTB business CTF 2023 - The Great Escape](https://www.hackthebox.com/events/htb-business-ctf-2023)

## Lazy Ballot

| Category | Tools used |
| -- | -- |
|Web|Burpsuite|

Teammate pointed out that this is a noSQL db. Searched online for noSQL injections payloads, specifically for couchDB.

Found payload: `{"$ne":""}` (credit to Cyber Mentor's YouTube video titled: Find and exploit NoSQL injection) and tried it in burp in the repeater tab and it worked 🎉.

To access admin account, in Burpsuite intercept the log in requets and change the password to the previously mentioned payload then forward request and you will be successfully redirected to the dashboard. 

Found the flag at the last page.

<img width="603" alt="screenshot2" src="https://github.com/this-is-emma/CTF-Write-Ups/assets/8417822/5f9bd4b8-5338-4cac-8902-685ede1084ca">



## Paid Cont-actor

| Category | Tools used |
| -- | -- |
|Blockchain|Python script ✚ Netcat |

Challenge provides an address with 2 ports. Using netcat, one port provide some information while the other is unresponsive. Per the instruction provided, the port that is unresponsive is the rpc url
 
Using netcat to interact with the responsive port, we are provided with a private key, address, target contract and setup contract


Also, the challenge files provide contract.col and Setup.sol files which after inspection show that the goal of the challenge is to change the contract status to signed 
After some googling and following the instructions provided in the readme, I created a Python script to interact with the blockchain network and sign the contract. See the script below: 

```
from web3 import Web3
from web3.middleware import geth_poa_middleware

# Connection details
rpc_url = "http://94.237.52.136:56575"
private_key = "0x24af31171e5fce72eef50a93a04638a2d0c403eae8882057f78feb74382ce7f4"
address = "0x96db456e6A1Ee7ac458b4BBb30e5F870709E4ccF"
target_contract_address = "0xC511CC742eD9A61A9775d50d49D512B07445A43C"
setup_contract_address = "0xa712a9914B7a6C639d5d331CAbcae010330012BA"

# Load contract ABI
contract_abi = [
    {
        "constant": False,
        "inputs": [
            {"name": "signature", "type": "uint256"}
        ],
        "name": "signContract",
        "outputs": [],
        "payable": False,
        "stateMutability": "nonpayable",
        "type": "function"
    },
    {
        "constant": True,
        "inputs": [],
        "name": "signed",
        "outputs": [
            {"name": "", "type": "bool"}
        ],
        "payable": False,
        "stateMutability": "view",
        "type": "function"
    }
]

# Create a Web3 instance
w3 = Web3(Web3.HTTPProvider(rpc_url))
w3.middleware_onion.inject(geth_poa_middleware, layer=0)

# Load the contract
contract = w3.eth.contract(address=target_contract_address, abi=contract_abi)

# Build and sign the transaction
nonce = w3.eth.get_transaction_count(address)
tx = contract.functions.signContract(1337).build_transaction({
    "from": address,
    "nonce": nonce,
    "gas": 100000,
    "gasPrice": w3.to_wei("10", "gwei")
})
signed_tx = w3.eth.account.sign_transaction(tx, private_key=private_key)

# Send the signed transaction
tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)

# Wait for the transaction to be mined
receipt = w3.eth.wait_for_transaction_receipt(tx_hash)

# Check if the signContract function succeeded
signed = contract.functions.signed().call()
print("signed:", signed)

```

After running the script, interacting with port again and selecting option 3 gives us the flag 🎉

<img width="625" alt="screen" src="https://github.com/this-is-emma/CTF-Write-Ups/assets/8417822/6ae0169f-2c54-42ae-bfa8-6dea135f1427">



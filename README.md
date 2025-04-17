# Assuming you are using a Python library like xrpl-py

from xrpl.clients import JsonRpcClient
from xrpl.models.transactions import Payment
from xrpl.wallet import Wallet
from xrpl.utils import xrp_to_drops

# Replace with your secret key (the sending account) - KEEP THIS PRIVATE!
YOUR_SECRET = "YOUR_SENDING_ACCOUNT_SECRET_KEY"
# Derive the wallet from the secret key
wallet = Wallet.from_seed(YOUR_SECRET)
sending_address = wallet.address

# Robinhood destination address
destination_address = "rEAKseZ7yNgaDuxH74PkqB12cVWohpi7R6"

# Robinhood memo tag
memo_value = "2826867381"

# Amount to send (in XRP)
amount_xrp = "10"

# Connect to an XRP Ledger node
client = JsonRpcClient("https://s1.ripple.com:51234") # Or another reliable public/private node

try:
    # Construct the Payment transaction
    payment = Payment(
        account=sending_address,
        destination=destination_address,
        amount=xrp_to_drops(amount_xrp), # Convert XRP to drops (1 XRP = 1,000,000 drops)
        memos=[
            {
                "Memo": {
                    "MemoData": memo_value.encode().hex().upper(), # Memo data as hex
                    "MemoFormat": "1", # Text UTF-8
                    "MemoType": "6"   # Account ID
                }
            }
        ]
    )

    # Sign the transaction
    signed_tx = wallet.sign(payment)

    # Submit the transaction
    tx_response = client.submit(signed_tx.tx_blob)
    print(f"Transaction submitted: {tx_response}")

except Exception as e:
    print(f"An error occurred: {e}")

finally:
    if 'client' in locals() and client.is_open():
        client.close()

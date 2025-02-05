# Uniswap v4 Python Token Swapper – Universal Router

A **user-friendly Python script** that demonstrates how to execute **SWAP_EXACT_IN** (Exact Input) swaps on **Uniswap v4 Universal Router** across various L2 networks, leveraging the **uniswap-universal-router-decoder** library.

---

## Features
- **Supports** Uniswap v4 Universal Router.
- **Works** with both v3 and v4 pools.
- **Auto-selects** the router address based on the connected chain (Base, Optimism, Polygon, Arbitrum, Ethereum).
- **Permit2 integrated** for simplified token approvals.
- **Checks & Cancels** stuck transactions automatically.
- Written in **Python** using `web3.py`, `eth_account`, and `uniswap-universal-router-decoder`.

---

## Installation

1. **Clone This Repo** (or download the script directly)
2. **Install Dependencies**:

```bash
pip install web3 eth_account eth_abi uniswap-universal-router-decoder
```

3. **(Optional) Create a Virtual Environment**
```bash
python -m venv venv
source venv/bin/activate  # On Linux/Mac
# or venv\Scripts\activate on Windows
```

4. **Ensure You Have Python 3.8+**
```bash
python --version
```

---

## Quick Start

```python
from web3 import Web3
from uniswap_swaps import Uniswap  # or whatever you name this script

# 1. Connect to an L2 or mainnet RPC
provider_url = "https://base-mainnet-rpc.example"  # Replace with your provider
web3 = Web3(Web3.HTTPProvider(provider_url))

# 2. Prepare wallet credentials
wallet_address = "0xYOUR_WALLET_ADDRESS"
private_key = "YOUR_PRIVATE_KEY"  # Keep secret!

# 3. Instantiate the Uniswap helper
uniswap = Uniswap(
    wallet_address=wallet_address,
    private_key=private_key,
    provider=provider_url,  # Used to auto-detect chain
    web3=web3
)

# 4. Perform a swap (SWAP_EXACT_IN)
# Example: Swap 1 tokenIn -> tokenOut at 0.3% fee in a v3 pool
amount_in_wei = web3.to_wei(1, 'ether')

try:
    tx_hash = uniswap.make_trade(
        from_token="0xTOKEN_IN_ADDRESS",
        to_token="0xTOKEN_OUT_ADDRESS",
        amount=amount_in_wei,
        fee=3000,         # e.g., 3000 for a 0.3% Uniswap V3 pool
        slippage=0.5,     # 0.5% slippage tolerance
        pool_version="v3"  # can be "v3" or "v4"
    )
    print(f"Swap transaction sent! Tx hash: {tx_hash.hex()}")
except Exception as e:
    print(f"Swap failed: {e}")
```

---

## How It Works

1. **Router Detection:** Based on your `provider_url`, the script automatically selects the correct **Uniswap v4 Universal Router** address for Base, Optimism, Polygon, Arbitrum, or Ethereum.
2. **Permit2 Approvals:** Checks and (if necessary) approves the Uniswap Permit2 contract before swapping, minimizing repeated approvals.
3. **Transaction Construction:** Leverages `uniswap-universal-router-decoder` to encode swap calls. Handles `v3_swap_exact_in` or `v4_swap` logic depending on `pool_version`.
4. **Transaction Submission:** Signs and sends a **type=2** (EIP-1559) transaction with dynamic gas.
5. **Stuck Transactions Check:** If a nonce is stuck, attempts to cancel it by sending a 0-value tx with a higher gas.

---

## Code Overview

- **`Uniswap` class**
  - **`__init__`:** Initializes Web3, detects chain, sets up router.
  - **`approve_permit2`:** Approves the Permit2 contract with infinite allowance.
  - **`check_permit2_allowance`:** Checks if there’s already sufficient allowance.
  - **`create_permit_signature`:** Generates a Permit2 signature for the swap.
  - **`make_trade`:** The main function for executing swaps.
  - **`check_for_stuck_transactions`:** Detects if a transaction nonce is stuck in pending.
  - **`cancel_transaction`:** Attempts to cancel a stuck transaction by overriding its nonce.
  - **`calculate_gas_parameters`:** Estimates baseFee and priorityFee, calculates total cost.

---

## Supported Chains (with Addresses)
| Chain     | Router Address                                |
|-----------|-----------------------------------------------|
| Ethereum  | `0x66a9893cc07d91d95644aedd05d03f95e1dba8af`   |
| Base      | `0x6ff5693b99212da76ad316178a184ab56d299b43`   |
| Optimism  | `0x851116d9223fabed8e56c0e6b8ad0c31d98b3507`   |
| Polygon   | `0x1095692a6237d83c6a72f3f5efedb9a670c49223`   |
| Arbitrum  | `0xa51afafe0263b40edaef0df8781ea9aa03e381a3`   |

> **Note:** These addresses may change if Uniswap deploys updates.

---

## Troubleshooting
- **Web3 Connection Failed:** Make sure your RPC URL is correct and the network is live.
- **Insufficient Balance or Allowance:** Ensure you have enough tokens/ETH for gas and correct Permit2 approvals.
- **Stuck Nonce:** The script automatically checks and attempts to cancel.
- **Unsupported Chain:** Add your chain to `ROUTER_ADDRESSES` or specify `ethereum`.

---

## Disclaimer
- **Use at your own risk** – This is experimental code for demonstration purposes.
- Always **test on a testnet** or with small amounts before using in production.
- Make sure you **keep your private keys secure**.

---

## Contributing
Pull requests are welcome! If you want to expand functionality or fix issues, feel free to open a PR.

---

## License
This project is released under the **MIT License**.

---

### Credits
- Built on top of [`uniswap-universal-router-decoder`](https://github.com/chain-cogs/uniswap-universal-router-decoder) for encoding/decoding logic.
- Thanks to [Web3.py](https://github.com/ethereum/web3.py/) and [Eth Account](https://github.com/ethereum/eth-account).



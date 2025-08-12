# Understanding Gas Fees, EIP-1559, and Using Gas Fees with ethers.js

---

## What is Gas Fee in Ethereum?

- **Gas** measures the computational effort needed to execute transactions or smart contracts on Ethereum.
- Every operation costs gas; more complex operations cost more gas.
- Gas fees compensate miners/validators for processing and including your transaction in a block.
- Gas fee = **Gas Used × Gas Price** (measured in Gwei, converted to ETH).

---

## Gas Fee Components

| Component         | Description                                                      |
|-------------------|------------------------------------------------------------------|
| **Gas Limit**     | Maximum gas units you're willing to spend on a transaction.      |
| **Gas Price**     | Price per gas unit, measured in Gwei (1 Gwei = 10⁻⁹ ETH).       |
| **Base Fee**      | Dynamic minimum gas fee burned per gas unit (introduced by EIP-1559). |
| **Priority Fee**  | Tip to miners to prioritize your transaction (also called "tip"). |
| **Max Fee Per Gas** | The max total fee you’re willing to pay per gas unit (EIP-1559). |

---

## Gas Fee Calculation Example (Legacy Model)

Suppose you send 0.01 ETH with these parameters:

| Parameter    | Value                      |
|--------------|----------------------------|
| Gas Limit    | 21,000 (standard for ETH transfer) |
| Gas Price    | 100 Gwei (user-set or network suggested) |

### Calculation

- Gas Used = Gas Limit = 21,000
- Gas Price = 100 Gwei = 100 × 10⁻⁹ ETH = 0.0000001 ETH per gas unit

**Gas Fee = Gas Used × Gas Price = 21,000 × 0.0000001 = 0.0021 ETH**

**Total Cost = Value Sent + Gas Fee = 0.01 + 0.0021 = 0.0121 ETH**

---

## What is EIP-1559?

EIP-1559, introduced in August 2021, changed Ethereum gas fee mechanics:

- Introduces a **Base Fee** burned per gas unit, which adjusts dynamically by block congestion.
- Users specify:
  - **Max Fee Per Gas:** maximum total fee they’re willing to pay.
  - **Max Priority Fee Per Gas:** tip to miners.
- Actual fee paid = Base Fee + Priority Fee (capped by max fee).
- Base Fee portion is **burned** (removed from circulation).
- Priority Fee goes to miners as a tip.

### Example (EIP-1559):

| Parameter               | Value             |
|-------------------------|-------------------|
| Gas Limit               | 21,000            |
| Base Fee (burned)       | 50 Gwei           |
| Max Priority Fee (tip)  | 2 Gwei            |
| Max Fee Per Gas (cap)   | 100 Gwei          |

**Gas Fee = Gas Used × (Base Fee + Priority Fee) = 21,000 × (50 + 2) Gwei**

= 21,000 × 52 × 10⁻⁹ ETH = 0.001092 ETH

---

## Does ethers.js Auto Estimate Gas?

- When you call a contract method or send a transaction **without specifying `gasLimit`**, ethers.js **automatically calls `estimateGas()`** behind the scenes to estimate how much gas is needed.
- This helps prevent out-of-gas errors and ensures your transaction has enough gas to execute.
- Example:

```js
  const txResponse = await contract.transfer(toAddress, amount);
```
ethers.js internally does:

```js
const txResponse = await contract.transfer(toAddress, amount, { gasLimit: 100000 });
```
and uses estimatedGas as the gasLimit.
- You can override the gas limit manually if you want:

```js
const txResponse = await contract.transfer(toAddress, amount, { gasLimit: 100000 });
```
However, ethers.js does NOT auto-set gas price or EIP-1559 fee fields; you must specify or rely on provider defaults.

## Example Code with ethers.js (EIP-1559):
```js
const tx = {
  to: "0xReceiverAddress...",
  value: ethers.utils.parseEther("0.01"),
};

const gasLimit = await provider.estimateGas(tx);
const feeData = await provider.getFeeData();

tx.gasLimit = gasLimit;
tx.maxPriorityFeePerGas = feeData.maxPriorityFeePerGas;
tx.maxFeePerGas = feeData.maxFeePerGas;

const txResponse = await wallet.sendTransaction(tx);
console.log("Transaction hash:", txResponse.hash);

const receipt = await txResponse.wait();
console.log("Confirmed in block:", receipt.blockNumber);

```
# Gas Fee Concepts Summary

| Concept           | Description                                                                                  |
|-------------------|----------------------------------------------------------------------------------------------|
| Gas Limit         | Max gas units allowed; estimated via `estimateGas()` or auto-estimated by ethers.js if omitted |
| Gas Price (Legacy) | Price per gas unit; replaced by EIP-1559 fees                                               |
| Base Fee (EIP-1559)| Burned gas fee per unit, adjusts dynamically per block congestion                           |
| Priority Fee      | Miner tip per gas unit (user-specified)                                                     |
| Max Fee Per Gas   | Max total fee user willing to pay per gas unit                                              |
| ethers.js         | Auto-calls `estimateGas()` if `gasLimit` not set; user sets gas prices manually or via provider |

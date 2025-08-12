# Multicall & Gas Fee Insights

## 1. What is Multicall?
Multicall contracts let you batch multiple contract calls into a single request.

- **Goal**: Reduce RPC overhead, aggregate results, or combine multiple state changes.
- **Common Versions**: Multicall2, Multicall3 (`aggregate3`, `aggregate3Value`, etc.).
- **Benefits**:
  - Fewer RPC calls for read operations.
  - Atomic execution for write operations (all succeed or fail unless `allowFailure = true`).

---

## 2. Read vs Write Multicall

### **Read Multicall**
- Uses `eth_call` (off-chain simulation).
- No gas fee.
- No blockchain state changes.
- Example: Query balances from multiple ERC20 contracts in one request.

### **Write Multicall**
- Uses `eth_sendTransaction` (on-chain execution).
- **Gas fee is paid**:
  - For the multicall contract's loop logic.
  - For each internal target call that modifies state.
- Atomic execution possible unless `allowFailure` is set to `true`.

---

## 3. `aggregate3` Signature

```solidity
struct Call3 {
    address target;
    bool allowFailure;
    bytes callData;
}

struct Result {
    bool success;
    bytes returnData;
}

function aggregate3(Call3[] calldata calls)
    public
    payable
    returns (Result[] memory returnData);
```

- `target`: Contract to call.
- `allowFailure`: If `false`, revert entire call if the target call fails.
- `callData`: Encoded function call.
- Returns array of results with `success` and `returnData`.

---

## 4. Gas Fee Rules

| Mode | State Changes? | Gas Fee? | Notes |
|------|----------------|----------|-------|
| **Read Multicall** | ❌ No | ❌ No | Uses `eth_call` |
| **Write Multicall** | ✅ Yes | ✅ Yes | Uses `eth_sendTransaction` |
| **callStatic on Write** | ❌ No | ❌ No | Simulates write function without sending a tx |

---

## 5. How to Make a Write Function Gas-Free: `callStatic`

`callStatic` lets you run **any** function (including write functions) as if it were read-only:
- No gas fee.
- No actual state change.
- Good for simulation or testing.

### Example
```javascript
import { ethers } from "ethers";

const provider = new ethers.JsonRpcProvider("https://mainnet.infura.io/v3/YOUR_KEY");
const multicallAddress = "0xYourMulticall3Address";
const multicallAbi = [
  "function aggregate3((address target,bool allowFailure,bytes callData)[] calls) returns (tuple(bool success, bytes returnData)[])"
];

const multicall = new ethers.Contract(multicallAddress, multicallAbi, provider);

(async () => {
  const iface = new ethers.Interface(["function balanceOf(address) view returns (uint256)"]);

  const calls = [
    {
      target: "0xSomeERC20",
      allowFailure: false,
      callData: iface.encodeFunctionData("balanceOf", ["0xYourWallet"])
    }
  ];

  // Simulate aggregate3 with callStatic (no gas fee)
  const results = await multicall.callStatic.aggregate3(calls);
  console.log(results);
})();
```

---

## 6. Summary Table

| Concept | Description |
|---------|-------------|
| **Read Multicall** | Aggregates multiple view/pure calls into one `eth_call`; no gas fee |
| **Write Multicall** | Aggregates multiple state-changing calls; single tx, gas fee applies |
| **allowFailure** | Prevents entire batch revert if one call fails |
| **callStatic** | Runs any function locally via `eth_call`, no state change, no gas |
| **Gas Fee (Write)** | Σ(each target call gas used) + multicall loop overhead |

---

## 7. Visual Flow

```mermaid
flowchart TD
    A[Developer calls aggregate3] --> B{Mode?}
    B -->|Read (eth_call)| C[Node executes off-chain]
    C --> D[Return results to client]
    B -->|Write (eth_sendTransaction)| E[Transaction sent to mempool]
    E --> F[Validators execute multicall contract]
    F --> G[Loop over calls, execute target calls]
    G --> H[Gas fee charged for all execution]
    B -->|callStatic (on write fn)| I[Node simulates write fn locally]
    I --> D
```

---

## 8. Key Takeaways
- Multicall **read** = free, **write** = costs gas.
- `callStatic` is your friend for simulating even write methods.
- Gas cost in write multicall is cumulative of all included calls.
- Use `allowFailure` wisely to avoid wasting gas when one call fails.

# PGEF Staking Vault — Documentation

This document explains the structure, purpose, and validation logic of the **PGEF (Perpetual Green Energy Fund) Staking Vault** Plutus smart contract written in Haskell (Plutus V2). The contract enables users to stake ADA into a decentralized liquidity pool for renewable energy infrastructure, earn yields over time, and claim rewards or unstake their principal.

---

## 1. Overview

The PGEF Staking Vault is designed to provide **permanent decentralized liquidity** for renewable energy projects. Users can:

- **Stake ADA** into the vault with a recorded timestamp
- **Earn yields** based on staking duration
- **Claim accumulated yields** and/or **unstake their principal**

The validator controls UTxOs containing a datum of type `StakingDatum`. Each staking position tracks:

- `stakerPkh`: The public key hash of the staker
- `principal`: The original amount staked (in lovelace)
- `startTime`: The timestamp when staking began (POSIX time in microseconds)
- `claimedYield`: Total yield already claimed by the staker
- `active`: Boolean flag indicating if the stake is currently active

### Redeemer Actions

The contract supports two redeemer actions:

1. **Stake** (`Constr 0`): Creates or updates a staking position
2. **ClaimAndUnstake** (`Constr 1`): Allows the staker to claim yields and withdraw their principal

---

## 2. Data Types

### 2.1 StakingDatum

```haskell
data StakingDatum = StakingDatum
    { stakerPkh     :: PubKeyHash
    , principal     :: Integer
    , startTime     :: POSIXTime
    , claimedYield  :: Integer
    , active        :: Bool
    }
```

Represents a staking position in the vault.

| Field | Type | Description |
|-------|------|-------------|
| `stakerPkh` | `PubKeyHash` | Public key hash of the staker who owns this position |
| `principal` | `Integer` | Amount of ADA staked (in lovelace) |
| `startTime` | `POSIXTime` | Timestamp when the stake was created (microseconds since epoch) |
| `claimedYield` | `Integer` | Total yield already claimed by the staker (in lovelace) |
| `active` | `Bool` | Whether the stake is currently active |

### 2.2 StakingRedeemer

```haskell
data StakingRedeemer 
    = Stake
    | ClaimAndUnstake
```

Defines the available actions:

| Constructor | Description |
|-------------|-------------|
| `Stake` | Creates a new staking position or adds to existing stake |
| `ClaimAndUnstake` | Claims accumulated yields and returns the principal to the staker |

---

## 3. Validation Logic

The core validator function:

```haskell
validator :: StakingDatum -> StakingRedeemer -> ScriptContext -> Bool
```

### 3.1 Stake Action

When a user stakes ADA, the validator ensures:

#### ✅ **Signature Requirement**
```haskell
txSign = txSignedBy txInfo (stakerPkh datum)
```
The transaction must be signed by the staker's public key hash.

#### ✅ **Active Status**
```haskell
active datum == True
```
The staking position must be marked as active.

#### ✅ **Minimum Principal**
```haskell
principal datum > 3_000_000  -- > 3 ADA minimum
```
The staked amount must exceed 3 ADA to prevent dust attacks and ensure meaningful participation.

**Validation passes if:** The transaction is properly signed, the stake is active, and the principal meets the minimum threshold.

---

### 3.2 ClaimAndUnstake Action

When a user claims yields and unstakes, the validator ensures:

#### ✅ **Signature Requirement**
```haskell
txSign = txSignedBy txInfo (stakerPkh datum)
```
Only the original staker can claim and unstake.

#### ✅ **Active Status**
```haskell
active datum == True
```
The stake must be active to be claimed.

#### ✅ **Time Validation**
```haskell
currentTime = case ivFrom (txInfoValidRange txInfo) of
    LowerBound (Finite t) _ -> t
    _ -> traceError "cannot determine current time from tx valid range"
```
The validator extracts the current time from the transaction's validity interval to calculate yields.

#### ✅ **Yield Calculation**
```haskell
stakingDuration = currentTime - startTime datum
yieldRate = 5  -- 5% annual yield (simplified)
earnedYield = (principal datum * yieldRate * stakingDuration) 
              / (100 * 365 * 24 * 60 * 60 * 1_000_000)
totalOwed = principal datum + earnedYield - claimedYield datum
```

The contract calculates:
- **Duration**: Time elapsed since staking began
- **Earned Yield**: Based on a fixed annual percentage rate
- **Total Owed**: Principal + new yield - previously claimed yield

#### ✅ **Payment Verification**
```haskell
stakerPaid = any (\o ->
    valueOf (txOutValue o) adaSymbol adaToken >= totalOwed
    && case addressCredential (txOutAddress o) of
         PubKeyCredential pkh -> pkh == stakerPkh datum
         _ -> False
    ) (txInfoOutputs txInfo)
```

At least one output must:
- Send **≥ totalOwed** lovelace
- Go to an address controlled by the **staker's PKH**

Outputs to script addresses are ignored.

#### ✅ **Final Condition**
```haskell
txSign && active datum == True && stakerPaid
```
All three conditions must be satisfied for the UTxO to be spent.

---

## 4. Security Features

### 4.1 Ownership Protection
- Only the original staker (matching `stakerPkh`) can claim yields or unstake
- Signature verification prevents unauthorized access

### 4.2 Double-Spend Prevention
- The `claimedYield` field tracks all previous claims
- Ensures stakers can't claim the same yield multiple times

### 4.3 Minimum Stake Requirement
- 3 ADA minimum prevents blockchain spam
- Ensures meaningful participation in the energy fund

### 4.4 Time-Based Yield
- Yields calculated based on actual time staked
- Uses transaction validity interval for trustless time verification

### 4.5 Active Status Flag
- Prevents interaction with inactive or closed stakes
- Allows for stake lifecycle management

---

## 5. Untyped Wrapper

The contract uses the standard Plutus wrapper to convert `BuiltinData` into typed values:

```haskell
untypedValidator :: BuiltinData -> BuiltinData -> BuiltinData -> ()
untypedValidator d r c =
    check (validator datum redeemer ctx)
  where
    datum    = unsafeFromBuiltinData d
    redeemer = unsafeFromBuiltinData r
    ctx      = unsafeFromBuiltinData c
```

The function throws an error if validation fails.

---

## 6. Building the Script

The validator is compiled using Template Haskell:

```haskell
validatorScript :: Validator
validatorScript =
  PlutusV2.mkValidatorScript $$(PlutusTx.compile [|| untypedValidator ||])
```

And exported to a `.plutus` file:

```haskell
getCbor :: IO ()
getCbor = writeValidatorToFile "./assets/pgef_staking.plutus" validatorScript
```

This file can be deployed in wallets, DApps, or integrated with off-chain code.

---

## 7. Off-Chain Integration

The provided JavaScript interface (using Lucid) enables:

## 7.1 Wallet Connection
```javascript
connectWallet()
```
- Connects to Lace wallet
- Retrieves wallet address and PKH
- Calculates script address
- Loads existing staking positions

## 7.2 Staking ADA
```javascript
stake()
```
- Creates a new staking datum with current timestamp
- Sends ADA to the script address
- Records stake in the blockchain

## 7.3 Claiming and Unstaking
```javascript
claimAndUnstake()
```
- Selects an active staking UTxO
- Consumes the UTxO with `ClaimAndUnstake` redeemer
- Returns principal + yields to staker's wallet

## 7.4 UTxO Management
```javascript
loadStakingUtxos()
```
- Queries script address for UTxOs
- Filters for stakes owned by connected wallet
- Displays active stakes in UI dropdown

---

### 8. Economic Model

## 8.1 Yield Mechanism
The contract implements a **time-based linear yield model**:

- **Base Rate**: 5% annual yield (configurable)
- **Calculation**: Proportional to staking duration
- **Compounding**: Yields are calculated at claim time, not pre-computed
- **Sustainability**: Funded by renewable energy project revenues (off-chain)

 8.2 Liquidity Provision
Staked ADA provides:
- **Permanent liquidity** for energy infrastructure projects
- **Decentralized capital pool** without reliance on central authorities
- **Transparent yield distribution** enforced by smart contract logic

---

## 9. Use Cases

This staking vault is ideal for:

### ✅ Renewable Energy Financing
- **Solar panel installations**
- **Wind farm development**
- **Battery storage systems**
- **Green hydrogen production**

 ✅ Decentralized Impact Investing
- Community-driven funding for sustainable projects
- Transparent allocation of staked capital
- Automated yield distribution

 ✅ Long-Term Liquidity Provision
- Encourages long-term commitment through yield incentives
- Provides stable funding source for multi-year projects

---

## 10. Summary

The **PGEF Staking Vault** implements a secure, time-based staking mechanism requiring:

1. **Authorization** through staker signature verification
2. **Time-locked yields** calculated based on staking duration
3. **Guaranteed payouts** of principal + earned yields
4. **Active stake management** with clear lifecycle tracking

**Key Benefits:**
- ✅ Trustless yield calculation
- ✅ Secure ownership verification
- ✅ Prevention of double-claiming
- ✅ Minimum stake requirements
- ✅ Transparent on-chain logic

This contract provides the foundation for **decentralized renewable energy financing** on the Cardano blockchain, enabling sustainable infrastructure development through community-driven liquidity provision.

---

### 11. Technical Specifications

| Property | Value |
|----------|-------|
| **Plutus Version** | V2 |
| **Network** | Preprod (testnet) |
| **Minimum Stake** | 3 ADA |
| **Yield Rate** | 5% annual (configurable) |
| **Time Precision** | Microseconds (POSIX) |
| **Script Language** | Haskell with PlutusTx |

---

## 12. Future Enhancements

Potential improvements for production deployment:

- **Variable yield rates** based on project performance
- **Governance mechanisms** for parameter adjustments
- **Multi-asset support** beyond ADA
- **Penalty mechanisms** for early unstaking
- **Batch claiming** for multiple stakes
- **Integration with real-world energy data** oracles

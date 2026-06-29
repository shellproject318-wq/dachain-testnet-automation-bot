# DAC Inception — Daily Multi-Wallet Bot

An automated bot to perform daily on-chain activities on the **DAC Inception Testnet** across multiple wallets.

---

## ✨ Features
faucet skiped (off)
| Feature | Description |
|---------|-------------|
| 💸 **Send TX** | Custom transaction count — amount auto-scales to balance |
| 📦 **Quantum Crate** | Open up to 5 Quantum Crates per day (150 QE each) — shows reset timer |
| 🔥 **Burn DACC** | Burn DACC to earn QE (configurable, max 0.1 DAC/cycle) |
| 🏅 **Badge Mint** | Auto-mint rank badges on-chain — **auto-skipped if all already minted** |
| ~~📋 **Activity Tasks**~~ | ~~Sync 14 on-chain tasks + visit 5 pages per cycle~~ — **disabled** |
| 🔄 **Multi-Wallet** | Run multiple wallets sequentially from `pk.txt` |
| 🌐 **Proxy Support** | Per-wallet proxy support for API + RPC (optional) |
| ♻️ **Auto Loop** | Automatically re-runs every **11–12 hours** (randomized) |
| 🛡️ **Error Handling** | Skips wallet on persistent server errors — no crash |
| ⚡ **429 Handling** | Exponential backoff on rate-limit — skips after 2 consecutive 429s |
| 🔐 **SIWE Auth** | Sign-In With Ethereum (nonce + signature) with address-only fallback |
| ⏱️ **RPC Timeout Skip** | Automatically skips wallet after **5 consecutive RPC timeout errors** — resets on success |

---

## 📋 Prerequisites

- **Node.js** v18 or higher
- **npm** or **yarn**

---

## 🚀 How to Use

### 1. Clone the Repository

```bash
git clone https://github.com/shellproject318-wq/dachain-testnet-automation-bot.git
cd dachain-testnet-automation-bot
```

### 2. Install Dependencies

```bash
npm install
```

### 3. Set Up Configuration Files

#### `pk.txt` — Wallet Private Keys *(required)*

```
0xabc123...
0xdef456...
0x789xyz...
```

> One private key per line. All keys must start with `0x`.

#### `address.txt` — Transaction Target Addresses *(optional)*

```
0xABCD...
0x1234...
```

> Used as TX destinations. If empty or missing, the bot generates random addresses.

#### `proxy.txt` — Proxy List *(optional)*

```
http://user:pass@ip:port
ip:port
user:pass@ip:port
```

> Proxies rotate per wallet. If missing, all wallets run direct.

### 4. Run the Bot

```bash
node auto.js
```

The bot starts immediately, prints a config summary, and loops every **11–12 hours** automatically.

> ✅ Fully non-interactive — no prompts. Safe to run via **PM2**, **nohup**, **screen**, or **cron**.

```bash
# PM2 (recommended)
pm2 start auto.js --name dachain-bot
pm2 save && pm2 logs dachain-bot

# nohup
nohup node auto.js &

# Cron (runs at 00:00 and 12:00 UTC)
0 0,12 * * * cd /root/dachain-bot && node auto.js >> bot.log 2>&1
```

---

## ⚙️ Default Config

| Parameter | Default | Description |
|-----------|---------|-------------|
| `txCount` | `3` | TX count per wallet per cycle *(reduced from 5)* |
| `txMaxAmt` | `0.05 DAC` | Max DAC per TX (auto-scaled to balance) |
| `burnAmount` | `0.005 DAC` | DACC burned per cycle |
| `loopMinHr` | `11` | Minimum hours between cycles |
| `loopMaxHr` | `12` | Maximum hours between cycles |
| `qcrateMax` | `5` | Max quantum crates per cycle |
| `mintBadge` | `true` | Enable rank badge minting (auto-skipped if all minted) |
| `MAX_TIMEOUT_ERRORS` | `5` | Max consecutive RPC timeout errors before wallet is skipped |

> To change defaults, edit the `CFG` object (and `MAX_TIMEOUT_ERRORS` constant) at the top of `auto.js`.

---

## ⏱️ Timing & Delays

| Location | Delay |
|----------|-------|
| 429 rate-limit (1st hit) | `Retry-After` header or **1.5 s** |
| 429 rate-limit (2nd hit) | Skip immediately |
| 500/502/503/504 retry | **2 s × attempt** (up to 5 retries) |
| Between TXs | Random **1–2 seconds** *(reduced from 2–5 s)* |
| Between Quantum Crate opens | Random **1.5–3 seconds** |
| Between badge mints | **1.5 seconds** |
| Between wallets | **3–6 seconds** |
| Between cycles (main loop) | Random **11–12 hours** |
| RPC wait (per check) | **5–8 seconds**, max **3 attempts** then skip |

---

## 🔄 Execution Flow (per wallet)

```
Timeout Skip Check  →  skip immediately if wallet reached 5x consecutive timeout
    │
Auth (SIWE nonce+sign → fallback address-only)
    │
    ├─ 1. Quantum Crate (≤5x)   → shows reset timer when limit reached
    ├─ 2. Send TX × 3           → amount auto-scaled to balance
    ├─ 3. Burn DACC → QE
    ├─ 4. Fetch Profile         → QE balance + badges[]
    │      │
    │      └─ 5. Mint Rank Badges
    │              ├─ AUTO-SKIP if all rank badges already minted on-chain
    │              ├─ POST /nft/claim-signature/  {rank_key}
    │              ├─ contract.claimRank(uint8, bytes)
    │              │       └─ on error: hasMinted() check
    │              └─ POST /nft/confirm-mint/     {rank_key, tx_hash}
    │
    ├─ 6. Activity Tasks        → DISABLED
    │
    └─ 7. Print Summary
         └─ On success: reset timeout counter for this wallet
```

---

## 🏅 Badge System

Rank badges are the only badges minted on-chain. Detection:

- Source: `profile.badges[]` (from `GET /api/inception/profile/`)
- Mintable: `badge__key.startsWith('rank_')` **AND** `nft_tx_hash === ""`
- **Auto-skip**: if `mintable.length === 0`, the entire mint step is skipped — no RPC calls made

| Rank | QE Reward | Requirement |
|------|-----------|-------------|
| Cadet | 10 | Sign up |
| Commando | 50 | 1,000 QE |
| Seal | 150 | 2,000 QE |
| Shadow Unit | 500 | 5,000 QE |
| Vanguard | 750 | 10,000 QE |
| Sentinel | 1,000 | 25,000 QE |
| Sovereign | 2,000 | 50,000 QE |
| Warrior | 2,500 | 100,000 QE |
| Architect | 3,000 | 200,000 QE |
| Interceptor | 3,500 | 300,000 QE |
| Phantom | 4,000 | 400,000 QE |
| Cipher | 4,500 | 500,000 QE |
| Crown | 5,000 | 750,000 QE |

**Contract:** `0xB36ab4c2Bd6aCfC36e9D6c53F39F4301901Bd647`
**ABI:** `claimRank(uint8 rankId, bytes signature)` · `hasMinted(address, uint8) → bool`

---

## 🛡️ Error Handling

| Error | Behaviour |
|-------|-----------|
| HTTP `429` (1st) | Wait Retry-After or 1.5 s, retry |
| HTTP `429` (2nd consecutive) | Skip endpoint — rate-limited |
| HTTP `500/502/503/504` | Linear backoff, up to 5 retries |
| `UNKNOWN_ERROR` / "coalesce" | No retry — calls `hasMinted()` to check on-chain state |
| `INSUFFICIENT_FUNDS` | Logged and skipped |
| RPC unreachable | Retries 3× (~15 s total), then skips TX for that wallet |
| **RPC timeout (≥ 5 consecutive)** | **Wallet skipped for the entire cycle — counter resets on next successful run** |
| Persistent server error | Wallet skipped, bot continues to next |
| `unhandledRejection` / `uncaughtException` | Suppressed if RPC/network error — no crash |

---

## ⏱️ RPC Timeout Skip — How It Works

This feature prevents the bot from getting stuck or wasting time on wallets with consistently unreachable RPC endpoints.

### Logic

```
Per cycle, before processing each wallet:
  if timeout_error_count[wallet] >= 5  →  skip wallet, log warning, continue

During wallet execution:
  if RPC timeout error occurs  →  increment timeout_error_count[wallet]
  if wallet completes successfully  →  reset timeout_error_count[wallet] to 0
```

### Counter Behaviour

| Situation | Counter |
|-----------|---------|
| RPC timeout error detected | `+1` per occurrence |
| Wallet finishes successfully | Reset to `0` |
| Counter reaches `5` | Wallet **skipped** in current and future cycles until a successful run resets it |

### Console Output

```
[10:30:01] ⚠ Wallet 3 (0x000..aB12...) skipped — RPC timeout error reached 5x
[10:30:05] ⚠ Wallet 7 RPC timeout #3/5 — retrying next cycle
[10:30:09] ⚠ Wallet 9 RPC timeout #5/5 — will be SKIPPED next cycle
```

### Configuration

To change the threshold, edit `MAX_TIMEOUT_ERRORS` near the top of `auto.js`:

```js
const MAX_TIMEOUT_ERRORS = 5; // increase to be more lenient, decrease to skip faster
```

---

## 📊 Sample Output

```
===============================================================
  DAC Inception Bot — v2.3 (optimized)
  TX: 3 | Max amt: 0.05 DAC | Burn: 0.005 DAC | Badge: ON
===============================================================

[10:30:00] 🚀 Starting cycle #1
-------------------------------------------------------
[10:30:01] ▶ [0x000..eF39] Wallet 1/50 | direct
[10:30:02] ✓ [0x000..eF39] Auth OK
[10:30:04] ℹ [0x000..eF39] Opening up to 5 Quantum Crate(s)...
[10:30:05] ✓ [0x000..eF39] Quantum Crate 4/5 ✓ — reward: 500 QE | QE total: 12662
[10:30:06] ⏭ [0x000..eF39] Quantum Crate: daily limit reached — resets in 20h 14m
[10:30:07] ℹ [0x000..eF39] Balance: 30.3587 DAC
[10:30:07] → [0x000..eF39] Sending 3 TX...
[10:30:13] ✓ [0x000..eF39] TX 3/3 ✓ → 0x9be7a4b8... | hash: 0x927c9272cd87...
[10:30:18] ✓ [0x000..eF39] Burn success — 0xf5d59c59...
[10:30:19] ✓ [0x000..eF39] QE Balance: 12712
[10:30:19] 🏅 [0x000..eF39] All rank badges already minted (9 on-chain) — skipping badge mint
[10:30:19] ⏭ [0x000..eF39] Activity Tasks — disabled
-------------------------------------------------------
[10:30:19] 📊 SUMMARY [0xD767..eF38]
   ✓ TX Sent       : 3/3
   ✓ Quantum Crate : 5/5 opened (+750 QE)
   ✓ Burn          : success
   🏅 Badges        : 9 already minted (auto-skipped)
   ℹ QE Balance    : 15712
   ⏭ Tasks         : disabled
-------------------------------------------------------
[10:xx:xx] ✓ Cycle done — 50 OK, 0 skipped
[10:xx:xx] Next cycle in 11.43 hours...
```

---

## 🔧 Optimization Notes

The following changes have been applied to reduce per-wallet execution time and improve reliability:

| Change | Before | After | Benefit |
|--------|--------|-------|---------|
| TX count | `5` | `3` | ~33 s saved |
| Sleep between TXs | `2–5 s` | `1–2 s` | ~6 s saved |
| Activity Tasks | Enabled | **Disabled** | ~14 s saved |
| Badge Mint | Always runs | **Auto-skipped if all minted** | ~60–100 s saved |
| RPC max attempts | `6` (~45 s) | `3` (~15 s) | ~30 s saved |
| **RPC timeout skip** | No limit | **Skip after 5 consecutive timeouts** | Prevents infinite stalls |
| **Total saved** | | | **~143–183 s/wallet** |

---

## 📁 Project Structure

```
dachain-bot/
├── auto.js           ← main script (non-interactive, v2.4 optimized)
├── pk.txt           ← wallet private keys (required)
├── address.txt      ← TX target addresses (optional)
├── proxy.txt        ← proxy list (optional)
├── state.json       ← saved state (auto-generated)
└── README.md
```

---

## ⚠️ Disclaimer

> This bot is intended for use on the **DAC Inception Testnet** only.  
> Do **not** use private keys from mainnet wallets containing real assets.  
> Use at your own risk.

# DAC Inception — Daily Multi-Wallet Bot

An automated bot to perform daily on-chain activities on the **DAC Inception Testnet** across multiple wallets.

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 💸 **Send TX** | Configurable TX count per wallet — amount auto-scales to balance |
| 🚰 **Faucet** | Auto-claim daily faucet — shows cooldown timer if already claimed |
| 📦 **Quantum Crate** | Open up to 5 Quantum Crates per day (150 QE each) — shows reset timer |
| 🔥 **Burn DACC** | Burn DACC to earn QE (configurable, max 0.1 DAC/cycle) |
| 🏅 **Badge Mint** | Auto-mint rank badges on-chain via `claimRank(uint8, bytes)` + signature flow |
| 📋 **Activity Tasks** | Sync 14 on-chain tasks + visit 5 pages per cycle |
| 🔄 **Multi-Wallet** | Run multiple wallets sequentially from `pk.txt` |
| 🌐 **Proxy Support** | Per-wallet proxy support for API + RPC (optional) |
| ♻️ **Auto Loop** | Automatically re-runs every **11–12 hours** (randomized) |
| 🛡️ **Error Handling** | Skips wallet on persistent server errors — no crash |
| ⚡ **429 Handling** | Exponential backoff on rate-limit — skips after 2 consecutive 429s |
| 🔐 **SIWE Auth** | Sign-In With Ethereum (nonce + signature) with address-only fallback |

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
node bot.js
```

The bot starts immediately, prints a config summary, and loops every **11–12 hours** automatically.

> ✅ Fully non-interactive — no prompts. Safe to run via **PM2**, **nohup**, **screen**, or **cron**.

```bash
# PM2 (recommended)
pm2 start bot.js --name dachain-bot
pm2 save && pm2 logs dachain-bot

# nohup
nohup node bot.js &

# Cron (runs at 00:00 and 12:00 UTC)
0 0,12 * * * cd /root/dachain-bot && node bot.js >> bot.log 2>&1
```

---

## ⚙️ Default Config

| Parameter | Default | Description |
|-----------|---------|-------------|
| `txCount` | `5` | TX count per wallet per cycle |
| `txMaxAmt` | `0.05 DAC` | Max DAC per TX (auto-scaled to balance) |
| `burnAmount` | `0.005 DAC` | DACC burned per cycle |
| `loopMinHr` | `11` | Minimum hours between cycles |
| `loopMaxHr` | `12` | Maximum hours between cycles |
| `qcrateMax` | `5` | Max quantum crates per cycle |
| `mintBadge` | `true` | Enable rank badge minting |

> To change defaults, edit the `CFG` object at the top of `bot.js`.

---

## ⏱️ Timing & Delays

| Location | Delay |
|----------|-------|
| 429 rate-limit (1st hit) | `Retry-After` header or **1.5 s** |
| 429 rate-limit (2nd hit) | Skip immediately |
| 500/502/503/504 retry | **2 s × attempt** (up to 5 retries) |
| Between TXs | Random **2–5 seconds** |
| After Faucet | **2 seconds** |
| Between Quantum Crate opens | Random **1.5–3 seconds** |
| Between badge mints | **1.5 seconds** |
| Between wallets | **3–6 seconds** |
| Between cycles (main loop) | Random **11–12 hours** |
| RPC wait (per check) | **5–8 seconds**, max **6 attempts** |

---

## 🔁 Execution Flow (per wallet)

```
Auth (SIWE nonce+sign → fallback address-only)
    │
    ├─ 1. Faucet Claim          → shows "next in Xh Ym" if already claimed
    ├─ 2. Quantum Crate (≤5x)   → shows reset timer when limit reached
    ├─ 3. Send TX × N           → amount auto-scaled to balance
    ├─ 4. Burn DACC → QE
    ├─ 5. Fetch Profile         → QE balance + faucet timer + badges[]
    │      │
    │      └─ 6. Mint Rank Badges (unminted only)
    │              ├─ POST /nft/claim-signature/  {rank_key}
    │              ├─ contract.claimRank(uint8, bytes)
    │              │       └─ on error: hasMinted() check
    │              └─ POST /nft/confirm-mint/     {rank_key, tx_hash}
    │
    ├─ 7. Activity Tasks
    │      ├─ POST /task/  × 14  (sync on-chain milestones)
    │      └─ POST /visit/ × 5   (page visit rewards)
    │
    └─ 8. Print Summary
```

---

## 🏅 Badge System

Rank badges are the only badges minted on-chain. Detection:

- Source: `profile.badges[]` (from `GET /api/inception/profile/`)
- Mintable: `badge__key.startsWith('rank_')` **AND** `nft_tx_hash === ""`

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
| RPC unreachable | Retries 6× (~45 s total), then skips TX for that wallet |
| Persistent server error | Wallet skipped, bot continues to next |
| `unhandledRejection` / `uncaughtException` | Suppressed if RPC/network error — no crash |

---

## 📊 Sample Output

```
===============================================================
  DAC Inception Bot — v2.3
  TX: 5 | Max amt: 0.05 DAC | Burn: 0.005 DAC | Badge: ON
===============================================================

[10:30:00] 🚀 Starting cycle #1
-------------------------------------------------------
[10:30:01] ▶ [0x000..eF39] Wallet 1/50 | direct
[10:30:02] ✓ [0x000..eF39] Auth OK
[10:30:02] ⏭ [0x000..eF39] Faucet: already claimed — next in 18h 32m
[10:30:04] ℹ [0x000..eF39] Opening up to 5 Quantum Crate(s)...
[10:30:05] ✓ [0x000..eF39] Quantum Crate 4/5 ✓ — reward: 500 QE | QE total: 12662
[10:30:06] ⏭ [0x000..eF39] Quantum Crate: daily limit reached — resets in 20h 14m
[10:30:07] ℹ [0x000..eF39] Balance: 30.3587 DAC
[10:30:07] → [0x000..eF39] Sending 5 TX...
[10:30:15] ✓ [0x000..eF39] TX 5/5 ✓ → 0x9be7a4b8... | hash: 0x927c9272cd87...
[10:30:20] ✓ [0x000..eF39] Burn success — 0xf5d59c59...
[10:30:21] ✓ [0x000..eF39] QE Balance: 12712
[10:30:21] 🏅 [0x000..eF39] Badges: 54 earned | 1 rank unminted | 8 rank minted
[10:30:21] 🏅 [0x000..eF39] Minting [Architect] +3000 QE (rank_architect)
[10:30:22] ℹ [0x000..eF39]   Got signature for [Architect] rank_id=8
[10:30:22] ℹ [0x000..eF39]   Estimating gas for claimRank(8)...
[10:30:23] ℹ [0x000..eF389   On-chain claimRank(8, sig) [Architect]...
[10:30:28] ✓ [0x000..eF38]   [Architect] minted — 0xabc123...
[10:30:30] ✓ [0x000..eF38] Activities: 8 synced | 5 visited
-------------------------------------------------------
[10:30:30] 📊 SUMMARY [0xD767..eF38]
   ✓ TX Sent       : 5/5
   ⏭ Faucet        : already claimed — next in 18h 32m
   ✓ Quantum Crate : 5/5 opened (+750 QE)
   ✓ Burn          : success
   🏅 Badges        : 1/1 minted | 8 already done
   ℹ QE Balance    : 15712
   ℹ Tasks         : 8 tasks synced, 5 pages visited
-------------------------------------------------------
[10:xx:xx] ✓ Cycle done — 50 OK, 0 skipped
[10:xx:xx] Next cycle in 11.43 hours...
```

---

## 📁 Project Structure

```
dachain-bot/
├── bot.js           ← main script (non-interactive, v2.3)
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

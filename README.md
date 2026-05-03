##updated
# DAC Inception — Daily Multi-Wallet Bot

An automated bot to perform daily on-chain activities on the **DAC Inception Testnet** across multiple wallets simultaneously.

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 💸 **Send TX** | Send 5 transactions per wallet to random addresses |
| 🚰 **Faucet** | Auto-claim daily faucet |
| 📦 **Quantum Crate** | Open up to 5 Quantum Crates per day (150 QE each) |
| 🔥 **Burn DACC** | Burn 0.005 DAC to earn QE |
| 🏅 **Badge Mint** | Mint / claim available badges on-chain |
| 🔄 **Multi-Wallet** | Run multiple wallets from `pk.txt` |
| 🌐 **Proxy Support** | Per-wallet proxy support (optional) |
| ♻️ **Auto Loop** | Automatically re-runs every **4–8 hours** (randomized) |
| 🛡️ **Error Handling** | Skips wallet on persistent server errors — no crash |

---

## 📋 Prerequisites

- **Node.js** v18 or higher
- **npm** or **yarn**

---

## 🚀 How to Use

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/dac-inception-bot.git
cd dac-inception-bot
```

### 2. Install Dependencies

```bash
npm install
```

Or with yarn:

```bash
yarn install
```

### 3. Set Up Configuration Files

Create the following files in the same directory as `bot.js`:

#### `pk.txt` — Wallet Private Keys *(required)*
```bash
0xabc123...
0xdef456...
0x789xyz...
```
> One private key per line. All keys must start with `0x`.

#### `address.txt` — Transaction Target Addresses *(optional)*
```bash
0xABCD...
0x1234...
```
> Used as TX destinations. If empty or missing, the bot generates random addresses.

#### `proxy.txt` — Proxy List *(optional)*
```bash
http://user:pass@ip:port
ip:port
user:pass@ip:port
```
> Proxies are assigned round-robin per wallet. If missing, the bot runs without proxy.

### 4. Run the Bot

```bash
node bot.js
```

The bot will start immediately and loop automatically every **4–8 hours**.

---

## ⏱️ Timing & Delays

| Location | Delay |
|----------|-------|
| Retry per attempt (`withRetry`) | Random **2–10 seconds** |
| Between TXs (same wallet) | Random **2–5 seconds** |
| After Faucet | **2 seconds** |
| After Quantum Crate | **2 seconds** |
| Between Quantum Crate opens | Random **1.5–3 seconds** |
| Between wallets | Random **3–6 seconds** |
| Between cycles (main loop) | Random **4–8 hours** |

---

## 🔁 Execution Flow (per wallet)

```text
Auth Login
    │
    ├─ 1. Faucet Claim
    ├─ 2. Quantum Crate (max 5x/day)
    ├─ 3. Send 5 TX
    ├─ 4. Burn DACC → QE
    ├─ 5. Mint Badges (on-chain)
    └─ 6. Fetch QE Balance & Print Summary
```

---

## 🛡️ Error Handling

- **Server errors (500/502/503/504/timeout)** → Auto-retry up to **5x** with a random **2–10 second** delay per attempt
- **Persistent server error** → Wallet is skipped, bot continues to the next wallet
- **`unhandledRejection` / `uncaughtException`** → Internal RPC errors are suppressed to prevent crashes

---

## 📊 Sample Output

```text
[12:00:01] 🔷 Starting cycle #1
───────────────────────────────────────────────────────
  DAC Inception Bot — 3 wallet(s) loaded
───────────────────────────────────────────────────────

[12:00:02] 🔷 [0xAbCd..1234] Wallet 1/3 | direct
[12:00:03] ✅ [0xAbCd..1234] Auth OK
[12:00:04] ✅ [0xAbCd..1234] Faucet: success
[12:00:06] 📦 [0xAbCd..1234] Opening up to 5 Quantum Crate(s)...
[12:00:08] ✅ [0xAbCd..1234] Quantum Crate 1/5 ✓ — reward: 150 QE | QE total: 150
...
[12:00:30] ───────────────────────────────────────────────
[12:00:30] 🔷 SUMMARY [0xAbCd..1234]
   ✅ TX Sent      : 5/5
   ✅ Faucet       : success
   ✅ Quantum Crate: 5/5 opened (+750 QE)
   ✅ Burn         : success
   🔷 QE Balance  : 900
   🔷 Badges      : 1/1 minted
[12:00:30] ───────────────────────────────────────────────

[12:xx:xx] ✅ Cycle done — 3 OK, 0 skipped
[12:xx:xx] Next cycle in 6.24 hours...
```

---

## 📁 Project Structure

```text
dac-inception-bot/
├── bot.js           ← main script
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

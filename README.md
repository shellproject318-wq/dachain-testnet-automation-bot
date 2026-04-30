# DAC Inception — Daily Testnet Bot
Automated daily activities for [DAC Inception](https://inception.dachain.io/activity) testnet.

**Chain:** DAC Quantum Chain (ID: 21894)
**RPC:** `https://rpctest.dachain.tech`
**Explorer:** `https://exptest.dachain.tech`

---

## Activities

| # | Action | Description |
|---|--------|-------------|
| 1 | 🚰 Faucet | Claim free DACC — **1x per day** (requires X or Discord linked) |
| 2 | 📦 Crate | Open daily Quantum Crate — **5x per day** → earn QE |
| 3 | 💸 TX | Send transactions — **15x per day** → earn TX badges |
| 4 | 🔥 Burn | Burn DACC → Quantum Energy (QE) |
| 5 | 🔄 Sync | Sync all activity to API |

## Badges (Auto-earned)

| Badge | Requirement | QE Reward |
|-------|-------------|-----------|
| Sign In | First login | 25 |
| First Crate | Open 1 crate | 25 |
| First Transaction | Send 1 tx | 50 |
| Getting Started | Send 3 tx | 25 |
| 10 Transactions | Send 10 tx | 100 |
| 50 Transactions | Send 50 tx | 250 |
| First Drip | Claim faucet 1x | 25 |
| Regular | Claim faucet 10x | 50 |
| Daily Streak | 3/7/14/21/30 days | 50–1000 |
| QE milestones | 500+ total QE | 50–5000 |

---

## Setup

### 1. Clone Repository

```bash
git clone https://github.com/shellproject318-wq/dachain-testnet-automation-bot.git
cd dachain-testnet-bot
npm install
```

### 2. Wallet Keys

Create `pk.txt` in the project directory — one private key per line:

```bash
# Single wallet
echo "0xYOUR_PRIVATE_KEY_HERE" > pk.txt

# Multi-wallet
echo "0xWALLET_1_KEY" > pk.txt
echo "0xWALLET_2_KEY" >> pk.txt
echo "0xWALLET_3_KEY" >> pk.txt
```

> ⚠️ Never share or commit `pk.txt`!

### 3. Prerequisites per Wallet

Each wallet must have:
- ✅ Connected at [inception.dachain.io](https://inception.dachain.io) at least once
- ✅ Linked Twitter (X) or Discord for faucet
- ✅ Some DAC balance for txs and burn (claim faucet first)

---

## Usage

### Single Run

```bash
node bot.js --once
```

### Loop Mode (every 6 hours)

```bash
node bot.js
```

The bot checks every **6 hours** (4 times per day: ~00:00, 06:00, 12:00, 18:00 UTC).
Daily limits are tracked per wallet in `state.json` and reset automatically after 24 hours:

| Task | Daily Limit |
|------|-------------|
| 🚰 Faucet | 1x / day |
| 📦 Crate | 5x / day |
| 💸 TX | 15x / day |

If a task has already reached its daily limit, the bot skips it automatically and waits for the next cycle.

### Cron Mode (4x daily: 00:00, 06:00, 12:00, 18:00 UTC)

```bash
node bot.js --cron
```

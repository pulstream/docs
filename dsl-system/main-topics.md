# Main topics

## Carbon Router Topics

This document defines topics produced by the Carbon Router.

There are two classes of topics:

- Raw topics: high-frequency deltas used by the UI for realtime rendering.
- Wasm topics: low-TPS, semantically meaningful events for WASM functions to follow.

### Throughput and batching

- When WASM topics can get high TPS, batch by 2 consecutive slots.
- Do not have WASM functions follow high-TPS raw topics; it can overload memory.
- Prefer whitelisting only WASM topics when creating a function.

## WASM Topics

This section summarizes what each topic does and when it emits. Values and schemas are intentionally omitted for simplicity.

### Token-driven pulses

- 📤 `wasm.token.holder-pulse`

  - Purpose: detect meaningful changes in token holders over a short window.
  - Emits when: holder count changes by at least ±10 and ≥ 5% across a 2-slot window.
  - Filters: tokens created after system start; liquidity < 1M.
  - Batch: 2 slots.

```rust
struct Message {
  _meta: {
    ts: u64,               // timestamp
    type: "holder_pulse", // common format
  },
  mint: String,
  slots: (u64, u64),
  after: usize,
  before: usize,
}
```

- 📤 `wasm.token.liquidity-pulse`

  - Purpose: detect meaningful liquidity changes.
  - Emits when: liquidity changes by ±5% and by ≥ 5 SOL across a 2-slot window.
  - Filters: tokens created after system start.
  - Batch: 2 slots.

```rust
struct Message {
  _meta: {
    ts: u64,                   // timestamp
    type: "liquidity_pulse",  // common format
  },
  mint: String,
  slots: (u64, u64),
  after: usize,
  before: usize,
}
```

- 📤 `wasm.token.trade-in-slot`
  - Purpose: per-slot trade summary for a token.
  - Emits when: total traded volume in the slot ≥ 5 SOL.
  - Filters: tokens created after system start; only while not bonded and up to 5 minutes after bonding.
  - Frequency: per slot.

```rust
struct Message {
  _meta: {
    ts: u64,
    type: "trade_in_slot", // common format
  },
  mint: String,
  count: u64,
  total_volume: f64,       // total SOL volume in slot
  last_updated_slot: u64,
  signature_logs: Vec<SignatureLog>,
  bonded: bool,            // token migrated / not migrated
}

pub struct SignatureLog {
  pub signature: String,         // tx signature
  pub signers: HashSet<String>,  // list of signers
  pub instructions: Vec<String>, // actions in transaction (buy, sell, create)
  pub buy_amt_sol: f64,          // total buy amount of tx
  pub sell_amt_sol: f64,         // total sell amount of tx
}
```

### User-driven actions

- 📤 `wasm.trader.trade-half-sol`

  - Purpose: capture individual trades with size ≥ 0.5 SOL.
  - Emits when: a qualifying trade occurs.
  - Filters: tokens created after system start; only while not migrated and up to 5 minutes after migration.

```rust
struct Message {
  _meta: {
    ts: u64,              // timestamp
    type: "trade",        // common format
  },
  signature: String,
  mint: String,
  payer: String,
  timestamp: i64,
  slot: u64,
  amount_in: f64,
  amount_out: f64,
  is_buy: bool,
}
```

- 📤 `wasm.trader.just-funded`
  - Purpose: flag addresses that traded ≥ 0.5 SOL and received SOL deposits in the last 24 hours.
  - Emits when: an address satisfies both conditions above.
  - Model:

```rust
struct Message {
  _meta: {
    ts: i64,
    type: "just_funded",  // common format
  },
  recipient: String,
  amt_sol: f64,                  // total sol
  details: HashMap<String, f64>, // payer -> amount sol
  payers: Vec<String>,           // list of payers
  exchange: Option<String>,      // e.g. OKX, Upbit, Coinbase, Bithumb, Kucoin, Bitfinex, Kraken, Crypto.com
  label: Option<String>,        // e.g. OKX: Hot Wallet, Binance 1
}
```

### Launchpad events

- 📤 `wasm.launchpad.pumpfun.created` and 📤 `wasm.launchpad.bonk.created`

  - Purpose: token creation events from respective launchpads.
  - Emits when: a new token is created on the launchpad.
  - Filters: tokens created after system start.

```rust
struct Message {
  _meta: {
    ts: u64,              // timestamp
    type: "create",       // common format
  },
  mint: String,
  name: String,
  symbol: String,
  uri: String,
  creator: String,
  creator_balance: f64, // creator sol balance
  tokens_migrated_count: usize, // Count of migrated tokens assigned to the creator
  creation_time: i64,
  total_supply: f64,
  decimals: u8,
}
```

- 📤 `wasm.launchpad.pumpfun.completed` and 📤 `wasm.launchpad.bonk.completed`
  - Purpose: token completion/migration events with finalization context.
  - Emits when: token completes/migrates on the launchpad.
  - Filters: tokens created after system start.

```rust
struct Message {
  _meta: {
    ts: u64,              // timestamp
    type: "completed",    // common format
  },
  mint: String,
  name: String,
  symbol: String,
  uri: String,
  creator: String,
  creation_time: i64,
  creation_slot: u64,
  total_supply: f64,
  decimals: u8,
  price: f64,
  market_cap: f64,
  liquidity: f64,
  token_liquidity: f64,
  migrated_at: Option<i64>,
  holders_count: usize,
}
```

## Raw Topics

- `raw.token-state-changed`

  - Purpose: realtime deltas for UI rendering.
  - Emits when: any token field changes; only changed fields are sent.

```rust
struct Message {
  _meta: {
    ts: u64,                 // timestamp
    type: "token_state_changed", // common format
  },
  mint: String,
  last_updated_slot: u64,
  total_supply: f64,
  price: f64,
  liquidity: f64,
  token_liquidity: f64,
  market_cap: f64,
  bonding_progress: f64,
  migrated: bool,
  migrated_at: Option<i64>,
  holders_count: usize,
  trades: u64,                // total trades (buys/sells)
  half_sol_trades: u64,       // total trades > 0.5 SOL
  top10_holder_percent: f64,  // top 10 holders percentage
  top100_holder_percent: f64, // top 100 holders percentage
  creator_tokens_created: usize,
  creator_tokens_migrated: usize,
}
```

- `raw.token_created`
  Merge `wasm.launchpad.pumpfun.created` and `wasm.launchpad.bonk.created` for a unified creation feed.

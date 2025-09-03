# RocksDB State

This document describes how data is organized and stored in RocksDB for fast retrieval by services and the UI. Each section lists the column family, key format, Rust models, and plain-language explanations of the fields.

Notes

- Amount, price, liquidity, and volume fields are numeric. Consumers should treat units consistently with the ingestion pipeline (commonly SOL-denominated values; use your pipeline’s unit conversions as needed).
- Unless stated, IDs are deterministic keys used to fetch a single record.

## 1. Token

### 1.1 Token info

- Column family: token
- Query key: `t:{mint}`

```rust
pub struct Token {
    pub id: String,                   // internal identifier for this token (typically equal to mint)
    pub program: String,              // origin program/launchpad or DEX program
    pub signature: String,            // creation transaction signature
    pub mint: String,                 // token mint address
    pub bonding_curve: Option<String>,// optional bonding curve id/address (pre-migration)
    pub name: String,                 // token display name
    pub symbol: String,               // token ticker symbol
    pub uri: String,                  // metadata URI
    pub creator: Pubkey,              // on-chain creator address
    pub creation_time: i64,           // unix timestamp when created
    pub creation_slot: u64,           // slot when created
    pub total_supply: u64,            // total minted supply (raw units, consider decimals)
    pub decimals: u8,                 // number of decimal places
    pub price: u64,                   // current price per token (unit per ingestion pipeline)
    pub liquidity: u64,               // current liquidity
    pub market_cap: u64,              // price × supply (pipeline-computed)
    pub bonding_progress: f64,        // progress along bonding curve (0–1)
    pub curve_type: u8,               // encoded bonding curve type id

    // Migration
    pub migrated: bool,               // whether token migrated to DEX pool
    pub migrated_at: Option<i64>,     // unix timestamp when migration happened

    // Pool info
    pub pool: Option<PoolInfo>,       // pool details after migration

    #[serde(default)]
    pub holders_count: usize,         // cached number of unique holders

    #[serde(default)]
    pub trades: u64,                  // total number of trades observed
    #[serde(default)]
    pub half_sol_trades: u64,         // total trades with size ≥ 0.5 SOL
}

pub struct PoolInfo {
    pub pool_program: String,     // program id of the pool
    pub pool_address: String,     // pool account address
    pub pool_authority: String,   // authority account for the pool
    pub pool_mint_account: String,// token mint account for pool token
    pub pool_wsol_account: String,// WSOL account for the pool
}
```

### 1.2 Holders

- Column family: token
- Query key: `h:{mint}`

```rust
pub struct TokenHolders {
    pub id: String,                         // record id (typically mint)
    pub last_updated_slot: u64,             // slot when this snapshot was updated
    pub mint: String,                       // token mint address
    pub top_holders: HashMap<String, u64>,  // top 100 holders: address -> balance
    pub holders_count: usize,               // total unique holders
}
```

### 1.3 Stats

- Column family: token
- Query key: `s:{mint}`

```rust
pub struct TokenTradeStats {
    pub id: String,              // record id (typically mint)

    pub mint: String,            // token mint address

    pub last_updated_timestamp: i64, // unix time of last update
    pub last_updated_slot: u64,      // slot of last update

    pub price_1m: f64,           // price over last 1 minute
    pub price_5m: f64,           // price over last 5 minutes
    pub price_1h: f64,           // price over last 1 hour
    pub price_6h: f64,           // price over last 6 hours
    pub price_24h: f64,          // price over last 24 hours

    pub buys_1m: u64,            // number of buys in last 1 minute
    pub buys_5m: u64,            // number of buys in last 5 minutes
    pub buys_1h: u64,            // number of buys in last 1 hour
    pub buys_6h: u64,            // number of buys in last 6 hours
    pub buys_24h: u64,           // number of buys in last 24 hours

    pub sells_1m: u64,           // number of sells in last 1 minute
    pub sells_5m: u64,           // number of sells in last 5 minutes
    pub sells_1h: u64,           // number of sells in last 1 hour
    pub sells_6h: u64,           // number of sells in last 6 hours
    pub sells_24h: u64,          // number of sells in last 24 hours

    pub volume_1m: u64,          // total volume (buy+sell) in last 1 minute
    pub volume_5m: u64,          // total volume (buy+sell) in last 5 minutes
    pub volume_1h: u64,          // total volume (buy+sell) in last 1 hour
    pub volume_6h: u64,          // total volume (buy+sell) in last 6 hours
    pub volume_24h: u64,         // total volume (buy+sell) in last 24 hours

    pub buy_volume_1m: u64,      // buy volume in last 1 minute
    pub buy_volume_5m: u64,      // buy volume in last 5 minutes
    pub buy_volume_1h: u64,      // buy volume in last 1 hour
    pub buy_volume_6h: u64,      // buy volume in last 6 hours
    pub buy_volume_24h: u64,     // buy volume in last 24 hours

    pub sell_volume_1m: u64,     // sell volume in last 1 minute
    pub sell_volume_5m: u64,     // sell volume in last 5 minutes
    pub sell_volume_1h: u64,     // sell volume in last 1 hour
    pub sell_volume_6h: u64,     // sell volume in last 6 hours
    pub sell_volume_24h: u64,    // sell volume in last 24 hours
}
```

### 1.4 100 first buy txs

- Column family: token
- Query key: `eb:{mint}`

```rust
pub struct TokenEarlyBuyers {
    pub id: String,               // Same as mint
    pub mint: String,             // Token mint address
    pub buy_txs: Vec<FirstBuyTx>, // Up to 100 early buy transactions
    pub last_updated_time: i64,   // Unix timestamp
    pub count: usize,             // Cached length to avoid counting the vec
}

pub struct FirstBuyTx {
    pub payer: String,     // Buyer address (payer of the transaction)
    pub signature: String, // Solana transaction signature
    pub volume: u64,       // SOL volume (amount_in)
    pub timestamp: i64,    // When the trade happened
}
```

## 2. Creator / Trader

### 2.1 Creator

- Column family: creator
- Query key: `c:{creator}`

```rust
pub struct CreatorTokenStats {
    pub id: String,                         // creator id (pubkey as string)
    pub last_updated_slot: u64,             // slot when last updated

    pub tokens_created_by_creator: Vec<String>,  // list of token mints created
    pub tokens_created_count: usize,             // number of tokens created

    pub tokens_migrated_by_creator: Vec<String>, // list of token mints migrated
    pub tokens_migrated_count: usize,            // number migrated

    pub tokens_with_mc_over_1m: Vec<String>,     // tokens with market cap > 1M
    pub tokens_with_mc_1m_count: usize,          // count for > 1M

    pub tokens_with_mc_over_5m: Vec<String>,     // tokens with market cap > 5M
    pub tokens_with_mc_5m_count: usize,          // count for > 5M

    pub tokens_with_mc_over_10m: Vec<String>,    // tokens with market cap > 10M
    pub tokens_with_mc_10m_count: usize,         // count for > 10M
}
```

### 2.2 Trader

- Column family: trader
- Query key: `t:{trader}`

```rust
pub struct Trader {
    pub id: String,                         // record id (pubkey as string)
    pub pubkey: String,                     // trader public key

    pub deposit_time: i64,                  // latest deposit time
    pub deposit_details: HashMap<String, f64>, // payer -> amount deposited
    pub deposit_exchange: Option<String>,   // inferred exchange (e.g., OKX)
    pub deposit_label: Option<String>,      // additional label (e.g., "OKX: Hot Wallet")


    pub last_trade_time: i64,               // latest trade time
    pub last_updated_timestamp: i64,        // when this record last updated

    // Total statistics (since account creation)
    pub total_volume: u64,                  // cumulative traded volume
    pub total_trades: u64,                  // cumulative number of trades

	  pub activity: VecDeque<TraderActivityEvent>, // last 20 buy/sell activity
}

pub struct TraderActivityEvent {
    pub slot: u64,           // slot when activity occurred
    pub timestamp: i64,      // unix time of activity
    pub mint: String,        // token mint traded
    pub is_buy: bool,        // true if buy, false if sell
    pub amount_in: u64,      // input amount
    pub amount_out: u64,     // output amount
}
```

## 3. Trade

### 3.1 Trade in slot

- Column family: trade
- Applies to: while token is bonding and for 5 minutes after migrating
- Query key: `ts:{mint}:{slot}`

```rust
pub struct TradeInSlot {
    pub id: String,                   // record id for (mint, slot)
    pub mint: String,                 // token mint address
    pub program: String,              // source program/market for trades in this slot
    pub count: u64,                   // number of trades observed in slot
    pub total_volume: u64,            // total traded volume in slot
    pub last_updated_slot: u64,       // slot when last updated
    pub signature_logs: Vec<SignatureLog>, // per-transaction breakdown
    pub bonded: bool,                 // whether token was migrated (bonded)
    pub is_bundle: bool,              // true if a bundle of txs in same slot
}
```

```rust
pub struct SignatureLog {
    pub signature: String,        // transaction signature
    pub signers: HashSet<String>, // list of signer addresses
    pub instructions: Vec<String>,// actions in transaction (buy, sell, create)

    pub buy_amt_sol: f64,         // total SOL bought in this tx
    pub sell_amt_sol: f64,        // total SOL sold in this tx
}
```

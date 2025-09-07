# Pulstream documentation

## Welcome to Pulstream 🚀

**Pulstream** is a high-performance stream processing platform built for real-time Solana data analysis and signal generation. It enables developers to create, deploy, and monetize custom data processing streams using WASM functions that react to blockchain events in real-time.

## ✨ Key features

- **⚡ Real-time Solana data processing**: Direct integration with Solana blockchain via Yellowstone gRPC geyser plugin
- **🔧 WASM-based functions**: Write logic in Rust, compile to WASM for sandboxed execution
- **📡 Low latency signal generation**: Sub-second detection and propagation of trading opportunities
- **📱 Automated social alerts**: Push notifications to Discord, Telegram, Twitter and other platforms
- **🤖 Automated trading execution**: Direct integration with DEXs and trading protocols
- **💰 Stream monetization**: Revenue sharing model for stream creators

## 🎯 Use cases
- **Trading signal generation**: Custom indicators and trading signals based on on-chain data
- **Arbitrage detection**: Identify price discrepancies across DEXs in real-time
- **DeFi analytics**: Track liquidity, yields, and protocol metrics
- **Portfolio tracking**: Monitor wallet activities and performance
- **Risk management**: Build alerts for position monitoring and risk metrics
- **Whale tracking**: Detect large transactions and wallet movements

## 🏗️ Architecture overview

Pulstream consists of four main components:

* **Carbon router** - Ingests real-time Solana blockchain data
* **Runtime engine** - Executes user functions in WASM environment  
* **Function server** - Manages function deployment via REST API
* **Signal distribution** - Delivers signals to consumers

[Learn more about the architecture →](./architecture/overview.md)

## 📚 Documentation overview

| Section | Description |
|---------|-------------|
| [🚀 Getting Started](./getting-started/quickstart.md) | Quick start guides and tutorials |
| [📋 DSL System](./dsl-system/structure.md) | Complete DSL system documentation |
| [🏗️ Architecture](./architecture/overview.md) | System design and components |
| [💻 Development](development/README.md) | Building and testing streams |
| [💰 Monetization](monetization/README.md) | Earn from your streams |

## ⚡ Quick start

Ready to build your first stream? Get started in just 5 minutes!

[**→ Go to Quickstart guide**](getting-started/quickstart.md)

## 🤝 Community & support

- [Twitter](https://x.com/pulstream_so)
- [GitHub](https://github.com/pulstream)
- [Website](https://pulstream.so)
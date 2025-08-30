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

### Trading & DeFi
- **Trading signal generation**: Custom indicators and trading signals based on on-chain data
- **Arbitrage detection**: Identify price discrepancies across DEXs in real-time
- **DeFi analytics**: Track liquidity, yields, and protocol metrics

### Portfolio & Risk management
- **Portfolio tracking**: Monitor wallet activities and performance
- **Risk management**: Build alerts for position monitoring and risk metrics
- **Whale tracking**: Detect large transactions and wallet movements

## 🏗️ Architecture overview

Pulstream consists of several interconnected components:
- **Carbon router**: Core ingestion service for Solana data
- **Runtime engine**: WASM execution environment
- **Function server**: RESTful API for function management
- **Signal distribution**: Real-time signal delivery system

[Learn more about the architecture →](architecture/README.md)

## 🚀 Get started in 5 minutes

### 1. Create your first stream

```json
{
  "rules": [
    {
      "conditions": {
        "type": "all",
        "rules": [
          {
            "field": "token_name",
            "operator": "starts_with",
            "value": "PEPE"
          }
        ]
      }
    }
  ]
}
```

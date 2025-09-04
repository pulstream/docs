{% mermaid %}
graph TB
    Solana[Solana blockchain]
    
    Solana --> YS[Yellowstone<br/>gRPC client]
    YS --> Router[Carbon router<br/>Rust native]
    
    Router --> RocksDB[RocksDB<br/>State management]
    Router --> Redpanda[Redpanda<br/>Mainstream]
    
    Redpanda --> Runtime[Runtime<br/>Wasm engine]
    RocksDB --> Runtime
    Runtime --> FS[Function server]
    Runtime --> Signals[Redpanda<br/>Signal stream]
    Redpanda --> SignalDistribution[Signal<br/>distribution]
    Signals --> SignalDistribution
    FS <--> UI
    SignalDistribution --> UI
{% endmermaid %}
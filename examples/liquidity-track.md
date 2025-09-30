**What we do:**  
- Create a stream to track whether tokens created by address **9k8jWWqfmTTXrc8gmZqYVxSFJhygURvyVRztivJXFTYP** experience rapid liquidity growth.
- Send a Telegram alert.

### Step 1: Create a stream using JSON DSL  
- Go to Pulstream, and login using wallet
- Create a new stream
- Paste the JSON code to playground

```json
{
  "topic_subscription": "wasm.token.liquidity-pulse",
  "state": [
    {
      "state_name": "mint",
      "source": "event_data",
      "key": "mint"
    },
    {
      "state_name": "name",
      "source": "event_data",
      "key": "name"
    },
    {
      "state_name": "symbol",
      "source": "event_data",
      "key": "symbol"
    },
    {
      "state_name": "creator",
      "source": "event_data",
      "key": "creator"
    },
    {
      "state_name": "creation_time",
      "source": "event_data",
      "key": "creation_time"
    },
    {
      "state_name": "creator_balance",
      "source": "event_data",
      "key": "creator_balance"
    }
  ],
  "conditions": {
    "type": "all",
    "rules": [
      {
        "field": "name",
        "operator": "equal",
        "value": "PUMP"
      },
      {
        "field": "creator_balance",
        "operator": "greater_than",
        "value": 1
      }
    ]
  }
}
```


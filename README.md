# Terraleaf staking

This is general purpose staking smart contract. 
It can be used for LP staking, SAS staking or governance. 

The reward for staked tokens is distributed proportionally on staking amount and duration.
The reward can be claimed every block. 

There is also an unbonding period which sets how long the staked
tokens are frozen before being released. These frozen tokens can neither
be used for voting, nor claimed by the original owner. After the period
can you get your tokens back or if you pay instant claim fee.

## Messages

### Init

```js
{
  "admins": [], // List of addresses that can update the contract config
  "staking_token": "terra1...",
  "reward_token": "terra1...", 
  "unbonding_period": 123, // How long staking tokens are frozen before release (in seconds)
  "instant_claim_percentage_loss": 5, // Instead of waiting for token release user can pay this tax and release them immediately
  "fee_recipient": "terra1...", // All fees will go to this address, it can withdraw them at any time from smart contract
  
  // Stablecoin fee will always be taken from the transaction sender for the operations that are configured here
  "stablecoin_action_fee": {
    "unbond": {
      "amount": "300000",
      "denom": "uusd"
    },
    "claim": {
      "amount": "300000",
      "denom": "uusd"
    }
    ...
  },

  // How much of reward tokens will be distributed amount stakers for particular period of time
  "reward_distribution_schedule": [
    {
       "amount": "100000",
       "start_time": 1234,
       "end_time": 5646
    },
    {
       "amount": "200000",
       "start_time": 1234,
       "end_time": 5646
    },
    ...
  ]
}
```

### Execute messages

#### Update Config 

Updates the contract config. Only admins can execute this message
It can be used to modify parameters defined in `Init` function.

```js
{
  "update_config": {
    // ... All the fields from the init message might be pasted here
  }
}
```

#### Unbond 

Unbound certain amount of staked tokens. 
Tokens are not released until unbonding period passes.

```js
{
  "unbond": {
    "tokens": "12345"
  }
}
```

#### Claim 

Transfer released tokens to sender wallet. 
Tokens has to be unbonded first, then can be claimed after unbonding period.

```js
{
  "claim": {}
}
```

#### Instant claim 

Transfer released tokens decreased by amount of percentage fee to sender wallet.
Tokens has to be unbonded first, then can be claimed immediately with percentage fee. 
In order to avoid percentage fee, user can use `claim` and wait unbonding period.

```js
{
  "instant_claim": {}
}
```

#### Withdraw reward

Withdraw accumulated reward to sender wallet.

```js
{
  "withdraw_reward": {}
}
```

#### Withdraw fee 

Withdraw accumulated reward to fee recipient wallet.

```js
{
  "withdraw_fee": {}
}
```

### Query messages

#### Config 

Returns smart contract configuration.

```js
// Query
{
  "config": {}
}

// Response
{
  // ... All the fields from the init message
}
```

#### State 

Returns smart contract state.

```js
// Query
{
  "state": {}
}

// Response
{
  "total_stake": "122424", // The amount of total staked tokens
  "last_updated": 123242, // The timestamp of last stake

  // The index used to calculate staker reward. 
  // The index is amount of reward per single staked token
  "global_reward_index": "12245", 
  "num_of_members": 1244 // The number of stakers
}
```

#### Member 

Returns the staker details

```js
// Query
{
  "member": {
    "address": "terra1..."  
  }
}

// Response
{
  "stake": "12242", // The amount of currently staked tokens
  "reward": "1224", // Accumulated reward
  "reward_index": "43", // Index used to calculate reward
  "withdrawn_reward": "23", // The amount of already withdrawn reward tokens
  
  // The list of unbonded tokens with their release time
  "unbonded_tokens": [
    {
      "amount": "12324",
      "release_at": 1324 
    },
    ...
  ]
}
```

#### ListMembers 

Returns list of stakers

```js
// Query
{
  "member": {
    // The optional field used for pagination (default value is 10, max value is 30 due to query nodes limitation) 
    "limit": 30,
    // The optional field used for pagination, used to start pagination from certain member address
    "start_after": "...."
  }
}

// Response
{
  "members": [
    {
      "address": "terra1...",
      "stake": "12242",
      "reward": "1224", 
      "reward_index": "43",
      "withdrawn_reward": "23",
      "unbonded_tokens": [
        {
          "amount": "12324",
          "release_at": 1324
        },
        ...
      ]
    },
    ...
  ]
}
```
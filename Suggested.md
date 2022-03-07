Merkle [Tree, Root]
    Need to have more regarding the rewards structure

Rewards

Rewards Epoch

Make sure 'notional value' makes sense in the blog post

'Tokenized' (as in tokenized cash flows)


## Swuniswap:

Range
Underlying Rate
Theta: Time Decay
Delta: Rate Sensitivity
Vega

'Principal Token' - zcToken
'Yield Token' - nToken

Automatic Market Maker (AMM)

Impermanent Loss

liquidity ticks: 
(from Uniswap Docs): "Ticks are the boundaries between discrete areas in price space. Ticks are spaced such that an increase or decrease of 1 tick represents a 0.0001% increase or decrease in price at any point in price space.

Ticks function as boundaries for liquidity positions. When a position is created, the provider must choose the lower and upper tick that will represent their position's borders."


Market:

Underlying: The underlying token address

Maturity: The Swivel market maturity

Decimals: The number of underlying token decimals

Network: The network on which you are market-making

Position:

UpperRate: The highest rate at which to quote

LowerRate: The lowest rate at which to quote

Amount: The amount of nTokens to use market-making

NumTicks: The number of liquidity ticks to split your amount into

ExpiryLength: How often orders should be refreshed (in seconds)

CompoundRateLean: How much your quote should change when Compound’s rate varies (e.g. 1 = 1:1 change in price)
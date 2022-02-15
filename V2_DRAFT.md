# Swivel Ubiquitous Language

## Table of Contents
1. [Shared Language](#shared-language)
2. [UI Language](#ui-language)
3. [Technical Language](#technical-language)
4. [Compund Language](#compound-language)
5. [Blockchain Terms](#blockchain-terms)

## Conventions
Terms are defined within their specified contexts. If encountered in a context but not explicitly re-defined or clarified for that context, the term is assumed to have the Shared Language definition.

Defined terms that occur within other terms' definition are indicated using title case and `Markup` formatting.

## Shared Language
These terms fulfill the same meaning across the Swivel stack, although any variation or further clarification will be noted within specific contexts.

### Asset
A currency, token, or other tradable object. In Swivel, `asset` refers to `Underlying` currency or Tokens.

### Underlying
The lending currency of a given `Market`, typically a `Stable Coin` or `Token`.

### Token
An `Erc20` compatible token deployed at a given Ethereum Address. May also be referenced as _Erc20_ or _uToken_.

### nToken
Swivel's native yield-bearing token, representing the remaining yield potential of the `Underlying` for a given `Market`. For example: `nDAI`, `nUSDC`.

### zcToken
Swivel's native principal token, representing the principal lent to a given `Market`. For example: `zcDAI`, `zcUSDC`.

### Balance
Amount of the `Underlying` or `Token` assets a given public key owns.

### Market
An asset-duration (underlying-duration) pairing that identifies which `Orderbook` / instrument is being used.

### Maturity
The duration a given `Market` generates yield, up to a specified date.

### Volume
All orders expose an amount which can be filled, wholly or partially, by an `Agreement`. This will be represented by either the `Principal` or `Interest`, depending on whether the order is `Amplified` (deprecated: 'floating') or `Fixed`.

### Yield
The amount of interest an `Agreement` has generated, expressed in terms of a percentage.

  
### Principal
### Premium

<hr>

## UI Language
These term definitions are understood in the context of the public interface at Swivel.finance.

### Lend
You can lend `Underlying` assets by placing a `Fixed Yield` order.

### Order
In the Front End context, an Order is any valid `Market Order` or `Limit Order` placed on a given `Market` to lend, buy or sell `Underlying` or `Token` assets.

### Orderbook
The table of valid, open `Orders` for a given `Market`

### Fill
Fill orders are `Orderbook` Orders that fulfill your `Market Order` or `Limit Order` partially or in full.

### Fixed Yield
A Fixed Yield order uses the Swivel protocol to give the user an immediate yield for lending `Underlying` assets. When a Fixed Yield order is filled, Swivel mints an equal amount of `nToken` and `zcToken` on the `Underlying`, leaving the `zcToken` in your balance and immediately selling the `nTokens` on the `Orderbook`, and returning the `Underlying` to your `Balance`.

### Amplified Yield (deprecated: 'Floating')
An Amplified Yield order purchases `nTokens` from the marketplace and adds them to your `Balance`, where they will continue to earn a variable yield until the given `Market` matures.

### Sell
You can sell your `nTokens` or `zcTokens` back to the `Orderbook` at any time with a Sell order.

### Market Order
Placing an `Order` as a `Taker`, transacting at the best available price on the `Orderbook`.

### Limit Order
Placing an `Order` at a determined price for the given order type and `Market`. `Valid`, unfilled `Limit Orders` appear in your Open Limit Orders view. Depending on `Order` size and `Orderbook` volume, Limit Orders may be filled partially over an undetermined amount of time.

### Position
The current state of your balance of `Underlying` and `Token` assets, for all active `Markets` you participate in.

<hr>

## Technical Language
These term definitions are understood in the context of Swivel's technical stack. UI terms above apply when seen in technical usage, unless re-defined within this Technical context.

### Order
An Entity, stored off chain.
UI analogs: *`Order`*, *`Market Order`,* *`Limit Order`*
#### Order Struct:
* **Key** - Keccack hash of (wallet address,nonce,time). May also be referred to as _orderKey_ when needed to differentiate from an Agreement's key.
* **Maker** - Public key of this Order's creator
* **Underlying** - Ethereum address of a deployed Erc20 token
* **Floating** - Boolean indicating if this order is floating or fixed side
* **Principal** - Volume to be filled in a floating side order. The amount of currency lent at a fixed-rate.
* **Interest** - Volume to be filled in a fixed side order.  The amount of currency necessary to back a fixed-rate.
* **Duration** - Timestamp indicating the the length of time this order is valid. Used to calculate an Agreement's release.
* **Expiry** - Timestamp marking this order's expiration

### Agreement
An Entity, stored on chain.

#### Agreement Struct:
* **Key** - Keccack hash of (wallet address,nonce,time). May also be referred to as _AgreementKey_ when needed to differentiate from an Order key.
* **Maker** - Public key of the creator of an Order this Agreement is filling
* **Taker** - Public key of this Agreement's creator
* **Underlying** - Ethereum address of a deployed Erc20 token
* **Floating** - Boolean indicating if this Agreement is Floating or Fixed side
* **Released** - Boolean indicating if this Agreement has been released or not
* **Principal** - Avalailable volume to be filled in a floating side order. When fixed this amount is determined by the interest / rate.
* **Interest** - Avalailable volume to be filled in a fixed side order.  When floating this amount is determined by the rate * principal.
* **Duration** - Timestamp indicating the the length of time this Agreement is valid
* **Release** - Timestamp marking this Agreement's term maturation. Calculated from the Duration of the Order this is filling

### Taker
This will be the public key of the owner of any `Agreement` created to fill an `Order`. Taker can be used in combination with other language as a specifier. _Taker volume_ for example, or _Taker side_.

In UI terms a user placing a Market Order is a "Taker".

### Cancelled
Only an `Order` may be cancelled. If it is, an entry into the on-chain _cancelled_ mapping is stored. No `Agreements` will then be accepted for this order. 

### Cancel Event
Emitted on chain upon the cancellation of an `Order` with the `Order`'s key.

### Expired
An Order which has aged past its Duration. Like a cancelled Order, it may no longer be filled by an Agreement.

### Signature
As ECDSA cryptographic signature following the EIP-712 standard which we use to verify the authenticity of an `Order`. You will see the `Components` of a Signature (V, R, and S) used throughout the protocol. The Hash and Sig solidity libraries exist to service this functionality. It should be noted and enforced that `Components` are derived from a `Signature` and that the two are not equivalent. a `Signature` is a 64-bytes and its derived `Components` are a _tuple_ of length 3 consisting of two 32-bytes, the canonical _R_ and _S_,  and a single uint _V_.

An Order's signature is considered `valid` if the extracted public key matches that of the maker.

### EIP-712 Domain
Mentioned here as the term `Domain` has very specific Domain Driven Design implications. You may see or hear the term domain intermingled with other verbage for the EIP-712 standard such as:
* TypeHash
* Separator
* Message

Any use of these terms, specifically _domain_ should be prefixed with EIP-712 to prevent language degradation and the confusion that follows it.

Implementation of Signature and Components are each tightly coupled to the EIP-712 standard.

### Valid
An `Order` is only valid if 
* non-cancelled, 
* non-expired, 
* not-fully-filled and 
* passes signature validation.

### Initiate Event
Emitted on chain upon the establishment of any `Agreement`. Publishes both `OrderKey` and `AgreementKey`

### Release Event
Emitted on chain upon the release of an Agreement. Publishes OrderKey and AgreementKey

### Approve
Terms such as approve and approval(s) when referencing specific functionality of the `Underlying` token. Use specifiers to avoid confusion with the compound token if needed. _underlying approval_ for example.

### Transfer
Terms such as transfer, transfers  and transfer from when referencing specific functionality of the `Underlying` token. As with approve, specify which token if necessary, _uToken transfer_ etc...

### Side
Indicates whether the bond is `fixed` and is lending `principal`, or is `floating` and is paying `interest`.

### Agreement

### Exit
Boolean value indicating that an `Order` is Selling `nTokens` of `zcTokens` back to the `Orderbook`.

### Vault
### Initiate
Boolean value indicating that an order is `Lending` new `Underlying` assets to the protocol, minting new `nTokens` and `zcTokens` into the `Vault`.

### Rate (Effective Rate)
The bond's (deprecated?) effective rate, averaged across each agreement. Depending on whether the bond is `fixed` or `floating` (UI: '`amplified`)', effective rate is calculated with respect to either `principal` or `interest`.

Calculated as the sum of each agreement's ( `agreement volume` / `bond volume` ) * `agreement rate` )

e.g. 

If (`agreement floating` == false) {

   effectiveRate = SUM(`agreement principal`/`bond volume`) * `agreement principal`)
  
}

### Deployments
The Swivel development team has a number of deploys for Production, Staging, and Development.

#### Production
The product deployment. This is used for all live user-facing deployments on any given chain or L2. e.g. `production.swivel.exchange` (updated from previous lang `mainnet.swivel.exchange`)

#### Staging
The staging deployment is a our advertised test net. Reflects promises about being a mirror of production, but without real money risk. It should be stable and function exactly like production. E.g. `staging.swivel.exchange` (updated from previous lang `rinkeby.swivel.exchange`)

#### Development
The Development deployment is a private, non-public facing Testnet version of the product. It has no promises about functionlity and will frequently break. E.g. `development.swivel.exchange`.


<hr>

## Compound Language
Some terms used in the context of the Compound protocol, and how Swivel utilizes it.

### Compound Token
A CErc20 at an address passed to the constructor of the Swivel contract at deployment.

Pass an array of underlying token addresses + array of CERC20 addresses to create a public mapping of address -> address 

### Mint
When an agreement is initiated and after funds have been sent to the Swivel contract, cTokens are minted on the Compound protocol to generate interest. 

The number of cTokens minted is calculated in terms of the underlying token consumed/transferred rather than the # of cTokens to be minted.

### Redeem
When an agreement has completed and "Release" is called, cTokens are redeemed on the Compound protocol to return an agreement's underlying deposit as well as the interest generated over the agreement's lending term. 

The number of cTokens redeemed is calculated in terms of the underlying token to be returned, rather than the # of cTokens to be consumed/transferred.

### Exchange Rate
Exchange Rate refers to the 2e26+ uint returned from the Compound Protocol by exchangeRateCurrent().

In order to calculate the interest generated by an agreement, the current ratio of underlying:cToken is stored in each agreement upon its initiation(exchangeRateCurrent()). 

Over time this ratio increases such that when "Release" is called, the difference between stored / current exchange rate can directly calculate the yield an agreement has realized.

### CToken is Erc20
Note that all functionality of the Erc20 standard is available to the Compound Token. Use _CErc20_ or _cToken_ to specify.

<hr>

## BlockChain Terms
### Erc20
- Erc20 is a token contract standard on the Ethereum Virtual Machine (EVM) that defines mechanics and best practices for fungible, transferrable digital tokens. 
### Stable Coins
- Stable coins are a class of Erc20 tokens that maintain a consistent peg to the value of the US Dollar. For example DAI, USDC.
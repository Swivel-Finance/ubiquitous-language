# Ubiquitous Language
A domain driven, living, published language for the Swivel Finance Protocol.

## Order
This is an Entity, stored off chain, possessing the following properties:

* Key - Keccack hash of (wallet address,nonce,time). May also be referred to as _orderKey_ when needed to differentiate from an Agreement's key.
* Maker - Public key of this Order's creator
* Underlying - Ethereum address of a deployed Erc20 token
* Floating - Boolean indicating if this order is floating or fixed side
* Principal - Avalailable volume to be filled in a floating side order. When fixed this amount is determined by the interest / rate.
* Interest - Avalailable volume to be filled in a fixed side order.  When floating this amount is determined by the rate * principal.
* Duration - Timestamp indicating the the length of time this order is valid. Used to calculate an Agreement's release.
* Expiry - Timestamp marking this order's expiration

### Asset
A currency, token, or other tradable object. In our context `asset` refers to DAI, USDC, USDT.

### Market (Market-Pair)
An asset-duration (underlying-duration) pairing that identifies which orderbook / instrument is being used. E.g. DAI-12MONTH or 0xD0f2390b9CCd0A9F6160E4Ab584A4Fb51f76bF14 - 31536000

### Volume
All orders expose an amount which can be filled, wholly or partially, by an Agreement. This will be represented by
either the principal or interest, depending on whether the Order is floating or fixed.

### Side
A term used on occasion to designate whether an order is `fixed`, `floating` side.

### Taker
This will be the public key of the owner of any Agreement created to fill an Order. Taker can be used in
combination with other language as a specifier. _Taker volume_ for example, or _Taker side_.

### Cancelled
Only an Order may be cancelled. If it is, an entry into the on-chain _cancelled_ mapping is stored. No Agreements
will then be accepted for this order. 

#### Cancel Event
Emitted on chain upon the cancellation of an Order with the Order's key.

### Expired
An Order which has aged past its Duration. Like a cancelled Order, it may no longer be filled by an Agreement.

### Signature
As ECDSA cryptographic signature following the EIP-712 standard which we use to verify the authenticity of an Order. You
will see the `Components` of a Signature (V, R, and S) used throughout the protocol. The Hash and Sig solidity libraries
exist to service this functionality. It should be noted and enforced that `Components` are derived from a `Signature` and that
the two are not equivalent. a `Signature` is a 64-bytes and its derived `Components` are a _tuple_ of length 3 consisting of
two 32-bytes, the canonical _R_ and _S_,  and a single uint _V_.

A Order's signature is considered `valid` if the extracted public key matches that of the maker.

#### EIP-712 Domain
Mentioned here as the term `Domain` has very specific Domain Driven Design implications. You may see or hear the term domain
intermingled with other verbage for the EIP-712 standard such as:
* TypeHash
* Separator
* Message

Any use of these terms, specifically _domain_ should be prefixed with EIP-712 to prevent language degredation and the confusion that follows it.
Obviously our implementation of Signature and Components are tightly coupled to the EIP-712 standard.

### Valid
An Order is only valid if non-cancelled, non-expired, not-fully-filled and passing signature validation.

---

## Agreement
An Entity, stored on chain, with the following properties:

* Key - Keccack hash of (wallet address,nonce,time). May also be referred to as _AgreementKey_ when needed to differentiate from an Order key.
* Maker - Public key of the creator of an Order this Agreement is filling
* Taker - Public key of this Agreement's creator
* Underlying - Ethereum address of a deployed Erc20 token
* Floating - Boolean indicating if this Agreement is Floating or Fixed side
* Released - Boolean indicating if this Agreement has been released or not
* Principal - Avalailable volume to be filled in a floating side order. When fixed this amount is determined by the interest / rate.
* Interest - Avalailable volume to be filled in a fixed side order.  When floating this amount is determined by the rate * principal.
* Duration - Timestamp indicating the the length of time this Agreement is valid
* Release - Timestamp marking this Agreement's term maturation. Calculated from the Duration of the Order this is filling

#### Initiate Event
Emitted on chain upon the establishment of any Agreement. Publishes both OrderKey and AgreementKey

### Release
After the lending term of an Agreement has passed it may be released, paying amounts owed to all parties involved.

### Active
The descriptive state of an agreement that has not yet been released.

#### Release Event
Emitted on chain upon the release of an Agreement. Publishes OrderKey and AgreementKey

### Yield
The amount of interest an agreement has generated in terms of a percentage.

## Underlying
An Erc20 compatible token deployed at a given Ethereum Address. May also be referenced as _Erc20_ or _uToken_.

### Approve
Terms such as approve and approval(s) when referencing specific functionality of the Underlying token. Use specifiers to avoid
confusion with the compound token if needed. _underlying approval_ for example.

### Transfer
Terms such as transfer, transfers  and transfer from when referencing specific functionality of the Underlying token.
As with approve, specificy which token if necessary, _uToken transfer_ etc...

### Balance
Amount of the underlying token a given public key owns. Specify which token to avoid confusion.

## Compound Token
A CErc20 at an address passed to the constructor of the Swivel contract at deployement.
* Perhaps pass an array of underlying token addresses + array of CERC20 addresses to create a public mapping of address -> address 

### Mint
When an agreement is initiated and after funds have been sent to the Swivel contract, cTokens are minted on the Compound protocol to generate interest. 

The number of cTokens minted is calculated in terms of the underlying token consumed/transferred rather than the # of cTokens to be minted.

### Redeem
When an agreement has completed and "Release" is called, cTokens are redeemed on the Compound protocol to return an agreements underlying deposit as well as the interest generated over the agreement's lending term. 

The number of cTokens redeemed is calculated in terms of the underlying token to be returned, rather than the # of cTokens consumed/transfered.

### Exchange Rate
Exchange Rate refers to the 2e26+ uint returned from the Compound Protocol by exchangeRateCurrent().

In order to calculate the interest generated by an agreement, the current ratio of underlying:cToken is stored in each agreement upon its initiation(exchangeRateCurrent()). 

Over time this ratio increases such that when "Release" is called, the difference between stored / current exchange rate can directly calculate the yield an agreement has realized.

#### CToken is Erc20
Note that all functionality of the Erc20 standard is available to the Compound Token. Use _CErc20_ or _cToken_ to specify.

---

## Bond 
A bucket of agreements associated with a given orderKey or agreementKey. 

### Market (Market-Pair)
An `asset`-`duration` (underlying-duration) pairing that identifies which orderbook / instrument the bond originated from. E.g. DAI-12MONTH or 0xD0f2390b9CCd0A9F6160E4Ab584A4Fb51f76bF14 - 31536000

### Side
Indicates whether the bond is `fixed` and is lending `principal`, or is `floating` and is paying `interest`.

### Volume
The total amount of `asset` under a given bond. Depending on whether the bond is `fixed` or `floating`, volume is the sum of all `principal` or `interest` across each agreement.

### Rate (Effective Rate)
The bond's effective `rate`, averaged across each agreement. Depending on whether the bond is `fixed` or `floating`, effective rate is calculated with respect to either `principal` or `interest`.

Calculated as the sum of each agreement's ( `agreement volume` / `bond volume` ) * `agreement rate` )

e.g. 

If (`agreement floating` == false) {

   effectiveRate = SUM(`agreement principal`/`bond volume`) * `agreement principal`)
  
}

### Release
Timestamp of the agreement with the with the highest `release` timestamp. Represents the time at which *all* agreements within the bond can be released.

# Ubiquitous Language
A domain driven, living, published language for the Swivel Finance Protocol.

## Order
This is an Entity, stored off chain, possessing the following properties:

* Key - Keccack hash of (@jules). May also be referred to as _OrderKey_ when needed to differentiate from an Agreement's key.
* Maker - Public key of this Order's creator
* Underlying - Ethereum address of a deployed Erc20 token
* Floating - Boolean indicating if this Order is floating or fixed side
* Principal - Avalailable volume to be filled in a floating side order. When fixed this amount is [@jules]
* Interest - Avalailable volume to be filled in a fixed side order.  When floating this amount is [@jules]
* Duration - Timestamp indicating the the length of time this Order is valid. Used to calculate an Agreement's release.
* Expiry - Timestamp marking this Order's expiration
* Nonce - @jules

### Volume
All Orders expose an amount which can be filled, wholly or partially, by an Agreement. This will be represented by
either the principal or interest, depending on whether the Order is floating or fixed.

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
An Order is only valid if non-cancelled, non-expired and passing signature validation.

#### other order specific terms...


## Agreement
An Entity, stored on chain, with the following properties:

* Key - Keccack hash of (@jules). May also be referred to as _AgreementKey_ when needed to differentiate from an Order key.
* Maker - Public key of the creator of an Order this Agreement is filling
* Taker - Public key of this Agreement's creator
* Underlying - Ethereum address of a deployed Erc20 token
* Floating - Boolean indicating if this Agreement is Floating or Fixed side
* Released - Boolean indicating if this Agreement has been released or not
* Rate - @jules
* Principal - Amount of volume this Agreement is filling in a floating side Order. When fixed this amount is [@jules]
* Interest - Amount of volume this Agreement is filling in a fixed side Order.  When floating this amount is [@jules]
* Duration - Timestamp indicating the the length of time this Agreement is valid
* Release - Timestamp marking this Agreement's term maturation. Calculated from the Duration of the Order this is filling
* Nonce - @jules

#### Initiate Event
Emitted on chain upon the establishment of any Agreement. Publishes both OrderKey and AgreementKey

### Volume
The amount of an Order's volume this agreement is filling. Depending on floating or fixed side it is present as Principal or Interest.

### Release
After the term of an Agreement has passed it may be released, paying amounts owed to all parties involved. @jules...

#### Release Event
Emitted on chain upon the release of an Agreement. Publishes OrderKey and AgreementKey

### other agreement language...

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
A CErc20 at an adress passed to the constructor of the Swivel contract at deployement. @jules... 

### Mint
@jules...

### Redeem
@jules...

### Exchange Rate
@jules...

#### CToken is Erc20
Note that all functionality of the Erc20 standard is available to the Compound Token. Use _CErc20_ or _cToken_ to specify.

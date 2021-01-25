# 2021-01-25


## Install

To install the command line tools, please see [dcrinstaller](https://github.com/decred/decred-release/tree/master/cmd/dcrinstall).
To install decrediton download, uncompress, and run [decrediton Linux](https://github.com/decred/decred-binaries/releases/download/v1.6.0/decrediton-v1.6.0.AppImage) or [decrediton macOS](https://github.com/decred/decred-binaries/releases/download/v1.6.0/decrediton-v1.6.0.dmg) or [decrediton Windows](https://github.com/decred/decred-binaries/releases/download/v1.6.0/decrediton-v1.6.0.exe).

See manifest-v1.6.0.txt, and the package specific manifest files for sha256 sums and the associated .asc files to confirm those shas.

See [README.md](./README.md#verifying-binaries) for more info on verifying the files.


## Contents

* [dcrd](#dcrd-v160)
* [dcrwallet](#dcrwallet-v160)
* [decrediton](#decrediton-v160)
* [dcrlnd](#dcrlnd-v030)
* [dcrdex](#dcrdex-v014)


# dcrd v1.6.0

This release of dcrd introduces a large number of updates.  Some of the key
highlights are:

* A new consensus vote agenda which allows the stakeholders to decide whether or
  not to activate support for a decentralized treasury
* Aggregate fee transaction selection in block templates (Child Pays For Parent)
* Improved peer discovery via HTTPS seeding with filtering capabilities
* Major performance enhancements for signature validation and other
  cryptographic operations
* Approximately 15% less overall resident memory usage
* Proactive signature cache eviction
* Improved support for single-party Schnorr signatures
* Ticket exhaustion prevention
* Various updates to the RPC server such as:
  * A new method to retrieve the current treasury balance
  * A new method to query treasury spend transaction vote details
* Infrastructure improvements
* Quality assurance changes

For those unfamiliar with the
[voting process](https://docs.decred.org/governance/consensus-rule-voting/overview/)
in Decred, all code needed in order to support a decentralized treasury is
already included in this release, however it will remain dormant until the
stakeholders vote to activate it.

For reference, the decentralized treasury work was originally proposed and
approved for initial implementation via the following Politeia proposal:
- [Decentralized Treasury Consensus Change](https://proposals.decred.org/proposals/c96290a2478d0a1916284438ea2c59a1215fe768a87648d04d45f6b7ecb82c3f)

The following Decred Change Proposal (DCP) describes the proposed changes in
detail and provides a full technical specification:
- [DCP0006](https://github.com/decred/dcps/blob/master/dcp-0006/dcp-0006.mediawiki)

**It is important for everyone to upgrade their software to this latest release
even if you don't intend to vote in favor of the agenda.**

## Downgrade Warning

The database format in v1.6.0 is not compatible with previous versions of the
software.  This only affects downgrades as users upgrading from previous
versions will see a one time database migration.

Once this migration has been completed, it will no longer be possible to
downgrade to a previous version of the software without having to delete the
database and redownload the chain.

The database migration typically takes about 5 to 10 minutes on HDDs and 2 to 4
minutes on SSDs.

## Notable Changes

### Decentralized Treasury Vote

A new vote with the id `treasury` is now available as of this release.  After
upgrading, stakeholders may set their preferences through their wallet or Voting
Service Provider's (VSP) website.

The primary goal of this change is to fully decentralize treasury spending so
that it is controlled by the stakeholders via ticket voting.

See the initial
[Politeia proposal](https://proposals.decred.org/proposals/c96290a2478d0a1916284438ea2c59a1215fe768a87648d04d45f6b7ecb82c3f)
for more details.

### Aggregate Fee Block Template Transaction Selection (Child Pays For Parent)

The transactions that are selected for inclusion in block templates that
Proof-of-Work miners solve now prioritize the overall fees of the entire
transaction ancestor graph.

This is beneficial for both miners and end users as it:

- Helps maximize miner profit by ensuring that unconfirmed transaction chains
  with higher aggregate fees are given priority over others with lower aggregate
  fees
- Provides a mechanism for users to increase the priority of an unconfirmed
  transaction by spending its outputs with another transaction that pays higher
  fees

This is commonly referred to as Child Pays For Parent (CPFP) as the spending
("child") transaction is able to increase the priority of the spent ("parent")
transaction.

### HTTPS Seeding

The initial bootstrap process that contacts seeders to discover other nodes to
connect to now uses a REST-based API over HTTPS.

This change will be imperceptible for most users, with the exception that it
accelerates the process of finding suitable candidate nodes that support desired
services, particularly in the case of recently-introduced services that have not
yet achieved widespread adoption on the network.

The following are some key benefits of HTTPS seeders over the previous DNS-based
seeders:

- Support for non-standard ports
- Advertisement of supported service
- Better scalability both in terms of network load and new features
- Native support for TLS-secured communication channels
- Native support for proxies which allows the use of anonymous overlay networks
  such as Tor and I2P
- No need for a large DNSSEC dependency surface
- Uses better audited infrastructure
- More secure
- Increases flexibility

### Signature Validation And Other Crypto Operation Optimizations

The underlying crypto code has been reworked to significantly improve its
execution speed and reduce the number of memory allocations.  While this has
more benefits than enumerated here, probably the most important ones for most
stakeholders are:

- Improved vote times since blocks and transactions propagate more quickly
  throughout the network
- The initial sync process is around 15% faster

### Proactive Signature Cache Eviction

Signature cache entries that are nearly guaranteed to no longer be useful are
now immediately and proactively evicted resulting in overall faster validation
during steady state operation due to fewer cache misses.

The primary purpose of the cache is to avoid double checking signatures that are
already known to be valid.

### Orphan Transaction Relay Policy Refinement

Transactions that spend outputs which are not known to nodes relaying them,
known as orphan transactions, now have the same size restrictions applied to
them as standard non-orphan transactions.

This ensures that transactions chains are not artificially hindered from
relaying regardless of the order they are received.

In order to keep memory usage of the now potentially larger orphan transactions
under control, more intelligent orphan eviction has been implemented and the
maximum number of allowed orphans before random eviction occurs has been
lowered.

These changes, in conjunction with other related changes, mean that nodes are
better about orphan transaction management and thus missing ancestors will
typically either be broadcast or mined fairly quickly resulting in fewer overall
orphans and smaller actual run-time orphan pools.

### Ticket Exhaustion Prevention

Mining templates that would lead to the chain becoming unrecoverable due to
inevitable ticket exhaustion will no longer be generated.

This is primarily aimed at the testing networks, but it could also theoretically
affect the main network in some far future if the demand for tickets were to
ever dry up for some unforeseen reason.

### New Initial State Protocol Messages (`getinitstate`/`initstate`)

This release introduces a pair of peer-to-peer protocol messages named
`getinitstate` and `initstate` which support querying one or more pieces of
information that are useful to acquire when a node first connects in a
consolidated fashion.

Some examples of the aforementioned information are the mining state as of the
current tip block and, with the introduction of the decentralized treasury, any
outstanding treasury spend transactions that are being voted on.

### Mining State Protocol Messages Deprecated (`getminings`/`minings`)

Due to the addition of the previously-described initial state peer-to-peer
protocol messages, the `getminings` and `minings` protocol messages are now
deprecated.  Use the new `getinitstate` and `initstate` messages with the
`headblocks` and `headblockvotes` state types instead.

### RPC Server Changes

The RPC server version as of this release is 6.2.0.

#### New Treasury Balance Query RPC (`gettreasurybalance`)

A new RPC named `gettreasurybalance` is now available to query the current
balance of the decentralized treasury.  Please note that this requires the
decentralized treasury vote to pass and become active, so it will return an
appropriate error indicating the decentralized treasury is inactive until that
time.

See the
[gettreasurybalance JSON-RPC API Documentation](https://github.com/decred/dcrd/blob/master/docs/json_rpc_api.mediawiki#gettreasurybalance)
for API details.

#### New Treasury Spend Vote Query RPC (`gettreasuryspendvotes`)

A new RPC named `gettreasuryspendvotes` is now available to query vote
information about one or more treasury spend transactions.  Please note that
this requires the decentralized treasury vote to pass and become active to
produce a meaningful result since treasury spend transactions are invalid until
that time.

See the
[gettreasuryspendvotes JSON-RPC API Documentation](https://github.com/decred/dcrd/blob/master/docs/json_rpc_api.mediawiki#gettreasuryspendvotes)
for API details.

#### New Force Mining Template Regeneration RPC (`regentemplate`)

A new RPC named `regentemplate` is now available which can be used to force the
current background block template to be regenerated.

See the
[regentemplate JSON-RPC API Documentation](https://github.com/decred/dcrd/blob/master/docs/json_rpc_api.mediawiki#regentemplate)
for API details.

#### New Unspent Transaction Output Set Query RPC (`gettxoutsetinfo`)

A new RPC named `gettxoutsetinfo` is now available which can be used to retrieve
statistics about the current global set of unspent transaction outputs (UTXOs).

See the
[gettxoutsetinfo JSON-RPC API Documentation](https://github.com/decred/dcrd/blob/master/docs/json_rpc_api.mediawiki#gettxoutsetinfo)
for API details.

#### Updates to Peer Information Query RPC (`getpeerinfo`)

The results of the `getpeerinfo` RPC are now sorted by the `id` field.

See the
[getpeerinfo JSON-RPC API Documentation](https://github.com/decred/dcrd/blob/master/docs/json_rpc_api.mediawiki#getpeerinfo)
for API details.

#### Enforced Results Limit on Transaction Search RPC (`searchrawtransactions`)

The maximum number of transactions returned by a single request to the
`searchrawtransactions` RPC is now limited to 10,000 transactions.  This far
exceeds the number of results for all typical cases; however, for the rare cases
where it does not, the caller can make use of the `skip` parameter in subsequent
requests to access additional data if they require access to more results.

See the
[searchrawtransactions JSON-RPC API Documentation](https://github.com/decred/dcrd/blob/master/docs/json_rpc_api.mediawiki#searchrawtransactions)
for API details.

#### New Index Status Fields on Info Query RPC (`getinfo`)

The results of the `getinfo` RPC now include `txindex` and `addrindex` fields
that specify whether or not the respective indexes are active.

See the
[getinfo JSON-RPC API Documentation](https://github.com/decred/dcrd/blob/master/docs/json_rpc_api.mediawiki#getinfo)
for API details.

### Version 1 Block Filters Deprecated

Support for version 1 block filters is deprecated and is scheduled to be removed
in the next release.   Use
[version 2 block filters](https://github.com/decred/dcps/blob/master/dcp-0005/dcp-0005.mediawiki#version-2-block-filters)
with their associated [block header commitment](https://github.com/decred/dcps/blob/master/dcp-0005/dcp-0005.mediawiki#block-header-commitments)
and [inclusion proof](https://github.com/decred/dcps/blob/master/dcp-0005/dcp-0005.mediawiki#verifying-commitment-root-inclusion-proofs)
instead.

## Changelog

This release consists of 616 commits from 17 contributors which total to 526
files changed, 63090 additional lines of code, and 26279 deleted lines of code.

All commits since the last release may be viewed on GitHub
[here](https://github.com/decred/dcrd/compare/release-v1.5.2...release-v1.6.0).

See the [dcrd's own release notes](https://github.com/decred/dcrd/releases/tag/release-v1.6.0) for a cateogrized breakdown of all commits since the last release.

### Code Contributors (Alphabetical Order)

- Brian Stafford
- Dave Collins
- David Hill
- degeri
- Donald Adu-Poku
- Jamie Holdstock
- Joe Gruffins
- Josh Rickmar
- Julian Yap
- Marco Peereboom
- Matheus Degiovani
- Matt Hawkins
- Ryan Riley
- Ryan Staudt
- Wisdom Arerosuoghene
- Youssef Boukenken
- zhizhongzhiwai


# dcrwallet v1.6.0

This release focuses on adding support for the decentralized Decred treasury, 
improved SPV syncing with version 2 committed filters, and client support for 
the new privacy-conscious VSP implementation to make mixed VSP ticketbuying 
viable.

A comprehensive list of improvements and bug fixes follows.

## New Features

### Decentralized Treasury

* Support for the decentralized treasury consensus change is added.  Two new 
JSON-RPC methods `sendtotreasury` and `spendfromtreasury` are added, to send to 
and spend from value in the treasury, respectively.  The vote version and 
current agendas have been updated to allow stakeholders to vote on the 
activation of the decentralized treasury.

### SPV Mode

* Version 2 committed filters are now used, rather than the previous version 1 
filters.  These filters are consensus validated by proof-of-work miners as part 
of the commitments in the block header.  Version 2 filters are smaller and also 
do not require knowledge of the exact outputs spent, but rather only the 
previous output script (or address).

* The `fundrawtransaction` JSON-RPC method is now directly implemented by 
dcrwallet, rather than delegating this method to dcrd through RPC passthrough. 
 This allows the method to be usable under SPV mode.

* A `WalletService.GetRawCFilters` gRPC method was added to query the 
wallet-stored version 2 committed filter for specified blocks.

* Both a `getpeerinfo` JSON-RPC method and `WalletService.GetPeerInfo` gRPC 
method were implemented to provide peer info in SPV mode.  The JSON-RPC method 
continues to return results from a connected dcrd when syncing in RPC mode.

* A `sendrawtransaction` implementation has been added to the JSON-RPC server. 
This allows arbitrary transactions to be published under SPV mode.

### Staking

* A client for the new vspd server was added, and dcrwallet supports this 
client functionality from both the ticket autobuyer and through various gRPC 
methods.

* A `ticketinfo` JSON-RPC method was added to provide detailed status 
information regarding all tickets from the wallet.

* Vote preferences may now be specified on a per-ticket basis with added 
optional parameters to the `setvotechoice` JSON-RPC method.  This feature is 
used by the new vspd server.

* The `stakepooluserinfo` JSON-RPC method has been reintroduced, after being 
removed from prior releases.  This is used by the new vspd server.

* Unpublished transactions are now supported.  When an unpublished transaction 
is saved to the database, the outputs it spends are tallied in balance results 
and are not spendable by other transactions.  Unpublishd transactions will not 
be automatically rebroadast to the network when the wallet starts up and begins 
syncing.  Unpublished transactions are used to record transactions paying vspd 
fees prior to the vspd instance accepting the client's ticket request.

* A `--manualtickets` flag was added to the application config.  This setting 
disables discovering any tickets from the network syncing, instead requiring any
 tickets to be manually added to the wallet using `addtransaction`.  This 
feature is used by the new vspd server to avoid voting on unprocessed tickets 
which used a vspd voting address.  The current state of this setting is 
reported in the `walletinfo` JSON-RPC result.

* The `WalletService.PurchaseTickets` gRPC method gained a `dont_sign_tx` 
parameter to support unsigned ticket purchasing and eventual hardware wallet 
signing.

### Mixing

* An `AccountMixerService` was added to the gRPC server to perform CoinShuffle++
 mixing on all funds received by an account.

* The `WalletService.PurchaseTickets` method gained support for specifying 
CoinShuffle++ options for mixed ticket buying.

* The `getcoinjoinsbyacct` JSON-RPC method and 
`WalletService.GetCoinjoinOutputspByAcct` gRPC methods were added to discover 
probable CoinJoin transactions and report them by account.

### Other new Features

* A `createsignature` JSON-RPC method was introduced, analogous to the gRPC 
`WalletService.CreateSignature` method.

* A `discoverusage` JSON-RPC method was introduced, which triggers the same 
address and account discovery as performed on startup when there are new blocks 
available.  However, this method is more general purpose and is useful when 
correcting issues with prior discoveries, at it allows specifying the exact 
starting blocks and a BIP0044 gap limit to use.

* A `WalletService.SignHashes` gRPC method was added to sign an arbitrary number
 of 32-byte hashes.  This method was used by the now-defunct TumbleBit 
implementation.

* A `WalletService.Spender` gRPC method was added to query the transaction and 
input index which spends a wallet output.

* The `WalletService.TransactionNotifications` gRPC method now provides more 
details about the block headers which were detached during a reorganize, rather 
than only their hashes.

* An `addtransaction` JSON-RPC method was added, allowing transactions to be 
manually added to the wallet, mined in a specified block, without discovering 
the transaction through the network.

* A `NetworkService.GetRawBlock` gRPC method was added to fetch raw blocks 
using the wallet's peer-to-peer implementation.

* An optional account parameter was added to the `listunspent` and 
`listlockunspent` JSON-RPC methods to filter results for a particular account.

* A `walletpassphrasechange` JSON-RPC method was added to modify the wallet's 
public data encryption passphrase.  Changing to the default insecure value 
"public" effectively removes any prompts for the public passphrase at startup.

* The `listunspent` JSON-RPC method now includes the hex encoding of a redeem 
script when the output is P2SH and the redeem script is known.

* Accounts are now able to be encrypted using separate, per-account passphrases.
 Unlocking an account only provides access to that account's private keys, 
and no others.  Account passphrases may be set using the `setaccountpassphrase` 
JSON-RPC method, and locked and unlocked by the `unlockaccount` and 
`lockaccount` methods.

* JSON-RPC clients may now be authenticated using TLS client certificates, and 
this authentication is now required for the gRPC server.  The feature may be 
enabled for JSON-RPC by using the `--jsonrpcauthtype=clientcert` config flag. 
 Client certificates read from a `clients.pem` file in the application directory
 are trusted by default, and this file may be modified by the `--clientcafile` 
config flag.  Additionally, an `--issueclientcert` flag is provided which 
causes the wallet to issue and send an ephemeral client certificate and key 
over the TX pipe to the parent process which forked dcrwallet.  Client 
certificates may be generated by the `gencerts` tool, which is now part of the 
Decred CLI distribution.

* gRPC methods to lock and unlock the wallet's global keys and 
individually-encrypted accounts are now added, and the passphrase in all 
requests which required an unlocked wallet are now optional.  As the gRPC 
server now requires client authentication, there is no a risk of an 
unauthenticated client from quickly hitting an already-unlocked wallet or 
account and using private keys it should not otherwise have access to.

## Other Improvements

* Peer-to-peer seeding is now performed over an HTTPS API rather than DNS. 
 This improves reliability (HTTPS is authenticated), as well as greater control 
of filtering results by various URL parameters.

* The LOGFLAGS environment variable may now include a UTC flag to cause the 
wallet to always log with UTC timestamps, regardless of the current system 
timezone.

* Many log messages were added, removed, or rewritten to better reflect the 
operational state of the application.

* The scrypt KDF used for wallet encryption keys now defaults to weaker 
parameters on simnet.  This is primarily done for quicker unit tests, but will 
also affect real dcrwallet simnet instances by requiring less time and memory 
to derive keys.

* Imported scripts are now recorded in plain text and the wallet does not need 
to be unlocked to retrieve the full script for the P2SH address.  This change is
 made under the assumption that imported redeem scripts should not be secrets 
themselves, but still require a signature check at the very least.

* Importing an already-existing redeem script from the `importscript` JSON-RPC 
method no longer starts a rescan.

* Outputs which are being mixed are now locked to prevent accidental spending 
before the mix completes.

* Mix denominations above the ticket price are now avoided when mixing large 
value outputs.  This improves pairing with the large volume of mixes occurring 
from ticket buying, as there are many pairings occurring at the standard 
denominations below the ticket price to mix CoinJoin change outputs.

* Mixed ticketbuying using the autobuyer will now default to buy one ticket per 
client connection if the limit is unset.  Setting a larger limit will continue 
to buy at most limit number of tickets per client connection.

* Output locking no longer considers differences between the regular and stake 
transaction trees.

* Wallet setup may now be performed by providing answers to the prompts from 
piped output or a redirected file, as long as the passphrase is provided using 
the `--pass` flag.

* Newly created simnet wallets now always use the SLIP0044 coin type.  This 
ensures that the printed mining address during the creation process will not 
become invalid after a coin type upgrade from the legacy to the SLIP0044 coin 
type following address discovery.

* The latest peer-to-peer protocol version is now supported.  The `miningstate` 
and `initstate` messages which are expected in this version are replied to with 
empty responses.

* Ticket purchasing will now attempt to buy fewer tickets than requested when 
there is a low balance, either due to a bad estimate of how many tickets could 
be purchased, or due to outputs being reserved to pay the fees for the new 
vspd server.

## Bug Fixes

* A memory leak of requests and responses made to a dcrd websocket server was 
plugged.

* Imported xpub accounts no longer produce errors while trying to read the 
account's xpriv.

* Created transactions are now checked against the current network's maximum 
transaction size limit, to avoid creating transactions which are too long for 
consensus-validating nodes to accept.

* An out-of-bounds panic seen during address discovery of imported xpub accounts
 was corrected.

* A data race during the subscribing of transaction notifications involving 
wallet addresses was fixed.

* The `getaddressesbyaccount` JSON-RPC method now returns results for the 
imported account.

* The database implementation used by dcrwallet (bbolt) was fixed and updated 
to remove invalid usage of Go's unsafe programming features.

* The peer-to-peer implementation now allows the same block to be requested 
concurrently by the same peer.  This fixes some occasional errors which stopped 
the SPV syncer under normal wallet usage.

* The autobuyer will no longer mix the change account when the wallet is not 
up to date with other peers on the network.  This avoids submitting mix requests
 involving outputs which may have already been spent.

* A memory leak of wallet address private keys when operating a wallet that 
remained always unlocked was plugged.

* The stakebase script found in vote transactions is now included when creating 
the unsigned vote, rather than during signing.  This fix ensures that the 
correct stakebase script for the active network is always used, instead of 
filling in the script for a different network.

* UTXO selection is now aware of output maturity and will not include immature 
outputs.

## Changelog

All commits since the last release may be viewed on GitHub 
[here](https://github.com/decred/dcrwallet/compare/v1.5.1...v1.6.0).

### Code Contributors

In alphabetical order:

- Alex Yocom-Piatt
- Brian Stafford
- Dave Collins
- David Hill
- Donald Adu-Poku
- Jamie Holdstock
- Jonathan Chappelow
- Josh Rickmar
- Julian Yap
- Kifen
- Marco Peereboom
- Matheus Degiovani
- Matt Hawkins
- Mike Belopuhov
- peterzen
- Victor Oliveira
- Wisdom Arerosuoghene


# decrediton v1.6.0

This release includes two major functionality improvements for staking 
and privacy.  The past privacy improvements that were available for dcrwallet 
are now implemented in decrediton for ticket purchasing and account mixing. 
Accountless VSP ticket purchasing has been added as well that should greatly 
simplify the staking process and increase privacy.

There have been numerous graphical improvements and bug fixes that will 
hopefully lead to a smoother UX and reduce support questions and intervention.

## New Features

### Privacy

We have added a new menu item that covers Privacy and Security tools. 
Users should go there to 'enable' privacy on their wallets.  This enabling 
process is not done automatically yet, mostly due to required user 
intervention with private passphrase entry to create needed mixed and unmixed 
accounts.  Hopefully in the future we will add this step to the wallet 
launcher.

Once enabled, the privacy page will transform into an account mixer form that 
allows users to mix funds from the unmixed account into the mixed account.  To 
follow what dcrwallet is doing, there is a log window below.  In the future, 
we may add better messaging that will allow updates to the mixing process 
instead of just showing raw logs.

Once privacy is enabled, spending to external addresses is only allowed from the 
mixed account.  This is to ensure privacy is not broken by spending from any 
unmixed account.  Additionally, it is not allowed to manually generate 
addresses in the mixed account, since only funds that have been properly mixed 
should be allowed to end up there.

There is a checkbox that allows users to forgo the external spending 
restriction.  There is a dominant warning and users must confirm the risks 
they are imposing by spending from unmixed accounts.

### Accountless VSP Staking with Optional Mixing

The new accountless VSP ticket purchasing process has been added. 
Now there is no need to create an account at VSP's website, add its API key in 
Decrediton and backup redeem scripts.  Users may now simply go 
to the tickets page, select the VSP they'd like to use and the number of 
tickets to purchase.

This new way of staking is more private even without mixing enabled since it 
eliminates address reuse and the use of email. It is recommended that users 
migrate their tickets to this new system sooner. Read more 
[here](https://blog.decred.org/2020/06/02/A-More-Private-Way-to-Stake/).

If privacy is enabled, the process of purchasing a 
ticket requires there to be a successful mix to occur.  Mix sessions happen 
every 20 minutes and participating in a single session is usually (though not 
always) sufficient.  If successful, the mixed split ticket funding 
transaction, the ticket, and the ticket's fee, should all be seen on the 
overview.  But as you may notice, the ticket's fee is not yet broadcast onto 
the network by the VSP until the ticket has been confirmed by 6 blocks.  If 
any tickets have missing or errored fees, the user will be notified if they 
try to close Decrediton.

## Other Updates

* Menu was reorganized and optimized to accomodate added tabs and tools.  Since 
we are quickly adding functionality we need to make sure the left-hand sidebar 
menu is as efficient as possible without becoming bloated with items. 
 Hopefully the current layout allows for more growth for tools and 
functionality.

* Peer count is now shown on the side bar.  This should alleviate issues where 
people don't know why their transactions aren't getting mined.

* An SPV indicator has been added to the sidebar.  Previously there 
was no way of understanding what mode the wallet was running in without 
looking at Settings page.

* Unmined transactions are now able to be abandoned under transaction details. 
 This should fix issues that people previously had unminable transactions 
"stuck" in their wallet.  If the network doesn't know about the transaction, 
then they should be able to be abandoned and the funds unreserved.

* A full refactor of components into functional components is now mostly 
complete.  This should now allow for more agile development moving forward.

## Code Contributors

In alphabetical order:

- Alex Yocom-Piatt
- Amir Massarwa
- artikozel
- bgptr
- David Hill
- degeri
- Fernando Guisso
- Guilherme Marques
- Jamie Holdstock
- JoeGruffins
- Jonathan Zeppettini
- karamble
- Matheus Degiovani
- Nicola Larosa
- Piotr Delikat
- Thiago F. Figueiredo
- unimere
- Victor Oliveira
- Victor Guedes
- Zubair Zia

Welcome our new additions to the decrediton team: Amir Massarwa, bgptr, 
Fernando Guisso, JoeGruffins, Guilherme Marques, Victor Guedes.


# dcrlnd v0.3.0

_Please note that while Bitcoin's Lightning Network has been in production for 
a few years, Decred's version still hasn't seen extensive use in mainnet. Users 
should be mindful of the total amount of funds committed to dcrlnd wallets and 
channels._

This is a major dcrlnd release including significant amount of changes.

This release brings dcrlnd in line with the upstream lnd 
[release v0.11.1](https://github.com/lightningnetwork/lnd/releases/tag/v0.11.1-beta)
 and also includes ports for versions 
[v0.11.0](https://github.com/lightningnetwork/lnd/releases/tag/v0.11.0-beta), 
[v0.10.0](https://github.com/lightningnetwork/lnd/releases/tag/v0.10.0-beta) and
 [v0.9.0](https://github.com/lightningnetwork/lnd/releases/tag/v0.9.0-beta).

## Vulnerability Fixes

This release includes fixes for 
[CVE-2020-26896](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-October/002857.html)
 and [CVE-2020-895](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-October/002858.html)
 made in the upstream lnd project. Fixes for these were released in upstream 
versions v0.11.0 and v0.10.0 respectively and the underlying issues were fully 
disclosed in Oct 20, 2020.

Additional context for the vulnerabilities and its impact in LN implementations,
 written by the original discoverer can be found 
[here](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-October/002859.html)
 and [here](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-October/002855.html)

## Database Migrations

This release contains database migrations for the new TLV encoding of invoices, 
payment address indexing and close summary information. Old versions of dcrlnd 
cannot use the new database version once these migrations are applied.

## Changelog

The major Decred-specific feature introduced in this release is the ability to 
run a dcrlnd instance connected to a dcrwallet running in SPV mode. This is 
useful mostly for Decrediton users that will now have the option to run dcrlnd 
even when their wallet is using the SPV configuration.

### Node Syncing Config

CLI users now have two options for the `--node` argument:

- `--node=dcrd` instructs dcrlnd to connect to a dcrd instance for on-chain 
  operations.
- `--node=dcrw` instructs dcrlnd to use the underlying dcrwallet instance for 
  on-chain operations.

When using `--node=dcrd`, the `--dcrd.`-namespaced options should be used to 
configure the connection to the underlying dcrd node.

When using `--node=dcrw`, either the `--dcrd.`-namespaced options should be 
used, in order to use an _embedded_ dcrwallet instance (that is, dcrwallet 
runs automatically inside dcrlnd) **or** the `--dcrw.`-namespaced options 
should be used to configure a __remote__ dcrwallet instance.

Note that SPV mode is only supported on remote dcrwallet instances.

For hub nodes (that is, nodes that are online most of the time and offer 
the ability to receive open channel requests) the recommended config setting 
is to use embedded wallets with a dcrd instance.

### Wumbo Channel Support

This release adapts the [Wumbo](https://github.com/lightningnetwork/lnd/pull/4429)
feature for the realities of Decred. Wumbo channel support can be enabled by 
running dcrlnd with `--protocol.wumbo-channels` and has a global maximum channel
 size of 500 DCR.

### Relevant Upstream Changes

The following is a non-exhaustive list of the relevant upstream changes that 
were ported to dcrlnd. These include changes from the upstream 
[v0.9](https://github.com/decred/dcrlnd/pull/74), 
[v0.10](https://github.com/decred/dcrlnd/pull/99) and 
[v0.11](https://github.com/decred/dcrlnd/pull/103) lines. Please refer to the 
respective upstream releases for additional information.

- Multi Path Payment (MPP) support so that a single payment can be split 
  among multiple channels.
- Track payments with a new Payment Address field.
- Additional TLV data sent in payments, which allows creating new use cases 
  to deliver payload data via LN payments.
- Keysend payment experiment which allows spontaneous payments without the 
  need for a precreated invoice.
- Upfront shutdown script support to enforce channel closure to pay to 
  pre-configured addresses.
- HTLC Interception API to allow creation of custom payment forwarding engines.
- Additional data in Channel Close Summaries.
- Add ability to limit max remote pending HTLC amount during channel opening.
- Anchor outputs experimental feature.
- External channel funding experimental feature.
- Healthchecks to ensure adequate operating conditions of the node
- Several bug fixes throughout the app.

## Porting Effort

A total of 450 upstream PRs were considered for inclusion. The list of of PRs 
can be found in the acompanying [upstream-prs.csv](/docs/upstream-prs.csv) doc.

## Decred Contributors (Alphabetical Order)

- Fernando Guisso
- Matheus Degiovani
- Ole Andre Birkedal

## Acknowledgement

The majority of the work included in this release is from features and bugfixes 
performed by the contributors to the upstream 
[lnd](https://github.com/lightningnetwork/lnd) project that were ported to 
Decred.

We wish to sincerely thank them for providing such a high quality project 
and hope we can continue to contribute in building a large scale and cross-coin 
LN ecosystem.

# dcrdex v0.1.4

This patch release makes a number of small UI improvments, including showing the
user's account ID on the settings page, focusing password field, and colorizing
various buy/sell elements, and fixes several bugs.  The main bug fixes deal 
with wallet settings changes, deposit address revalidation, Decred withdraws 
working for the full balance, and historical order and match display.

Please read the [initial release (v0.1.0) notes](https://github.com/decred/dcrdex/releases/tag/release-v0.1.0) for important information and instructions.

## Client (dexc)

### Features and Improvements

- The account ID for each configured DEX server is now displayed on the settings 
page.  When not logged in, there is a placeholder that says to login to display 
the account ID. ([1a5c070](https://github.com/decred/dcrdex/commit/1a5c070196ab7899214b524a9c681168fbdcfd75))
- Focus password field in order dialogs. ([5eb9fb2](https://github.com/decred/dcrdex/commit/5eb9fb2de51722158eaf1d2122c11f30154bd9b3))
- Colorize the "Side" column in the orders table. ([83b07cd](https://github.com/decred/dcrdex/commit/83b07cd08a8a7ecced012335092c0f196d7fcfb0))
- The registration fee address is no longer logged if there is a funding error 
since there is nothing a user can do with the address other than shoot 
themselves in the foot and send to it manually.  Registration fee payment should
 only be done via the app. ([dc67cdb](https://github.com/decred/dcrdex/commit/dc67cdbb09fe6e296164da0b916ab8a1744912f6))
- Wallet balances are updated on all wallet settings changes. ([8ff4d94](https://github.com/decred/dcrdex/commit/8ff4d943d69182b9866faf6637e9e3c17e97db69))
- Wallet sync status is more consistently checked on wallet (re)connect events, 
and continually check sync status on RPC errors as it is common for node/wallet
 startup to initially error and then start reporting status (e.g. bitcoind's 
 "verifying blocks..." error while starting up). ([1c9ca02](https://github.com/decred/dcrdex/commit/1c9ca02db974cbd76dceac5b29a825b5cc805c84), [53194c6](https://github.com/decred/dcrdex/commit/53194c615ee3179c2c6ec08278a41bbd9b234634))

### Fixes

- Fix changing wallet settings possibly interrupting active swaps. ([f0a304f](https://github.com/decred/dcrdex/commit/f0a304f7ea74af3ce75f3edc1cbb3f4f524f1c84))
- Fix a case where a wallet can become unlockable without restarting dexc if 
dexc were started with both active orders and an unlocked wallet. ([f8c47a1](https://github.com/decred/dcrdex/commit/f8c47a163387b8c63201f2f9ad1053a205e6203f))
- Fix duplicate notify_fee requests that resulted from multiple fee coin 
waiters being created for the same coin. ([ee1bd84](https://github.com/decred/dcrdex/commit/ee1bd84c8ef6136fcbbcf764782b610d20c3540c))
- Fix retrieving the full list of historical orders ([2846814](https://github.com/decred/dcrdex/commit/284681488b5812157dd8624151efc576764eb824))
- Fix incorrect year displayed for a match's date. ([a347b0f](https://github.com/decred/dcrdex/commit/a347b0f34d0fd143b566b59588cda4f86f1b218b))
- Wallet deposit addresses are validated and more often refreshed whenever the 
wallet is connected ([c3990c7](https://github.com/decred/dcrdex/commit/c3990c765f7a7de2017da08c29fb9fae8853a522), [6a66a1c](https://github.com/decred/dcrdex/commit/6a66a1cb7701ed6d6e7187231a46ad1f2a74a782))
- Correctly handle chain sync status when in initial block download state, but 
blocks are up-to-date with headers. This is only possible in practice with 
simnet. ([3523de1](https://github.com/decred/dcrdex/commit/3523de11b270fed9162c0b2bd8aee2333fe2e8f6))
- Fix DCR withdraws in various cases. ([d0ba1e5](https://github.com/decred/dcrdex/commit/d0ba1e5dbcdc063c8fb4abf95725c67174868291))
- Allow dexc to shutdown without hanging if a wallet was unexpectedly shutdown 
first. ([7321c36](https://github.com/decred/dcrdex/commit/7321c364297b8f5c0dd85cf798902b169bd3eebf))
- When loading active matches on login, correctly skip adding cancel order 
matches to the trades map. ([61697bb](https://github.com/decred/dcrdex/commit/61697bbc4364466d9eb55763aed8e7fb849e01e0))
- Prevent login while already logged in from re-creating the entries in the 
trades map. ([b6f81ad](https://github.com/decred/dcrdex/commit/b6f81adcc9a05f4c604420b3b138f1286b25c9c7))
- Resolve a data race on wallet reconfigure for DCR. ([bca1325](https://github.com/decred/dcrdex/commit/bca1325ab1ccdd21b3447571693a8212e5874e97))
- Avoid a possible deadlock on wallet reconfigure. ([4bed3e2](https://github.com/decred/dcrdex/commit/4bed3e2f55f97cac45ca30cf7ad4faac94d20604))

## Developer

- Simnet harnesses are quicker to start, being based on archives, and more well 
funded. ([0de8945](https://github.com/decred/dcrdex/commit/0de89456c129bc39a200e816fb660f216a7d41e2))
- Update simnet trade tests for current wallet unlocking system and more well 
funded harnesses. ([e198b1f](https://github.com/decred/dcrdex/commit/e198b1f095be8cad51c8e49604c873ed2ac4f02d))

## Server (dcrdex)

Create a fee rate scaling administrative endpoint. ([7a3f183](https://github.com/decred/dcrdex/commit/7a3f18313a34a5945c064a06a1b85bfdc07b0dd4))
The endpoint is `api/asset/{sym}/setfeescale/{scale}`, using a GET request 
instead of POST for convenience.

## Code Summary

27 commits, 52 files changed, 1582 insertions(+), and 890 deletions(-)

https://github.com/decred/dcrdex/compare/v0.1.3...v0.1.4

6 contributors

- Amir Massarwa (@amassarwi)
- Brian Stafford (@buck54321)
- David Hill (@dajohi)
- Jonathan Chappelow (@chappjc)
- Kevin Wilde (@kevinstl)
- @peterzen


# dcrdex v0.1.3

This patch release includes a workaround for a bug in the Safari browser, an 
important fix to a possible deadlock (client hang), and a minor fix to the 
client's validation of the server's order matching.

NOTE: If you use the dexcctl program (an optional command line tool), you will 
need to move any dexcctl.conf file from the ".dexc" folder to a new ".dexcctl" 
folder.

Please read the [initial release (v0.1.0) notes](https://github.com/decred/dcrdex/releases/tag/release-v0.1.0) for important information and instructions.

## Client (dexc)

### Fixes

- Eliminate a possible deadlock (hang) introduced in v0.1.2. ([65c9830](https://github.com/decred/dcrdex/commit/65c98309370779e747d676b2c29020610645284d))
- Fix the client's validation of the server's deterministic epoch matching 
result.  This avoids an error message in the logs, but the bug was otherwise 
not a problem. ([10b4689](https://github.com/decred/dcrdex/commit/10b4689ae9a1118f94747951fd3ac444e490faab))

### Other

The location of the dexcctl.conf file is now in ~/.dexcctl instead of ~/.dexc 
(or the corresponding "appdata" folders on Windows and macOS)  ([16a0fb0](https://github.com/decred/dcrdex/commit/16a0fb003e2eca51fd2c29b938b0ec9bf681f7e5))

## Server (dcrdex)

There are no server changes.

## Code Summary

5 commits, 17 files changed, 188 insertions(+), and 174 deletions(-)

https://github.com/decred/dcrdex/compare/v0.1.2...v0.1.3

2 contributors

- Brian Stafford (@buck54321)
- Jonathan Chappelow (@chappjc)


# dcrdex v0.1.2

This patch release improves handling of slow contract audits, fixes a bug on 
32-bit systems, automatically unlocks wallets to avoid swap failures if the 
wallet unexpectedly became locked, and improves the display of canceled orders. 
 There are also a few usability improvements for developers.

Please read the [initial release (v0.1.0) notes](https://github.com/decred/dcrdex/releases/tag/release-v0.1.0) for important information and instructions.

## Client (dexc)

### Features and Improvements

#### User-facing

- When already logged in, automatically attempt to unlock wallets as needed for
trades.  This helps prevent users from breaking their swaps by accidentally 
locking their wallets. ([de40913](https://github.com/decred/dcrdex/commit/de409134c37270145dc7094e89d6ef9d8e2d1f74))
- Display cancel order matches differently from trade matches. ([b013581](https://github.com/decred/dcrdex/commit/b01358159eeb7cbe5024f58f035306e98bb0a2f8))

#### Developer

- Create a `Ready` method so consumer packages know when the client core is 
done starting up.  ([c3d9e80](https://github.com/decred/dcrdex/commit/c3d9e80602e9cad8cc7ebc80e2d7e96a2257d3ab))
- Increase notification channel capacity to prevent dropped notifications when 
there are many simultaneous events. ([2de62a3](https://github.com/decred/dcrdex/commit/2de62a378d8b964c6ff2a485ca907b0b1c2b7ac4))
- Remove the obsolete (and incomplete) terminal UI.  ([75ff8d0](https://github.com/decred/dcrdex/commit/75ff8d09f6f5f898dfd23ebbacbb7a3f1d2e473f))

### Fixes

- Workaround for 64-bit atomic variable access on 32-bit platforms. ([3abaf43](https://github.com/decred/dcrdex/commit/3abaf434a3da3603916969f7af4b0c487b76b149))
- Prevent contract auditing from blocking incoming messages.  Continue to 
search for counterparty contracts until it succeeds or the match is revoked, 
and log a warning of the audit is taking a long time. ([23f2f36](https://github.com/decred/dcrdex/commit/23f2f362486141419d4a321674229f3716fd4faf))

## Server (dcrdex)

There are no server changes.

## Code Summary

9 commits, 44 files changed, 621 insertions(+), and 2,652 deletions(-)

https://github.com/decred/dcrdex/compare/v0.1.1...v0.1.2

3 contributors

- Brian Stafford (@buck54321)
- David Hill (@dajohi)
- Jonathan Chappelow (@chappjc)


# dcrdex v0.1.1

This patch release addresses two important match recovery bugs, and a number of
minor bugs. This release also includes several improvements to the client's user
 interface. New client features include Tor proxy support and new deposit 
 address generation.

Please read the [initial release (v0.1.0) notes](https://github.com/decred/dcrdex/releases/tag/release-v0.1.0) for important information and instructions.

## Client (dexc)

### Features and Improvements 

- Add the mainnet ["client quick start" guide](https://github.com/decred/dcrdex#client-quick-start-installation). ([a383d5e](https://github.com/decred/dcrdex/commit/a383d5e76d2de90969f4eaf372084d290a051032))
- Tor support for connections with DEX servers. ([824f1c0](https://github.com/decred/dcrdex/commit/824f1c0da0b17afcab271c60665be6f8da3d6025)) **WARNING**: This should be used with caution since Tor is slow and unreliable.
- On dexc start-up, display a link (URL) to the browser page, and if there are 
active orders, warn the user. ([a01e403](https://github.com/decred/dcrdex/commit/a01e403d09c491765d71ac34fc2d60b7898c3596))
- Add the ability to generate new deposit addresses. ([860af3e](https://github.com/decred/dcrdex/commit/860af3e19b49db9fb6b68016e894edd71361db3d))
- Various browser UI improvements, including order dialog wording and button 
formatting. ([dbf9d2c](https://github.com/decred/dcrdex/commit/dbf9d2c1f7c4644530b8a91f407818d4f435aa7b))
- Dialogs now have a close/cancel button. ([6716b58](https://github.com/decred/dcrdex/commit/6716b58c87933e71661f922d8cd2f479e6851a0d))
- Taker redemption transactions are more readily batched, potentially requiring 
fewer transactions for a taker order that matches with multiple maker orders. ([3ea75a9](https://github.com/decred/dcrdex/commit/3ea75a91d8e6935ad2cde128190042bde24f1e1d))
- When any node (e.g. bitcoind and dcrd) is still synchronizing with the 
network, new orders cannot be placed. ([2cac73a](https://github.com/decred/dcrdex/commit/2cac73a6655550d3cdd02ca2591844988e8126e7))

### Fixes

- Match recover is more robust, with fixes to revoked match handling on 
reconnect or restart. ([9790fb1](https://github.com/decred/dcrdex/commit/9790fb1cfb6e7ed7ffadc4624979ca57341d2ca0))
- Resolve a potential deadlock during match status resolution, ([c09017d](https://github.com/decred/dcrdex/commit/c09017d8170602bfae4fc2e34edd5ccfee34127e))
- Explicitly set js Content-Type in webserver to workaround misconfigured 
operating systems, such as Windows with misconfigured CLASSES_ROOT registry entries. ([f632893](https://github.com/decred/dcrdex/commit/f6328937f815f210daf4d910cb4722205ddf6e79))
- Delete obsolete notifications on frontend. ([8a69e99](https://github.com/decred/dcrdex/commit/8a69e991b518170eacc23d99eb8e1629ce7517d4))
- Avoid harmless but confusing warnings about returning zero coins when 
resumed trades are later completed. ([a01e403](https://github.com/decred/dcrdex/commit/a01e403d09c491765d71ac34fc2d60b7898c3596))
- Avoid redundant swap negotiation invocations on restart with unknown matches 
reported from a server. ([c0adb26](https://github.com/decred/dcrdex/commit/c0adb2659fe4ab341fc75b995c261b2cee029675))
- Orphaned cancel orders that could be created in certain circumstances are now 
retired during status resolution of the linked trade. ([867ba89](https://github.com/decred/dcrdex/commit/867ba894b6da4f6cda16ba4c371eb2436cc4d977))

## Server (dcrdex)

- Fix book purge heap orientation. ([eb6ccd4](https://github.com/decred/dcrdex/commit/eb6ccd464af474c4d7b506ee465a66e1f69f534f))
- Avoid orphaned epoch status orders when shutting down via SIGINT *without* a 
preceding suspend command. ([d463439](https://github.com/decred/dcrdex/commit/d4634395495f25b92cdc935fb7a72f311d23d118))
- When any node (e.g. bitcoind and dcrd) is still synchronizing with the 
network, relevant markets will not accept new orders. ([2cac73a](https://github.com/decred/dcrdex/commit/2cac73a6655550d3cdd02ca2591844988e8126e7))

## Code Summary

17 commits, 66 files changed, 2216 insertions(+), 566 deletions(-)

https://github.com/decred/dcrdex/compare/release-v0.1.0...release-v0.1.1

3 contributors

- Brian Stafford (@buck54321)
- David Hill (@dajohi)
- Jonathan Chappelow (@chappjc)


# dcrdex v0.1.0 (beta)

## Important Notices

- Ensure your nodes (and wallets) are fully **synchronized with the blockchain 
network before placing orders**. The software will verify this for you in the 
next patch release.
- **Never shutdown your wallets with dexc running**. When shutting down, always 
stop dexc before stopping your wallets.
- If you have to restart dexc with active orders or swaps, you must 
**immediately login again with your app password when dexc starts up**. The 
next patch release wil inform you on startup if this is required.
- There is a ~10 minute "inaction timeout" when it becomes your client's turn 
to broadcast a transaction, so be sure not to stop dexc or lose connectivity 
for longer than that or you risk having your active orders and swaps/matches 
revoked. If you do have to restart dexc, remember to login as soon as you start 
it up again.
- Only one dexc process should be running for a given user account at any time. 
For example, if you have identical dexc configurations on two computers and you 
run and login on both, neither dexc instance will be adequately connected to 
successfully negotiate swaps. Also note that order history is not synchronized 
between different installations at this time.
- Your DEX server accounts exist inside the dexc.db file, the location of which 
depends on operating system, but is typically in ~/.dexc/mainnet/dexc.db or 
%HOMEPATH%\Local\Dexc\mainnet\dexc.db.  Do not delete this account or you will 
need to register again at whatever server was configured in it.

## Overview

This release of DCRDEX includes a client program called dexc and a server 
called dcrdex. Users run their own wallets (e.g. dcrwallet or bitcoind), which
dexc works with to perform trades via atomic swaps. dcrdex facilitates price 
discovery by maintaining an order book for one or more markets, and coordinates 
atomic swaps directly between pairs of traders that are matched according to 
their orders. The server is generally run on a remote system, but anyone may 
operate a dcrdex server.

This release supports Decred, Bitcoin, and Litecoin.

### Client (dexc)

- Provides a browser-based interface, which is self-hosted by the dexc 
program, for configuring wallets, displaying market data such as order books, 
and placing and monitoring orders.
- Communicates with any user-specified dcrdex server.
- Funds orders and executes atomic swaps by controlling the external wallets 
(dcrwallet, etc.).

### Server (dcrdex)

- Accepts orders from clients who prove ownership of on-chain coins to fund 
the order.
- Books and matches orders with an epoch-based matching algorithm.
- Relays swap data between matched parties, allowing the clients to perform 
the transactions themselves directly on the assets' blockchains.
- Has a one-time nominal (e.g. 1 DCR) registration fee, which acts as an 
anti-spam measure and to incentivize completing swaps.
- Enforces the code of community conduct by suspending accounts that 
repeatedly violate the rules.

## Features

### Markets and Orders

The server maintains a familiar market of buy and sell orders, each with a 
quantity and a rate. A market is defined by a pair of assets, where one asset 
is referred to as the **base asset**. For example, in the "DCR-BTC" market, 
DCR is the base asset and BTC is known as the **quote asset**. A market is 
also specified by a **lot size**, which is a quantity of the base asset. Order 
quantity must be a multiple of lot size, with the exception of market buy 
orders that are necessarily specified in units of the quote asset that is 
offered in the trade. The intent of a client to execute an atomic swap of one 
asset for another is communicated by submitting orders to a specific market 
on a dcrdex server.

The two types of trade orders are market orders, which have a quantity but no 
rate, and limit orders, which also specify a rate. Limit orders also have a 
**time-in-force** that specifies if the order should be allowed to become 
booked or if it should only be allowed to match with other orders when it is 
initially processed. The time-in-force options are referred to as "standing" 
or "immediate", where standing indicates the order is allowed to become booked 
while immediate restricts that order to being a taker order by only allowing a 
match when it is initially processed.

The following image is an example order submission dialog from a testnet 
DCR-BTC market with a 40 DCR lot size that demonstrates limit order buying 2 
lots (80 DCR) at a rate of 0.001207 BTC/DCR using a standing time-in-force to 
allow the order to become booked if it is not filled:

![submit order dialog](https://user-images.githubusercontent.com/9373513/97030709-c8fd9500-1524-11eb-8bd0-3eb4cf95e8c8.png)

Checking the "Match for one epoch only" box above specifies that the limit 
order's time-in-force should be immediate, while unchecking it allows the order 
to be booked if it does not match with another order at first. The concept of 
epochs is described in the [Epoch](#epochs) section.

### Order Funding

Since orders must be funded by coins from the user's wallets, placing an order 
"locks" an amount in the relevant wallet. For example, a buy order on the 
DCR-BTC market marks a certain quantity of BTC as locked with the user's wallet.
 (This involves no transactions or movement of funds.) This amount will be 
 shown in the "locked" row of the Balances table.

It is important to note that the amount that is locked by the order may be 
**larger than the order quantity** since the "locked" amount is dependent on 
the size of the UTXO (for UTXO-based assets like Bitcoin and Decred) that is 
reserved for use as an input to the swap transaction, where the amount that 
does not enter the contract goes in a change address. This is no different 
from when you make a regular transaction, however because the input UTXOs are 
locked in advance of broadcasting the actual transaction that spends them, you 
will see the amount locked in the wallet until the swap actually takes place.

Depending on the asset, there may be a wallet setting on the Wallets page to 
pre-size funding UTXOs to avoid this over-locking, but (1) it involves an extra 
transaction that pays to yourself before placing the order, which has on-chain 
transaction fees that may be undesirable on chains like BTC, and (2) it is only 
applied for limit orders with standing time-in-force since the the UTXOs are 
only locked until the swap transaction is broadcasted, which is relatively 
brief for taker-only orders that are never booked.

### Epochs

An important concept with DCRDEX is that newly submitted orders are processed 
in short windows of time called **epochs**, the length of which is part of the 
server's market configuration, but is typically on the order of 10 seconds. 
When a valid order is received by the server, it enters into the pool of epoch 
orders before it is matched and/or booked. The motivation for this approach is 
described in detail in the DCRDEX specification. The Your Orders table will 
show the status of such orders as "epoch" until they are matched at the end of 
the epoch, as described in the next section.

Order cancellation requests are also processed in the epoch with trade 
(market/limit) orders since a cancellation is actually a type of order. 
However, from the user's perspective, cancelling an order is simply a matter 
clicking the cancel icon for one of their booked orders.

### Matching

When the end of an epoch is reached, the orders it includes are then matched 
with the orders that are already on the book. A key concept of DCRDEX order 
matching is a deterministic algorithm for shuffling the epoch orders so that 
it is difficult for a user to game the system. To perform the shuffling of the 
closed epoch prior to matching, clients with orders in the epoch must provide 
to the server a special value for each of their orders called a **preimage**, 
which must correspond to another value that was provided when the order was 
initially submitted called the **commitment**. This is done automatically by 
dexc, requiring no action from the user.

If an order fails to match with another order, it will become either 
**booked** or **executed** with no part of the order filled. The Your Orders 
table displays the current status and remaining quantity of each of a user's 
orders. If an order does match with another trade order, the order status will 
become **settling**, and atomic swap negotiation begins.

### Settlement

When maker orders (on the book) are matched with taker orders from an epoch, 
the atomic swap sequence begins. No action is required from either user during 
the process.

In the current atomic swap protocol, the **maker initiates** by broadcasting a 
transaction with a swap contract on the relevant asset network, and informing 
the server of the transaction and the full contract. The server audits the 
contract, and if it is successfully validated, the information is relayed to 
the taker, who independently audits the contract to ensure it meets their 
expectations. The transaction containing the maker's swap contract must then 
be mined and reach the **swap confirmation requirement**, which is also a 
market setting. For example, Bitcoin might require 3 confirmations while other 
chains like Litecoin might be considerably more. When the required number of 
confirmations is reached, the **taker participates** by broadcasting a 
transaction with their swap contract and informing the server. Again, the 
server and the counterparty audit the contract and wait for that asset's swap 
confirmation requirement. When the required confirmations are reached, the 
**maker redeems** the taker's contract and informs the server of the redemption 
transaction. This is the end of the process for the maker, as the redemption 
spends the taker's contract, paying to an address controlled by the maker. 
The server relays the maker's redeem data to the taker, and the 
**taker redeems** immediately, ending the swap.

The Order Details page shows each match associated with an order. For example, 
a match where the user was the taker is shown below with links to block 
explorers for each of the transactions described above. The maker will have 
their redemption listed, but not the taker's.

![match details](https://user-images.githubusercontent.com/9373513/97028559-eda43d80-1521-11eb-9ab6-2e0b21df584d.png)

Orders may be partially filled in increments of the lot size. Hence a single 
order may have more than one match (and thus swap) associated with it, each of 
which will be shown on the Order Details page.

Wallet balances will change during swap negotiation. When the client broadcasts 
their swap contract, the amount locked in that contract will go into the 
"locked" row for the asset that funded the order. When the counterparty 
redeems their contract, that amount will be reduced by the contract amount, 
and the user will redeem the counterparty contract, thus adding to the balance 
of the other asset. This is the essence of the atomic swap. Note that until 
the redemption transactions are confirmed, the redeemed amount may remain in 
the wallet's "immature" balance category, but this depends on the asset.

### Revoked Matches

While the atomic swap process requires no party to trust the other, a swap may 
be forced into an alternate path ending in one or both users refunding 
themselves by spending their own contract after the lock time expires. This 
happens when one of the parties fails to act in the expected time frame, an 
**inaction timeout**. When an inaction timeout occurs the following happens:

- The match is revoked, and both parties are notified.
- The at-fault user has their order revoked (if it was partially filled and 
still booked) and is notified.
- The at-fault user has their score adjusted according to type of match 
failure. See below for descriptions of each type and the associated user score 
adjustments.

The general categories of match failures are:

- `NoMakerSwap`: A match is made, but the maker does not initiate the swap. 
No transactions are created in this case.
- `NoTakerSwap`: The maker (initiator) broadcasts their swap contract 
transaction and informs the server, but the taker (participant) fails to 
broadcast their swap contract and inform the server. The maker will 
automatically refund their contract when it expires after 20 hrs.
- `NoMakerRedeem`: The taker broadcasts their swap and informs the server, but 
the maker does not redeem it. The taker will refund when their contract expires 
after 8 hrs. Note that the taker's client begins watching for an unannounced 
redeem of their contract by the maker, which reveals the secret and permits 
the taker to redeem as well, completing the swap although in a potentially 
extended time frame.
- `NoTakerRedeem`: The maker redeems the taker's contract and informs the 
server, but the taker fails to redeem the maker's contract even though they 
can do so at any time. This case is not disruptive to the counterparty, and is 
only detrimental to the takers, so it is of minimal concern.

NOTE: The *order* remaining amounts are still reduced at match time although 
they did not settle that portion of the order.

### User Scoring

Users have an incentive to respond with their preimage for their submitted 
orders and to complete swaps as the match negotiation protocol specifies, and 
if they repeatedly fail to act as required, their account may be suspended. 
This may require either communicating an excusable reason for the issue to the 
server operator, or registering a new account. However, a reasonable scoring 
system is established to balance the need to deter intentional disruptions 
with the reality of unreliable consumer networks and other such technical 
issues.

In this release, there are two primary inaction violations that adjust a 
users score: (1) failure to respond with a preimage for an order when the 
epoch for that order is closed (preimage miss), and (2) swap negotiation 
resulting in match revocation as described in the [previous section](#revoked_matches).

The score threshold at which an account becomes suspended (ban score) is an 
operator set variable, but the default is 20.

The adjustment to the at-fault user's score depends on the match failure:

| Match Outcome   | Points | Notes                                              |
|-----------------|-------:|----------------------------------------------------|
| `NoMakerSwap`   |      4 | book spoof, taker needs new order, no locked funds |
| `NoTakerSwap`   |     11 | maker has contract stuck for 20 hrs                |
| `NoMakerRedeem` |      7 | taker has contract stuck for 8 hrs                 |
| `NoTakerRedeem` |      1 | counterparty not inconvenienced, only self         |
| `Success`       |     -1 | offsets violations                                 |

A preimage miss adds 2 points to the users score.

The above scoring system should be considered tentative while it is evaluated 
in the wild.

### Order Size Limits

This release uses an experimental system to set the maximum order quantity 
based on their swap history. It is likely to change, but it is described in [PR #750](https://github.com/decred/dcrdex/pull/750).


## Code Summary

This release consists of 473 pull requests comprising 506 commits from 12 contributors.

Contributors (alphabetical order):

- Brian Stafford (@buck54321)
- David Hill (@dajohi)
- @degeri
- Donald Adu-Poku (@dnldd)
- Fernando Abolafio (@fernandoabolafio)
- Joe Gruffins (@JoeGruffins)
- Jonathan Chappelow (@chappjc)
- Kevin Wilde (@kevinstl)
- @song50119
- Victor Oliveira (@vctt94)
- Wisdom Arerosuoghene (@itswisdomagain)
- @zeoio

(there is no previous release to which a diff can be made)
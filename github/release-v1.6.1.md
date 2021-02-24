## Install

To install Decrediton desktop wallet, download, uncompress, and run
[Decrediton Linux](https://github.com/decred/decred-binaries/releases/download/v1.6.1/decrediton-v1.6.1.AppImage)
or
[Decrediton macOS](https://github.com/decred/decred-binaries/releases/download/v1.6.1/decrediton-v1.6.1.dmg)
or
[Decrediton Windows](https://github.com/decred/decred-binaries/releases/download/v1.6.1/decrediton-v1.6.1.exe).

To install the command-line tools, please see
[dcrinstall](https://github.com/decred/decred-release/tree/master/cmd/dcrinstall).

See decred-v1.6.1-manifest.txt and the other manifest files for SHA-256 hashes
and the associated .asc signature files to confirm those hashes.

See [README.md](./README.md#verifying-binaries) for more info on verifying the
files.


## Contents

* [dcrd](#dcrd-v161)
* [dcrwallet](#dcrwallet-v161)
* [Decrediton](#decrediton-v161)
* [dcrdex](#dcrdex-v015)


# dcrd v1.6.1

This is a patch release of dcrd which includes the following changes:

- Correct a hard to hit issue where connections might not be reestablished after
  a network outage under some rare circumstances
- Allow stakeholders to make use of the staking system to force proof-of-work
  miners to upgrade to the latest version so voting on the new consensus changes
  can commence

## Changelog

This patch release consists of 3 commits from 1 contributor which total to 3
files changed, 30 additional lines of code, and 9 deleted lines of code.

All commits since the last release may be viewed on GitHub
[here](https://github.com/decred/dcrd/compare/release-v1.6.0...release-v1.6.1).

### Protocol and Network

- server: Notify block mgr later and track ntfn
  ([decred/dcrd#2588](https://github.com/decred/dcrd/pull/2588))
- server: Force PoW upgrade to v8
  ([decred/dcrd#2597](https://github.com/decred/dcrd/pull/2597))

### Misc

- release: Bump for 1.6.1
  ([decred/dcrd#2600](https://github.com/decred/dcrd/pull/2600))

### Code Contributors (alphabetical order)

- Dave Collins


# dcrwallet v1.6.1

This release focuses on fixing issues with the new VSP fee payments and
account mixing.

## New Features

* A `WalletServer.SetVspdVoteChoice` gRPC method was added, allowing clients to
  update the agenda preferences for the new VSP software.

## Bug Fixes

* An additional transaction may be created now when an account does not have
  enough UTXOs to pay for both a ticket and a VSP fee.  This avoids insufficient
  balance errors where possible and prevents the user from needing to split a
  UTXO themselves.

* Several issues causing double spends when dealing with unpublished
  transactions were corrected.  This improves the reliability of paying the VSP
  fee.

* Account balances no longer report the outputs of unpublished transactions as
  spendable with minconf=0.  Instead, these balances are added to the
  unconfirmed balance.

* Account and output mixing now considers the required fee necessary when
  selecting which common output amount and output count to mix with.  This fixes
  mixing for some outputs which are currently being rejected by the
  CoinShuffle++ server due to not paying enough of the required transaction fee.

* Fixed a chance to select wrong mix denomination during output mixing, which
  could result in a very high transaction fee.

* The `signrawtransaction` JSON-RPC method was changed to return an error
  if the transaction being signed has no inputs.

* The salsa20 and blake2b dependencies were updated to prevent possible memory
  corruption caused by smashing the SP register in optimized assembly
  implementations.

## Changelog

All commits since the last release may be viewed on GitHub
[here](https://github.com/decred/dcrwallet/compare/release-v1.6.0...release-v1.6.1).


### Code Contributors (alphabetical order)

- Josh Rickmar


# Decrediton v1.6.1

This patch release fixes several issues discovered in the v1.6.0 release.

Most changes are focused on improving the staking experience with the new VSP
system: unexpected insufficient balance error is fixed, successes and failures
of various staking operations are better reported now, setting consensus vote
choices on the new VSP was implemented.

## Updates

* Consensus change voting is now working as expected.  When a user sets their
  vote choice, it is updated in their local wallet and also sent to any legacy
  VSP.  Every live ticket they have assigned with a new VSP is updated as well.

* Due to the way the coins (UTXOs) are handled when purchasing tickets, there is
  a possibility of the underlying dcrwallet purchasing fewer tickets than what
  the user requested.  This condition is now explained with a better message,
  to help the user understand why, for example, they only got 1 ticket when
  trying to buy 2.

* Labels on the Staking tab have been updated to make things more clear with the
  new tickets and the old tickets.

* An initial Traditional Chinese translation was completed by @smartwojak and
  verified by long-standing community member Hugo Chang (@changhugo).

* Running or attempting multiple things at the same time is no longer allowed
  to avoid possible issues or unexpected errors.  For instance, when the mixer
  is running, users may not purchase tickets or run the autobuyer, and vice
  versa.  To perform an action, the user needs to turn off a running activity
  before proceeding. Several tooltips have been added to make the user aware of
  the situation.

* Loading indicators have been added to various buttons related to ticket
  purchasing, to indicate that the user should wait for a long-running operation
  (like mixed ticket purchasing) to complete.

* Success and failure messages have been added to various new ticket purchasing
  actions. Now users will be shown a message when they successfully complete (or
  receive errors for) the following actions: verify that visible tickets are
  assigned to VSPs and have their fees paid (Process Managed), discover tickets
  not yet assigned to a VSP and pay their fees (Process Unmanaged), and sync
  failed VSP tickets.

## Bug Fixes

* Added a timeout when not receiving VSP status response within 5 seconds.

* Transaction history filtering has been fixed and now allows to select multiple
  types of transactions at once.

* Tickets will now show as "Processing", "Error" or "Paid" shortly after
  purchase.  Previously they would be shown as "Solo" until a restart or another
  block was mined.

* Added explicit wallet lock calls to ensure that wallet is locked after mixing
  or ticket autobuyer requests.

* There were a few reports of incorrectly created legacy ticket purchases
  due to a still unknown cause.  To work-around this we've added sanity checks
  prior to purchase request to dcrwallet to avoid any potential malformed
  requests from being sent.  This won't solve the core issue, but should at
  least notify users of something wrong occuring and give us data to investigate
  further.

## Changelog

All commits since the last release may be viewed on GitHub
[here](https://github.com/decred/decrediton/compare/release-v1.6.0...release-v1.6.1).

### Code Contributors (alphabetical order)

- Alex Yocom-Piatt
- Amir Massarwa
- bgptr
- Guilherme Marques
- JoeGruffins
- Matheus Degiovani
- Scott Christian
- smartwojak
- Victor Oliveira


# dcrdex v0.1.5

This patch release provides several important bug fixes.

Please read the
[initial release (v0.1.0) notes](https://github.com/decred/dcrdex/releases/tag/release-v0.1.0)
for important information and instructions.

## Client (dexc)

### Features and Improvements

* The user's account ID is now logged on connection and authentication with a
  DEX server.
  ([8ce328](https://github.com/decred/dcrdex/commit/8ce328cacc4d7b0d35973a39917eef48ac1d1d64))

### Fixes

* Fix a possible panic when reconfiguring a wallet that is not connected.
  ([dfe4cd](https://github.com/decred/dcrdex/commit/dfe4cd12234d1d17d6114f3de8f062ff912c594b))

* When resuming trades on startup and login, counterparty contract audits now
  retry repeatedly, as is the case when an audit request is initially received.
  This prevents a match from being incorrectly revoked on startup if the
  wallet/node fails to locate the counterparty contract immediately.
  ([dfe4cd](https://github.com/decred/dcrdex/commit/dfe4cd12234d1d17d6114f3de8f062ff912c594b))

* The client's database subsystem is always started first and stopped last.
  This is a prerequisite for the following wallet lock-on-shutdown change.
  ([b4ef3f](https://github.com/decred/dcrdex/commit/b4ef3ff01f3a1567aecf762c5db75f83a9687d64))

* On shutdown of client `Core`, the wallets are now locked even if the
  `PromptShutdown` function is not used.  This does not affect dexc users,
  only direct Go consumers of the `client/core.Core` type.
  ([70044e](https://github.com/decred/dcrdex/commit/70044e68740faffc4888c6f4b4303806531a0255))

* Fix a possible interruption of the DEX reconnect loop if the config response
  timed out.
  ([4df683](https://github.com/decred/dcrdex/commit/4df683a10d755d71f37c979655b6ceea6343db8d))

* Update the crypto/x/blake2 dependency to prevent silent memory corruption
  from the hash function's assembly code.
  ([c67af3](https://github.com/decred/dcrdex/commit/c67af3f3b88750e69957e019d9eacc80d6aa7555))

* Handle orders that somehow lose their funding coins.  Previously, such
  orders would forever be logged at startup but never retired, and any matches
  from such orders that required swap negotiation of other recovery would have
  been improperly abandoned.
  ([a7b5aa](https://github.com/decred/dcrdex/commit/a7b5aa0a67dd2962c33d229cf101c59e85cb7b85))

## Server (dcrdex)

There are no substantive server changes, just a few logging improvements.

## Code Summary

11 commits, 13 files changed, 564 insertions(+), and 254 deletions(-)

https://github.com/decred/dcrdex/compare/v0.1.4...v0.1.5

3 contributors

- Brian Stafford (@buck54321) (review)
- David Hill (@dajohi) (blake2 fix)
- Jonathan Chappelow (@chappjc)

## Install

To install Decrediton desktop wallet, download, uncompress, and run
[Decrediton Linux](https://github.com/decred/decred-binaries/releases/download/v1.6.2/decrediton-v1.6.2.AppImage)
or
[Decrediton macOS](https://github.com/decred/decred-binaries/releases/download/v1.6.2/decrediton-v1.6.2.dmg)
or
[Decrediton Windows](https://github.com/decred/decred-binaries/releases/download/v1.6.2/decrediton-v1.6.2.exe).

To install the command-line tools, please see
[dcrinstall](https://github.com/decred/decred-release/tree/master/cmd/dcrinstall).

See decred-v1.6.2-manifest.txt and the other manifest files for SHA-256 hashes
and the associated .asc signature files to confirm those hashes.

See [README.md](./README.md#verifying-binaries) for more info on verifying the
files.


## Contents

* [dcrd](#dcrd-v162)
* [dcrwallet](#dcrwallet-v162)
* [Decrediton](#decrediton-v162)

<a name="dcrd-v162" />

# dcrd v1.6.2

This is a patch release of dcrd to introduce a quality of life change for
lightweight clients, such as SPV wallets, by not sending them a certain class
of announcements that only full nodes are equiped to handle.

## Changelog

This patch release consists of 2 commits from 1 contributor which total to 3
files changed, 55 additional lines of code, and 31 deleted lines of code.

All commits since the last release may be viewed on GitHub
[here](https://github.com/decred/dcrd/compare/release-v1.6.1...release-v1.6.2).

### Protocol and Network

- server: Only send fast block anns to full nodes ([decred/dcrd#2609](https://github.com/decred/dcrd/pull/2609))

### Misc

- release: Bump for 1.6.2 ([decred/dcrd#2629](https://github.com/decred/dcrd/pull/2629))

### Code Contributors (alphabetical order)

- Dave Collins

<a name="dcrwallet-v162" />

# dcrwallet v1.6.2

This release focuses on bug fixes and feature improvements for VSP ticketbuying
and change mixing.

## New Features

* A `accountunlocked` JSON-RPC method was added, allowing clients to determine
  whether an account has been encrypted with a unique passphrase, and if it is
  currently unlocked if so.

* The `setvotechoices` JSON-RPC method will now use the vspd client to set vote
  choices at the VSP, if any is configured in the application settings and the
  ticket was bought for the VSP.

## Bug Fixes

* A UTXO selection issue which caused "low balance" errors during the additional
  split transaction sometimes necessary when purchasing tickets with a VSP was
  fixed.  Some UTXOs of the account were not always being considered during the
  creation of this transaction, which led to the balance errors.

* A check for a too-low fee when mixing at the smallest common amount was added.
  This previously was causing "invalid submission" errors, as the server would
  reject the submission for not paying enough fee.

## Changelog

All commits since the last release may be viewed on GitHub
[here](https://github.com/decred/dcrwallet/compare/release-v1.6.1...release-v1.6.2).

### Code Contributors (alphabetical order)

* Jonathan Chappelow
* Josh Rickmar

<a name="decrediton-v162" />

# Decrediton v1.6.2

This patch release for Decrediton includes just a few small changes for copy
and buttons missing text.

## Bug Fixes

* Missing Legacy Ticket purchase button text.

* Incorrect copy on Governance for thresholds for upgrading the network.

## Changelog

All commits since the last release may be viewed on GitHub
[here](https://github.com/decred/decrediton/compare/release-v1.6.1...release-v1.6.2).

### Code Contributors (alphabetical order)

- Alex Yocom-Piatt
- Amir Massarwa
- bgptr
- Matheus Degiovani

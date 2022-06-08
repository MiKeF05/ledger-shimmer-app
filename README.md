# Shimmer App for Ledger Hardware Wallets

[![license](https://img.shields.io/github/license/IOTA-Ledger/blue-app-iota.svg)](https://github.com/IOTA-Ledger/blue-app-iota/blob/master/LICENSE)


***It is strongly recommended to take a few minutes to read this document. You need to make sure to fully understand how the Ledger hardware wallet works in combination with the Shimmer network.***

## Table of contents

- [Shimmer App for Ledger Hardware Wallets](#shimmer-app-for-ledger-hardware-wallets)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
    - [Terminology](#terminology)
    - [Shimmer Block](#shimmer-block)
    - [Parts of an Shimmer Block](#parts-of-an-shimmer-block)
  - [How Ledger Hardware Wallets Work](#how-ledger-hardware-wallets-work)
  - [Shimmer Specific Considerations for Ledger Hardware Wallets](#shimmer-specific-considerations-for-ledger-hardware-wallets)
    - [Shimmer User-Facing App Functions](#shimmer-user-facing-app-functions)
      - [Functions](#functions)
      - [Display](#display)
    - [Shimmer Security Concerns Relating to Ledger Hardware Wallets](#shimmer-security-concerns-relating-to-ledger-hardware-wallets)
    - [Limitations of Ledger Hardware Wallets](#limitations-of-ledger-hardware-wallets)
  - [FAQ](#faq)
      - [I lost my ledger, what should I do now?](#i-lost-my-ledger-what-should-i-do-now)
  - [Development](#development)
    - [Preparing development environment](#preparing-development-environment)
    - [Compile and load the Shimmer Ledger app](#compile-and-load-the-shimmer-ledger-app)
  - [Specification](#specification)

---

## Introduction

Shimmer is an unique cryptocurrency with specific design considerations that must be taken into account. This document explains how the Ledger hardware wallet works and how to stay safe when using a Ledger to store Shimmer funds.

### Terminology

*Seed:* A single secret key from which all private keys are derived (aka "*24 words*", [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)).

*Private Key:* Private keys are derived from the seed ([BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) with ED25519 curve, [SLIP10](https://github.com/satoshilabs/slips/blob/master/slip-0010.md). The private key is used to generate a public key. The private key is also used to prove that you own said public key by means of creating a signature for a specific transaction.

*Public Key:* The public part of a private/public key pair.

*Address:* A hashed representation of the public key that is used to receive funds.

*Input:* The unspent transaction output (UTXO) from which funds are transfered.

*Output:* The address to which funds are transfered.

*Change/Remainder:* After sending funds to a 3rd party, all remaining funds on the account must be transferred to a new address - this is called the change or remainder address.

### Shimmer Block

A block is mainly just a group of inputs and outputs. If Bob has 10SMR and he wants to send 3SMR to Alice then the block could look like this:

**input:** Bob -10SMR

**output:** Alice +3SMR

**output:** Bob +7SMR (change output / remainder)

This example highlights how the Shimmer network handles blocks. First an input of 10SMR from Bob's address is used. After this 3SMR are sent to Alice and the remaining funds are sent to a new address belonging to Bob's seed.

All inputs require the private key to generate signatures which prove that you own the funds.

Because blocks are *atomic units*, the network will never accept Bob's input of -10SMR without accepting sending 3SMR to Alice and 7SMR to a new address owned by Bob (in this example). This new address where the remaining funds are sent to is called *remainder address.*

### Parts of a Shimmer Block

A Shimmer block is split into different parts. The first part is to create a block and signatures for this block.

The second part is to attach this block to the tangle. Therefore the block needs between one and eight tip messages which are requested from a node of the Shimmer network.

The Ledger hardware wallet is **only** responsible for generating signatures for a specific block. After that the host machine (or anybody else for that matter) can take the signatures and broadcast it to the network (however the signatures are only valid for the specific block).

## How Ledger Hardware Wallets Work

The Ledger hardware wallet works by deterministically generating Shimmer keys based on your 24 word mnemonic (created when setting up the device).

Instead of storing the seed on your computer (which could be stolen by an attacker) the seed is **only** stored on the Ledger device itself. This ensures that the seed is never sent to the host machine it is connected to.

Instead the host machine needs to ask the Ledger device to provide the information (such as public keys or signatures). When blocks are created the host generates an unsigned block and sends it to the Ledger device for signing. The Ledger uses the private keys associated with the inputs to generate unique signatures and transmits **only the signatures** to the host machine.

The host is then able to use these signatures (which are only valid for that specific block) to broadcast the block to the network. However, as you can see, neither the seed nor any of the private keys ever leave the Ledger device.

See [Ledger's documentation](https://developers.ledger.com/) to get more info about the inner workings of the Ledger hardware wallets.

## Shimmer Specific Considerations for Ledger Hardware Wallets

### Shimmer User-Facing App Functions

#### Functions

- *Display Address:* The wallet is able to ask the Ledger device to display the address for a specific index of the seed. It **only** accepts the index and subsequently generates the address itself. It thus verifies that the address belongs to the corresponding Ledger.

    *Note: Only the remainder address (shown as "Remainder") will be verified as belonging to the Ledger. Outputs shown as "Send To" are not.*

- *Sign Transaction:* The wallet will generate a block for the user to approve before the Ledger signs it. **Ensure all amounts and addresses are correct before signing**. These signatures are then sent back to the host machine.

#### Display

**TODO**


### Shimmer Security Concerns Relating to Ledger Hardware Wallets

All warnings on the Ledger are there for a reason. **MAKE SURE TO READ THEM** and refer to this document if there is any confusion.

- **Don't rush through signing a block.**
    Before a block is signed all outputs of a block are shown on the display. The user can scroll through the individual outputs and finally choose whether to sign the block ("Accept") or not ("Reject").

    Outputs that go to 3rd party addresses are shown on the display with "Send To". The address to which the rest is sent is shown as "remainder".

    In addition to the amount the remainder also shows the BIP32 path. This address is calculated on the Ledger and ensures that the rest of the SMR funds are sent to an address owned by the Ledger device.

    If the input amount perfectly matches the output amount then there will be no remainder. **If there is no remainder, double check that you are sending the proper amount to the proper address because there is no remainder being sent back to your seed.**

### Limitations of Ledger Hardware Wallets

Due to the memory limitations of the Ledger Nano S/X, the blocks have certain restrictions. The Ledger Nano S can only accept blocks with about 17 inputs/outputs (e.g. 4 inputs, 13 outputs). The Ledger Nano X has the ability to accept blocks with approximately 180 inputs/outputs. 

## FAQ

#### I lost my ledger, what should I do now?

Hopefully you wrote down your 24 recovery words and your optional passphrase in a safe place. If not, all your funds are lost.

If you did, the best solution is to buy a new Ledger and enter your 24 recovery words and your optional passphrase in the new device.<br>
After installation of the Shimmer Ledger app, all your funds are restored. Take care to reinitialize your seed index correctly.

## Development

You either can

- run the app in a Ledger Nano S/X simulator or
- load the app on your read Ledger Nano S

In both cases, you find instructions here: [Ledger-Shimmer-App-Docker Repository](docker)

### Preparing Development Environment

For active development it might be easier to install the development environment locally instead of using Docker:

- Clone this repo
- Ensure that all git submodules are initialized
    ```
    git submodule update --init --recursive
    ```
- Set up your development environment according to [Ledger Documentation - Getting Started](https://developers.ledger.com/docs/nano-app/build/).

### Compile and Load The Shimmer Ledger App

After the development environment has been installed, the app can be built and installed in the following way:

- Connect your Ledger to the computer and unlock it
- To load the app, be sure that the dashboard is opened on the Ledger
- Run the following command to compile the app from source and load it
    ```
    make load
    ```
- Accept all the messages on the Ledger

## Specification

See: [APDU API Specification](docs/specification_shimmer.md)

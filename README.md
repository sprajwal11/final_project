# final_project

# NFT Contract README

## Introduction

Welcome to the NFT Contract README! This is a smart contract created for managing non-fungible tokens (NFTs) on the Blockchain Network platform. The contract offers key functionalities for minting, burning, and transferring NFTs, which represent unique digital or physical assets on the blockchain. This README serves as your comprehensive guide on how to deploy and interact with the NFT contract.

## Functionalities

### Mint
- **Purpose**: Generate new NFTs.
- **How to Use**: Mint new NFTs by specifying the content or asset and the recipient's address.

### Burn
- **Purpose**: Permanently remove NFTs from circulation.
- **How to Use**: Permanently destroy specific NFTs.

### Transfer
- **Purpose**: Change ownership of NFTs.
- **How to Use**: Transfer NFTs from one address to another, allowing actions like buying, selling, gifting, or other forms of ownership transfer.

# Deployment

## Código Platform

Código is an AI-Powered Code Generation Platform designed for blockchain developers and web3 teams. It helps save development time and enhances code security across various blockchains.

## Getting Started

To get started with Código, you have two options:

- **Código Studio**: A web-based IDE environment that includes all the necessary tools and programs for developing Solana programs using the CIDL. You can access it [here](https://studio.codigo.ai).

- **Código CLI**: If you prefer to work from your local environment, download the latest version of the [Código CLI](https://github.com/Codigo-io/platform/releases) tailored to your operating system. After downloading, place the Código CLI in your preferred location and add this location to your environment PATH.

## CIDL Quickstart

This quickstart guide will introduce you to Código’s Interface Description Language (CIDL) by building a simple Solana counter program. If you're working from your local environment, ensure you have successfully installed and configured the Solana tool suite. If you're using Código Studio, don't worry, as the Solana tool suite is pre-installed and configured.

### 1. NFT Contract

Start by creating an `nft.yaml` file and copy-pasting the following CIDL:

```yaml

cidl: "0.8"
info:
  name: nft
  title: RiseIn NFT
  version: 0.0.1
  license:
    name: Unlicense
    identifier: Unlicense
types:
  GemMetadata:
    solana:
      seeds:
        - name: "gem"
        - name: mint
          type: sol:pubkey
    fields:
      - name: color
        type: string
        solana:
          attributes: [ cap:16 ]
      - name: rarity
        type: string
        solana:
          attributes: [ cap:16 ]
      - name: short_description
        type: string
        solana:
          attributes: [ cap:255 ]
      - name: mint
        type: sol:pubkey
      - name: assoc_account
        type: rs:option<sol:pubkey>
methods:
  - name: mint
    uses:
      - csl_spl_token.initialize_mint2
      - csl_spl_assoc_token.create
      - csl_spl_token.mint_to
      - csl_spl_token.set_authority
    inputs:
      - name: mint
        type: csl_spl_token.Mint
        solana:
          attributes: [ init ]
      - name: gem
        type: GemMetadata
        solana:
          attributes: [ init ]
          seeds:
            mint: mint
      - name: color
        type: string
      - name: rarity
        type: string
      - name: short_description
        type: string
  - name: transfer
    uses:
      - csl_spl_assoc_token.create
      - csl_spl_token.transfer_checked
    inputs:
      - name: mint
        type: csl_spl_token.Mint
      - name: gem
        type: GemMetadata
        solana:
          attributes: [ mut ]
          seeds:
            mint: mint
  - name: burn
    uses:
      - csl_spl_token.burn
    inputs:
      - name: mint
        type: csl_spl_token.Mint
      - name: gem
        type: GemMetadata
        solana:
          attributes: [ mut ]
          seeds:
            mint: mint
```

### 2. Generate the Solana Program and Client Library

In your terminal, enter the following command:
```shell
codigo solana generate nft.yaml

```
After this step, two new directories, "program_client" and "program," will be created relative to the `nft.yaml` file.

### 3. Implement the Business Logic

In your terminal, enter the following command:

In the `program` directory, you'll find a "src" directory containing three `.rs` files: `mint.rs`, `burn.rs`, and transfer.rs. You need to implement the business logic in these files just below the comment line `// Implement your business logic here....`

### Mint business logic:
```rust
[Insert Mint business logic here]
```
### Burn business logic:
```rust
[Insert Burn business logic here]

```
### Transfer business logic:
```rust
[Insert Transfer business logic here]

```


### 4. Build and Deploy the Program

In your terminal, navigate to the `program` directory and run:


```shell
cargo build-sbf

```

If an error occurs, execute the following command:


```shell
cargo install build-bpf


```
After that, run a local Solana validator by opening a new terminal and typing:


```shell
solana-test-validator


```
Now, deploy the program by opening another terminal and navigating to the `program` directory. Execute the following command:


```shell
solana program deploy target/deploy/counter.so

```
Make sure to save the program ID provided as you'll need it later.

### 5. Test your contract

Create a new file named `app.ts` inside the directory `program_client` and copy and paste the following code into the `app.ts` file:

> Replace the “PASTE_YOUR_PROGRAM_ID” with the program id you got when deploying the Solana program.

```typescript
import {Connection, Keypair, PublicKey, sendAndConfirmTransaction, SystemProgram, Transaction,} from "@solana/web3.js";
import * as fs from "fs/promises";
import * as path from "path";
import * as os from "os";
import {
    burnSendAndConfirm,
    CslSplTokenPDAs,
    deriveGemMetadataPDA,
    getGemMetadata,
    initializeClient,
    mintSendAndConfirm,
    transferSendAndConfirm,
} from "./index";
import {getMinimumBalanceForRentExemptAccount, getMint, TOKEN_PROGRAM_ID,} from "@solana/spl-token";

async function main(feePayer: Keypair) {
    const args = process.argv.slice(2);
    const connection = new Connection("https://api.devnet.solana.com", {
        commitment: "confirmed",
    });

    const progId = new PublicKey(args[0]!);

    initializeClient(progId, connection);


    /**
     * Create a keypair for the mint
     */
    const mint = Keypair.generate();
    console.info("+==== Mint Address  ====+");
    console.info(mint.publicKey.toBase58());

    /**
     * Create two wallets
     */
    const johnDoeWallet = Keypair.generate();
    console.info("+==== John Doe Wallet ====+");
    console.info(johnDoeWallet.publicKey.toBase58());

    const janeDoeWallet = Keypair.generate();
    console.info("+==== Jane Doe Wallet ====+");
    console.info(janeDoeWallet.publicKey.toBase58());

    const rent = await getMinimumBalanceForRentExemptAccount(connection);
    await sendAndConfirmTransaction(
        connection,
        new Transaction()
            .add(
                SystemProgram.createAccount({
                    fromPubkey: feePayer.publicKey,
                    newAccountPubkey: johnDoeWallet.publicKey,
                    space: 0,
                    lamports: rent,
                    programId: SystemProgram.programId,
                }),
            )
            .add(
                SystemProgram.createAccount({
                    fromPubkey: feePayer.publicKey,
                    newAccountPubkey: janeDoeWallet.publicKey,
                    space: 0,
                    lamports: rent,
                    programId: SystemProgram.programId,
                }),
            ),
        [feePayer, johnDoeWallet, janeDoeWallet],
    );

    /**
     * Derive the Gem Metadata so we can retrieve it later
     */
    const [gemPub] = deriveGemMetadataPDA(
        {
            mint: mint.publicKey,
        },
        progId,
    );
    console.info("+==== Gem Metadata Address ====+");
    console.info(gemPub.toBase58());

    /**
     * Derive the John Doe's Associated Token Account, this account will be
     * holding the minted NFT.
     */
    const [johnDoeATA] = CslSplTokenPDAs.deriveAccountPDA({
        wallet: johnDoeWallet.publicKey,
        mint: mint.publicKey,
        tokenProgram: TOKEN_PROGRAM_ID,
    });
    console.info("+==== John Doe ATA ====+");
    console.info(johnDoeATA.toBase58());

    /**
     * Derive the Jane Doe's Associated Token Account, this account will be
     * holding the minted NFT when John Doe transfer it
     */
    const [janeDoeATA] = CslSplTokenPDAs.deriveAccountPDA({
        wallet: janeDoeWallet.publicKey,
        mint: mint.publicKey,
        tokenProgram: TOKEN_PROGRAM_ID,
    });
    console.info("+==== Jane Doe ATA ====+");
    console.info(janeDoeATA.toBase58());

    /**
     * Mint a new NFT into John's wallet (technically, the Associated Token Account)
     */
    console.info("+==== Minting... ====+");
    await mintSendAndConfirm({
        wallet: johnDoeWallet.publicKey,
        assocTokenAccount: johnDoeATA,
        color: "Purple",
        rarity: "Rare",
        shortDescription: "Only possible to collect from the lost temple event",
        signers: {
            feePayer: feePayer,
            funding: feePayer,
            mint: mint,
            owner: johnDoeWallet,
        },
    });
    console.info("+==== Minted ====+");

    /**
     * Get the minted token
     */
    let mintAccount = await getMint(connection, mint.publicKey);
    console.info("+==== Mint ====+");
    console.info(mintAccount);

    /**
     * Get the Gem Metadata
     */
    let gem = await getGemMetadata(gemPub);
    console.info("+==== Gem Metadata ====+");
    console.info(gem);
    console.assert(gem!.assocAccount!.toBase58(), johnDoeATA.toBase58());

    /**
     * Transfer John Doe's NFT to Jane Doe Wallet (technically, the Associated Token Account)
     */
    console.info("+==== Transferring... ====+");
    await transferSendAndConfirm({
        wallet: janeDoeWallet.publicKey,
        assocTokenAccount: janeDoeATA,
        mint: mint.publicKey,
        source: johnDoeATA,
        destination: janeDoeATA,
        signers: {
            feePayer: feePayer,
            funding: feePayer,
            authority: johnDoeWallet,
        },
    });
    console.info("+==== Transferred ====+");

    /**
     * Get the minted token
     */
    mintAccount = await getMint(connection, mint.publicKey);
    console.info("+==== Mint ====+");
    console.info(mintAccount);

    /**
     * Get the Gem Metadata
     */
    gem = await getGemMetadata(gemPub);
    console.info("+==== Gem Metadata ====+");
    console.info(gem);
    console.assert(gem!.assocAccount!.toBase58(), janeDoeATA.toBase58());

    /**
     * Burn the NFT
     */
    console.info("+==== Burning... ====+");
    await burnSendAndConfirm({
        mint: mint.publicKey,
        wallet: janeDoeWallet.publicKey,
        signers: {
            feePayer: feePayer,
            owner: janeDoeWallet,
        },
    });
    console.info("+==== Burned ====+");

    /**
     * Get the minted token
     */
    mintAccount = await getMint(connection, mint.publicKey);
    console.info("+==== Mint ====+");
    console.info(mintAccount);

    /**
     * Get the Gem Metadata
     */
    gem = await getGemMetadata(gemPub);
    console.info("+==== Gem Metadata ====+");
    console.info(gem);
    console.assert(typeof gem!.assocAccount, "undefined");
}

fs.readFile(path.join(os.homedir(), ".config/solana/id.json")).then((file) =>
    main(Keypair.fromSecretKey(new Uint8Array(JSON.parse(file.toString())))),
);
```

Open the terminal, navigate to the `program_client` directory, and execute the following commands:

```shell
npm install ts-node --save-dev
```

Finally, to run the test, execute the following command:

```shell
npx ts-node app.ts
```

## Next steps

**Congratulations!** 🎉👏 you created your first Solana smart contract using the CIDL and integrated the generated TypeScript client library with an application. To summarize what we learned:

- CIDL stands for Código Interface Description Language, and it is the input for Código’s Generator.
- After completing the CIDL, developers only need to concentrate on implementing the business logic of the smart contract. 100% of the client libraries and smart contracts boilerplate are automatically generated.

These links may help you on your journey to writing smart contracts with the CIDL:

- [Overview](https://docs.codigo.ai/)
- [Learning the Basics](https://docs.codigo.ai/c%C3%B3digo-interface-description-language/learning-the-basics)
- [Part I - Building Solana Programs](https://docs.codigo.ai/guides/part-1-building-solana-programs)

## Interaction

Users can interact with the NFT contract by:

- Minting new NFTs: The contract owner or designated users can mint NFTs by calling the mint function.
- Burning NFTs: The contract owner can remove specific NFTs from circulation using the burn function.
- Transferring NFTs: Users can transfer NFT ownership by calling the transfer function.


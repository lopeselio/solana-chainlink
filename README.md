<p align="center">
  <a href="https://solana.com">
    <img alt="Solana" src="https://i.imgur.com/uBVzyX3.png" width="250" />
    <img alt="Chainlink" src="https://i.imgur.com/GPpjgHf.png" width="100" height="100" />
  </a>
</p>


# Chainlink Price Feeds on Solana

Chainlink is a decentralized oracle network that currently provides decentralization at both the oracle and data source level. By using multiple independent Chainlink nodes, the user can defend against one oracle being a single point of failure. Similarly, using multiple data sources for sourcing market prices, the user can defend against one data source being a single source of truth.

This project demonstrates how to use the [Chainlink Price Data Feeds](https://docs.chain.link/docs/using-chainlink-reference-contracts/) to enable Solana Rust programs to retrieve the latest pricing data of an asset in a single call.

We are going to:

* write and deploy a Rust Program on the Solana [Devnet](https://docs.solana.com/clusters)
* Retrieve data feeds using the Solana Web3 JavaScript API using Node

## Quick Start

The following dependencies are required to build and run this example, depending
on your OS, they may already be installed:

- Install node (v14 recommended)
- Install npm
- Install Rust v1.56.1 or later from https://rustup.rs/
- Install Solana v1.8.2 or later from
  https://docs.solana.com/cli/install-solana-cli-tools

If this is your first time using Rust, these [Installation
Notes](README-installation-notes.md) might be helpful.

## Getting Started
```bash
git clone https://github.com/solana-labs/example-helloworld.git && cd example-helloworld
```
Open the project in your preferred IDE. In the `src` folder, you'll find two ways to build the program. One uses the C language while the other uses Rust. Since we are building with Rust, go ahead and open the `program-rust` folder and ignore `program-c`. There is also a `client` folder but we'll get to it later. For now, we are interested in `lib.rs` inside the program-rust src. The code looks like this:

```
use borsh::{BorshDeserialize, BorshSerialize};
use solana_program::{
    account_info::{next_account_info, AccountInfo},
    entrypoint,
    entrypoint::ProgramResult,
    msg,
    program_error::ProgramError,
    pubkey::Pubkey,
};

/// Define the type of state stored in accounts
#[derive(BorshSerialize, BorshDeserialize, Debug)]
pub struct GreetingAccount {
    /// number of greetings
    pub counter: u32,
}

// Declare and export the program's entrypoint
entrypoint!(process_instruction);

// Program entrypoint's implementation
pub fn process_instruction(
    program_id: &Pubkey, // Public key of the account the hello world program was loaded into
    accounts: &[AccountInfo], // The account to say hello to
    _instruction_data: &[u8], // Ignored, all helloworld instructions are hellos
) -> ProgramResult {
    msg!("Hello World Rust program entrypoint");

    // Iterating accounts is safer then indexing
    let accounts_iter = &mut accounts.iter();

    // Get the account to say hello to
    let account = next_account_info(accounts_iter)?;

    // The account must be owned by the program in order to modify its data
    if account.owner != program_id {
        msg!("Greeted account does not have the correct program id");
        return Err(ProgramError::IncorrectProgramId);
    }

    // Increment and store the number of times the account has been greeted
    let mut greeting_account = GreetingAccount::try_from_slice(&account.data.borrow())?;
    greeting_account.counter += 1;
    greeting_account.serialize(&mut &mut account.data.borrow_mut()[..])?;

    msg!("Greeted {} time(s)!", greeting_account.counter);

    Ok(())
}
//Minus the tests.
```
Let's try to first examine the code line by line, so that we understand the basics of Rust, and writing programs(smart-contracts) in Rust
Rust allows us to build on code written by others using crates. A crate can contain several modules and we specify the modules we want to bring into scope. It is just analogous to npm package imports in javascript.

```
use borsh::{BorshDeserialize, BorshSerialize};
```
Borsh is the binary serialisation format, designed to serialize any objects to canonical and deterministic set of bytes. `BorsheSerialize` is used for converting data(structs, ints, enums, etc) into bytecode while `BorsheDeserialize` reconstructs the bytecode into data. Serializing is necessary because the programs must be parsed in BPF format.

Next we import a crate containing a bunch of Solana source code that we'll leverage to write on-chain programs

```
use solana_program::{
    account_info::{next_account_info, AccountInfo},
    entrypoint,
    entrypoint::ProgramResult,
    msg,
    program_error::ProgramError,
    pubkey::Pubkey,
};
```
1. `account_info` contains `next_account_info` which is a public function that returns the next `AccountInfo` or a `NotEnoughAccountKeys` error.
2. Entrypoint Macros are a way of writing code that writes other code. They reduce the amount of code you need to write!
3. We then bring `ProgramResult`, that returns `Ok` if the program runs well or `ProgramError` if the program fails. Result is an enum in Rust which is defined as having two variants, `Ok` and `Err`. We use it for error handling.
4. msg is a macro that's used for logging in Solana. ProgramError allows you to implement program-specific error types and see them returned by the Solana runtime.
5. We use the `pubkey` it to pass the public keys of our accounts.


### Configure CLI

> If you're on Windows, it is recommended to use [WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10) to run these commands

1. Set CLI config url to localhost cluster

```bash
solana config set --url localhost
```

2. Create CLI Keypair

If this is your first time using the Solana CLI, you will need to generate a new keypair:

```bash
solana-keygen new
```


### Install npm dependencies

```bash
npm install
```

### Build the on-chain program

There is both a Rust and C version of the on-chain program, whichever is built
last will be the one used when running the example.

```bash
npm run build:program-rust
```

```bash
npm run build:program-c
```

### Deploy the on-chain program

```bash
solana program deploy dist/program/helloworld.so
```

### Run the JavaScript client

```bash
npm run start
```

### Expected output

Public key values will differ:

```bash
Let's say hello to a Solana account...
Connection to cluster established: http://localhost:8899 { 'feature-set': 2045430982, 'solana-core': '1.7.8' }
Using account AiT1QgeYaK86Lf9kudqKthQPCWwpG8vFA1bAAioBoF4X containing 0.00141872 SOL to pay for fees
Using program Dro9uk45fxMcKWGb1eWALujbTssh6DW8mb4x8x3Eq5h6
Creating account 8MBmHtJvxpKdYhdw6yPpedp6X6y2U9dCpdYaZJdmwV3A to say hello to
Saying hello to 8MBmHtJvxpKdYhdw6yPpedp6X6y2U9dCpdYaZJdmwV3A
8MBmHtJvxpKdYhdw6yPpedp6X6y2U9dCpdYaZJdmwV3A has been greeted 1 times
Success
```


### Customizing the Program

To customize the example, make changes to the files under `/src`.  If you change
any files under `/src/program-rust` or `/src/program-c` you will need to
[rebuild the on-chain program](#build-the-on-chain-program) and [redeploy the program](#deploy-the-on-chain-program).

Now when you rerun `npm run start`, you should see the results of your changes.

## Pointing to a public Solana cluster

Solana maintains three public clusters:
- `devnet` - Development cluster with airdrops enabled
- `testnet` - Tour De Sol test cluster without airdrops enabled
- `mainnet-beta` -  Main cluster

Use the Solana CLI to configure which cluster to connect to.

To point to `devnet`:
```bash
solana config set --url devnet
```

To point back to the local cluster:
```bash
solana config set --url localhost
```


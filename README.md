# DeFi Stablecoin

This repository contains the code for a decentralised, over-collateralised stablecoin protocol built as part of the **Cyfrin Updraft Foundry course**.
The goal of this project is to allow users to deposit crypto collateral (WETH & WBTC) and mint a stable token pegged to the US dollar. The protocol enforces a minimum health factor, triggers liquidations when positions become under-collateralised and includes extensive fuzz/invariant tests to ensure solvency.

The overall design is split into two contracts:

1.  `DecentralizedStableCoin.sol` - An ERC-20 token representing the stablecoin (DSC). Only the engine contract can mint or burn DSC.
2.  `DSCEngine.sol` - The engine that handles collateral deposits, minting & burning DSC, health checks, and liquidations.

  


# Table of Contents

-   [About](#about)
-   [Getting Started](#getting-started)
    -   [Requirements](#requirements)
    -   [Quickstart](#quickstart)
    -   [Optional Gitpod](#optional-gitpod)
    -   [Updates](#updates)
-   [Usage](#usage)
    -   [Start a local node](#start-a-local-node)
    -   [Deploy](#deploy)
    -   [Deploy - Other Network](#deploy---other-network)
-   [Testing](#testing)
    -   [Test Coverage](#test-coverage)
-   [Deployment to a testnet or mainnet](#deployment-to-a-testnet-or-mainnet)
-   [Scripts](#scripts)
-   [Estimate gas](#estimate-gas)
-   [Formatting](#formatting)
-   [Slither](#slither)
-   [Additional Info](#additional-info)
-   [Summary](#summary)
-   [Thank you!](#thank-you)
-   [Contact](#contact)
-   [License](#license)

  


# About

The **DeFi Stablecoin** protocol enables users to deposit WETH and WBTC as collateral and mint a USD-pegged token called DSC. The system remains solvent by:

-   Requiring over-collateralisation (liquidation threshold at 50%, meaning 200% collateral).
-   Maintaining an on-chain health factor for each position. If the health factor drops below 1, anyone can liquidate the position and receive a bonus.
-   Using Chainlink price feeds (via a small `OracleLib`) with staleness checks to ensure fresh prices.

Note: This repository is for educational purposes. **Do not** use real funds in production deployments.

  


# Getting Started

## Requirements

To work with this repo you'll need:

-   **Git** - used to clone the repo. Verify with `git --version`
-   **Foundry** - the toolchain used for building, testing and deploying smart contracts. Install via [official docs](https://book.getfoundry.sh/getting-started/installation ) and verify with `forge --version`. At the time of writing, the course uses a nightly build.

## Quickstart

Clone the repository and build the contracts:

```bash
git clone https://github.com/VolodymyrStetsenko/DeFi_Stablecoin
cd DeFi_Stablecoin

# install submodules & dependencies
forge install

# compile everything
forge build
```

## Optional Gitpod

If you prefer a cloud-based environment, you can run this repo in Gitpod or any other online IDE. In that case you can skip the `clone` step and open the repo directly in your [workspace](https://gitpod.io/#https://github.com/VolodymyrStetsenko/DeFi_Stablecoin ).

## Updates

The version of OpenZeppelin contracts used in this course has changed since some lecture recordings. Make sure you install version `v4.9.x` (or the version specified in `foundry.toml`) instead of the latest to avoid mismatched function signatures.

  


# Usage

## Start a local node

Launch a local Anvil instance in a separate terminal:

```bash
anvil
```

This will fork the network you specify (defaults to mainnet). You can also use `anvil --fork-url $SEPOLIA_RPC_URL` to fork a testnet.

## Deploy

Deploy the contracts to your local node. Make sure Anvil is running before executing:

```bash
forge script script/DeployDSC.s.sol \
--broadcast \
--rpc-url http://127.0.0.1:8545
```

You will see the deployed addresses for `DSCEngine` and `DecentralizedStableCoin` in the output.

## Deploy - Other Network

To deploy to a public network, set environment variables (see [Deployment](#deployment-to-a-testnet-or-mainnet )) and then run:

```bash
forge script script/DeployDSC.s.sol \
--broadcast \
--verify \
--rpc-url $SEPOLIA_RPC_URL
```

  


# Testing

The course explores four test tiers: **unit**, **integration**, **forked**, and **staging**. This repository focuses on the first two via fuzzing and invariant tests. Run all tests with:

```bash
forge test -vv
```

## Test Coverage

Generate a coverage report to see which lines and branches are exercised:

```bash
forge coverage --report summary
```

For more detail, generate a debug report:

```bash
forge coverage --report debug
```

  


# Deployment to a testnet or mainnet

1.  **Setup environment variables**

    Set the following variables in a `.env` file or export them in your shell:

    -   `PRIVATE_KEY` – The private key of your deployer account (use a test key, not your main wallet).
    -   `SEPOLIA_RPC_URL` – The RPC endpoint for Sepolia or whichever network you're using.
    -   `ETHERSCAN_API_KEY` - Optional. Enables contract verification on Etherscan.

2.  **Get testnet ETH**

    Use a faucet (e.g. [faucets.chain.link](https://faucets.chain.link/ ) for Sepolia) to fund your deployer account.

3.  **Deploy**

    ```bash
    forge script script/DeployDSC.s.sol \
    --broadcast \
    --verify \
    --rpc-url $SEPOLIA_RPC_URL
    ```

  


# Scripts

Once deployed, you can interact with your contracts using `cast` commands. Below are some example calls (replace addresses with your own):

1.  **Deposit WETH** - deposit 0.1 ETH into WETH contract:
    ```bash
    cast send <WETH_CONTRACT_ADDRESS> "deposit()" --value 0.1ether --rpc-url $SEPOLIA_RPC_URL
    ```

2.  **Approve DSCEngine** - approve the engine to spend your WETH:
    ```bash
    cast send <WETH_CONTRACT_ADDRESS> "approve(address,uint256)" <DSCENGINE_ADDRESS> 0xDE0B6B3A7640000 --rpc-url $SEPOLIA_RPC_URL
    ```

3.  **Deposit collateral and mint DSC** - deposit and mint in one transaction:
    ```bash
    cast send <DSCENGINE_ADDRESS> "depositCollateralAndMintDsc(address,uint256,uint256)" \
    <WETH_CONTRACT_ADDRESS> 0xDE0B6B3A7640000 1000000000000000000 \ # 1 WETH and mint 1 DSC
    --rpc-url $SEPOLIA_RPC_URL
    ```

Refer to the scripts in the `script/` directory for more examples.

  


# Estimate gas

You can estimate gas usage of your contracts by generating a snapshot:

```bash
forge snapshot
```

This will produce a `gas-snapshot` file in the root of the project.

  


# Formatting

To ensure consistent code style across the repository, run the following:

```bash
forge fmt
```

You can add `forge fmt --check` to a continuous integration pipeline to reject PRs that are not formatted correctly.

  


# Slither

Run static analysis with Slither to detect common vulnerabilities:

```bash
pip install slither-analyzer
slither . --config-file slither.config.json
```

The configuration file (`slither.config.json`) specifies custom detectors and is included in the repo.

  


# Additional Info

Chainlink's `chainlink-brownie-contracts` library is an official repository maintained by the Chainlink team. It packages Chainlink's contracts from npm for consumption in Foundry projects. Using this library ensures you are using released versions rather than unreleased code.

  


# Summary

This stablecoin protocol demonstrates how to build a DeFi application with Foundry, covering contract design, price oracles, over-collateralisation, liquidations, fuzzing and invariant tests. It is for educational purposes and should not be used with real funds.

  


# Thank you!

If you found this project helpful, please consider following or supporting the creator. Your feedback and contributions are welcome!

  


# Contact

For questions, support or collaborations, reach out via:

-   **X (Twitter)**: [@carstetsen](https://twitter.com/carstetsen )
-   **Telegram**: [Zero2Auditor](https://t.me/Zero2Auditor )
-   **LinkedIn**: [Volodymyr Stetsenko](https://www.linkedin.com/in/володимир-стеценко-b9b81b265/ )
-   **GitHub**: [Cyfrin/foundry-defi-stablecoin-cu](https://github.com/Cyfrin/foundry-defi-stablecoin-cu/blob/main/README.md )

  


# License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for full details.

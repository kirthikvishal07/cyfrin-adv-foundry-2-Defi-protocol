# Decentralized Stablecoin with Foundry

This repository contains a minimal overcollateralized stablecoin system built with Foundry.

The project follows the common "DSC + engine" pattern:

- `DecentralizedStableCoin.sol` is the ERC20 stablecoin token (`DSC`)
- `DSCEngine.sol` holds the core collateral, mint, burn, redeem, and liquidation logic
- Chainlink price feeds are used to value collateral in USD
- The system is designed to stay overcollateralized at all times

The current collateral set is:

- `WETH`
- `WBTC`

This project looks inspired by the Maker-style model, but stripped down for learning and experimentation.

## What This Repo Includes

- Foundry smart contract project structure
- Deployment scripts for local Anvil and Sepolia-style configuration
- Mock price feeds and mock ERC20 collateral for local testing
- Unit tests for constructor validation, pricing math, and collateral deposit flows

## Architecture

### `DecentralizedStableCoin`

[`src/DecentralizedStableCoin.sol`](/D:/foundry/adv-foundry-nft-f26/adv-foundry-stablecoin-f26/src/DecentralizedStableCoin.sol)

This is the token contract for `DSC`.

- Inherits from OpenZeppelin `ERC20Burnable` and `Ownable`
- Only the owner can mint and burn
- Ownership is transferred to `DSCEngine` during deployment

### `DSCEngine`

[`src/DSCEngine.sol`](/D:/foundry/adv-foundry-nft-f26/adv-foundry-stablecoin-f26/src/DSCEngine.sol)

This is the main protocol contract.

Responsibilities:

- Accept approved collateral deposits
- Mint DSC against deposited collateral
- Burn DSC
- Redeem collateral
- Liquidate unhealthy positions
- Read token prices from Chainlink feeds

Important protocol constants currently in the contract:

- Liquidation threshold: `50`
- Liquidation bonus: `10%`
- Minimum health factor: `1e18`

At a high level, the health factor is intended to ensure users remain safely overcollateralized before they mint or withdraw.

## Local and Network Config

[`script/HelperConfig.s.sol`](/D:/foundry/adv-foundry-nft-f26/adv-foundry-stablecoin-f26/script/HelperConfig.s.sol)

The helper script chooses config based on `block.chainid`.

- On `Sepolia (11155111)`, it uses hardcoded feed/token addresses and reads `PRIVATE_KEY` from environment variables
- Otherwise, it deploys mock price feeds and mock collateral tokens for local development

Current mock prices:

- `ETH/USD = 2000`
- `BTC/USD = 1000`

## Deployment

[`script/DeployDSC.s.sol`](/D:/foundry/adv-foundry-nft-f26/adv-foundry-stablecoin-f26/script/DeployDSC.s.sol)

Deployment flow:

1. Load active network config
2. Build collateral token and feed arrays
3. Deploy `DecentralizedStableCoin`
4. Deploy `DSCEngine`
5. Transfer stablecoin ownership to the engine

## Project Structure

```text
.
|-- src/
|   |-- DSCEngine.sol
|   `-- DecentralizedStableCoin.sol
|-- script/
|   |-- DeployDSC.s.sol
|   `-- HelperConfig.s.sol
|-- test/
|   |-- mocks/
|   `-- unit/
|-- lib/
|-- foundry.toml
`-- makefile
```

## Getting Started

### Prerequisites

- [Foundry](https://book.getfoundry.sh/getting-started/installation)
- Git

Optional for Sepolia deployment:

- `SEPOLIA_RPC_URL`
- `PRIVATE_KEY`
- `ETHERSCAN_API_KEY`

### Install Dependencies

If submodules or libraries are not already present:

```bash
forge install
```

This repo already contains the main dependencies under `lib/`:

- OpenZeppelin Contracts
- Forge Std
- Chainlink Brownie Contracts
- Foundry DevOps

### Build

```bash
forge build
```

### Test

```bash
forge test
```

### Format

```bash
forge fmt
```

### Coverage

```bash
forge coverage
```

## Makefile Commands

[`makefile`](/D:/foundry/adv-foundry-nft-f26/adv-foundry-stablecoin-f26/makefile)

Available shortcuts include:

- `make build`
- `make test`
- `make coverage`
- `make snapshot`
- `make format`
- `make anvil`
- `make deploy`

Deploy locally:

```bash
make anvil
make deploy
```

Deploy to Sepolia:

```bash
make deploy ARGS="--network sepolia"
```

## Testing

Current unit tests live in:

- [`test/unit/DSCEngineTest.t.sol`](/D:/foundry/adv-foundry-nft-f26/adv-foundry-stablecoin-f26/test/unit/DSCEngineTest.t.sol)

The current test suite covers:

- Constructor array length validation
- USD value calculation
- Token amount from USD conversion
- Reverting on zero collateral deposit
- Reverting on unsupported collateral
- Successful collateral deposit and account info checks

## Current Status

This repo is a strong learning/project base, but it is not production-ready yet.

Notable observations from the current codebase:

- `getHealthFactor()` is declared but not implemented in `DSCEngine`
- The test suite is still fairly small compared to the protocol surface area
- There are debug `console.log` calls in core contract logic
- The repository includes a pre-generated `lcov.info`, but I could not re-run `forge test` in this environment because `forge` is not installed here

## Security Notes

This project manages collateral and mint/burn logic, so safety matters a lot.

Before using it beyond local learning or demos, it would be wise to add:

- More unit and invariant tests
- Fuzz testing for health factor and liquidation behavior
- Edge-case checks around zero debt, redemption, and liquidation
- A full security review or audit

## Foundry Config

[`foundry.toml`](/D:/foundry/adv-foundry-nft-f26/adv-foundry-stablecoin-f26/foundry.toml)

Key config values:

- `src = "src"`
- `out = "out"`
- `libs = ["lib"]`
- remappings for OpenZeppelin and Chainlink contracts
- `ffi = true`

## License

The project source files are marked with the `MIT` license.

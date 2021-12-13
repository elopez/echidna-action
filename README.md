# Echidna action

This action allows you to run `echidna-test` from within a GitHub Actions
workflow.

## Inputs

| Key                  | Description
|----------------------|------------
| `files`              | **Required** Solidity files or folders to analyze.
| `contract`           | Contract to analyze.
| `config`             | Config file (CLI arguments override config options).
| `format`             | Output format: json, text, none. Disables interactive UI.
| `corpus-dir`         | Directory to store corpus and coverage data.
| `check-asserts`      | Check asserts in the code.
| `multi-abi`          | Use multi-abi mode of testing.
| `test-limit`         | Number of sequences of transactions to generate during testing.
| `shrink-limit`       | Number of tries to attempt to shrink a failing sequence of transactions.
| `seq-len`            | Number of transactions to generate during testing.
| `contract-addr`      | Address to deploy the contract to test.
| `deployer`           | Address of the deployer of the contract to test.
| `sender`             | Addresses to use for the transactions sent during testing.
| `seed`               | Run with a specific seed.
| `crytic-args`        | Additional arguments to use in crytic-compile for the compilation of the contract to test.
| `solc-args`          | Additional arguments to use in solc for the compilation of the contract to test.
| `solc-version`       | Version of the Solidity compiler to use.
| `negate-exit-status` | Apply logical NOT to echidna-test's exit status (for testing the action).

## Outputs

This action has no outputs.

## Examples

### Basic usage

```yaml
uses: crytic/echidna-action@v1
with:
  files: contracts/
  contract: Marketplace
```

### Example workflow: Hardhat

The following is a complete GitHub Actions workflow example. It will trigger
with commits on `master` as well as any pull request opened against the `master`
branch. It will configure NodeJS 16.x on the runner, and then install project
dependencies (using `npm ci`). Once the environment is set up, it will build the
project (using `npx hardhat compile`) and finally run Echidna against a contract
called `TokenEchidna`. The workflow will fail if Echidna breaks any of the
invariants in `TokenEchidna`.

In this example, we are leveraging `crytic-args` to pass
`--hardhat-ignore-compile`. This skips building the project as part of the
Echidna action, and instead takes advantage of the already built contracts. This
is required, as the Echidna action environment does not have NodeJS available.

```yaml
name: Echidna Test

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout repository
      uses: actions/checkout@v2
    - name: Use Node.js 16
      uses: actions/setup-node@v2
      with:
        node-version: 16
        cache: 'npm'
    - name: Install dependencies
      run: npm ci
    - name: Compile contracts
      run: npx hardhat compile
    - name: Run Echidna
      uses: crytic/echidna-action@v1
      with:
        solc-version: 0.7.6
        files: .
        contract: TokenEchidna
        crytic-args: --hardhat-ignore-compile
```

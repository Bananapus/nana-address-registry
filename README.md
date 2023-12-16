# Juicebox Hook Registry

Provides an accessible function linking pay/redeem hooks with their corresponding deployer addresses.

This registry uses `create` and [`create2`](https://docs.soliditylang.org/en/v0.8.23/control-structures.html#salted-contract-creations-create2) to generate a deterministic address for a hook based on a deployer address and a nonce. That address is then used as a key to store the deployer's address.

This registry allows clients to easily and trustlessly check a given hook's deployer. This can be used to figure out whether a hook is "safe" or not, as determined by the client's developers.

*If you're having trouble understanding this contract, take a look at the [core Juicebox contracts](https://github.com/bananapus/juice-contracts-v4) and the [documentation](https://docs.juicebox.money/) first. If you have questions, reach out on [Discord](https://discord.com/invite/ErQYmth4dS).*

## Develop

`juice-hook-registry` uses the [Foundry](https://github.com/foundry-rs/foundry) development toolchain for builds, tests, and deployments. To get set up, install [Foundry](https://github.com/foundry-rs/foundry):

```bash
curl -L https://foundry.paradigm.xyz | sh
```

You can download and install dependencies with:

```bash
forge install
```

If you run into trouble with `forge install`, try using `git submodule update --init --recursive` to ensure that nested submodules have been properly initialized.

Some useful commands:

| Command               | Description                                         |
| --------------------- | --------------------------------------------------- |
| `forge install`       | Install the dependencies.                           |
| `forge build`         | Compile the contracts and write artifacts to `out`. |
| `forge fmt`           | Lint.                                               |
| `forge test`          | Run the tests.                                      |
| `forge build --sizes` | Get contract sizes.                                 |
| `forge coverage`      | Generate a test coverage report.                    |
| `foundryup`           | Update foundry. Run this periodically.              |
| `forge clean`         | Remove the build artifacts and cache directories.   |

To learn more, visit the [Foundry Book](https://book.getfoundry.sh/) docs.

We recommend using [Juan Blanco's solidity extension](https://marketplace.visualstudio.com/items?itemName=JuanBlanco.solidity) for VSCode.

## Flow

- After deploying a hook, any addresses can call `JBAddressRegistry.registerAddress(address deployer, uint256 nonce)` to add it to the registry. The registry will compute and store the corresponding hook address.
- Alternatively, `JBAddressRegistry.registerAddress(address deployer, bytes32 salt, bytes calldata bytecode)` will compute and store the hook deployed from a contract using `create2`.

The registry doesn't enforce `IERC165` or the implementation of any hook interfaces, meaning it could be used for any contract deployed with `create`/`create2`.

Clients can retrieve the nonce for the contract and an EOA using `provider.getTransactionCount(address)` from `ethers.js` or `web3.eth.getTransactionCount` from `web3.js` just *before* the hook's deployment. If registering a hook later on, clients may need to manually calculate the nonce.

The `create2` salt is determined by a given deployer's logic. The deployment bytecode can be retrieved offchain (from the deployment transaction) or onchain (with `abi.encodePacked(type(deployedContract).creationCode, abi.encode(constructorArguments))`).

This registry is the second iteration and will fall back to the previous version as needed when calling `deployerOf`.

## Risk

Malicious hooks have a token minting access. Clients should provide comprehensive information to project owners and users on the potential for unintended or adversarial behaviour, especially for unknown hooks.

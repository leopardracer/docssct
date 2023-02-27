---
layout: ../../layouts/MainLayout.astro
section: chainlinkFunctions
date: Last Modified
title: "Getting Started"
whatsnext:
  {
    "Try out the Chainlink Functions Tutorials": "/chainlink-functions/tutorials/",
    "Read the Concepts page to learn about the core concepts behind Chainlink Functions": "/chainlink-functions/resources/concepts/",
    "Read the Architecture to understand how Chainlink Functions operates": "/chainlink-functions/resources/architecture/",
  }
setup: |
  import ChainlinkFunctions from "@features/chainlink-functions/common/ChainlinkFunctions.astro"
---

Use the [Chainlink Functions Starter Kit](https://github.com/smartcontractkit/functions-hardhat-starter-kit) with [Hardhat](https://hardhat.org/) to set up your on-chain contracts, test your requests, and send your requests to be fulfilled by the Chainlink Functions Decentralized Oracle Network (DON). This guide shows you how to complete the following tasks to get started with Chainlink Functions:

- Set up your Web3 wallet and find your private key
- Install the required frameworks
- Configure the starter kit with your environment variables
- Simulate a Chainlink Functions request
- Set up a subscription on the Polygon Mumbai testnet
- Send a Chainlink Functions request to the DON on the Polygon Mumbai testnet

:::note[Request Access]
Chainlink Functions is currently in a limited BETA.
Apply [here](http://functions.chain.link/) to add your EVM account address to the Allow List.
:::

## Set up your environment

You must provide the private key from a testnet wallet to run the examples in this documentation. Install a Web3 wallet, configure [Node.js](https://nodejs.org/en/download/), clone the [Starter Kit](https://github.com/smartcontractkit/functions-hardhat-starter-kit.git), and configure a `.env` file with the required variables.

Install and configure your Web3 wallet:

1. [Install the MetaMask wallet](/getting-started/deploy-your-first-contract#install-and-fund-your-metamask-wallet) or other Ethereum Web3 wallet.

1. Set the network for your wallet to Polygon Mumbai.

1. Request testnet MATIC from the [Polygon Faucet](https://faucet.polygon.technology/).

1. Request testnet LINK from [faucets.chain.link/mumbai](https://faucets.chain.link/mumbai).

Install the required frameworks and dependencies:

1. [Install Node.js 18](https://nodejs.org/en/download/). Optionally, you can use the [nvm package](https://www.npmjs.com/package/nvm) to switch between Node.js versions with `nvm use 18`.

1. In a terminal, clone the [Chainlink Functions Starter Kit](https://github.com/smartcontractkit/functions-hardhat-starter-kit.git) repository and change directories.

   ```shell
   git clone https://github.com/smartcontractkit/functions-hardhat-starter-kit.git && \
   cd functions-hardhat-starter-kit/
   ```

1. Run `npm install` to install the dependencies.

   ```shell
   npm install
   ```

1. Configure a `.env` file with the basic variables that you need to send your requests to the Polygon Mumbai network.

   ```shell
   touch .env && open .env
   ```

   ```text
   MUMBAI_RPC_URL="https://rpc.ankr.com/polygon_mumbai"
   PRIVATE_KEY="<YOUR_PRIVATE_KEY>"
   ```

   - `MUMBAI_RPC_URL`: Set a URL for the Polygon Mumbai testnet. You can use the URL in the example config or sign up for a personal endpoint from [Alchemy](https://www.alchemy.com/), [Infura](https://www.infura.io/), or another node provider service.

   - `PRIVATE_KEY`: Find the private key for your testnet wallet. If you use MetaMask, follow the instructions to [Export a Private Key](https://metamask.zendesk.com/hc/en-us/articles/360015289632-How-to-export-an-account-s-private-key). Set this in the `.env` file. **Note**: The Chainlink Functions hardhat starter kit uses your private key to sign any transactions you make such as deploying your consumer contract, creating subscriptions, and making requests.

1. Compile the contracts in the repository. You will see several compile warnings, but no errors.

   ```shell
   npx hardhat compile
   ```

Simulate a request to test your environment and make sure everything is configured correctly. Run the `npx hardhat functions-simulate` command. The simulation runs on a local Hardhat network (a local Ethereum network node designed for development) and executes a request defined in the default [Functions-request-config.js](https://github.com/smartcontractkit/functions-hardhat-starter-kit/blob/main/Functions-request-config.js) file. The starter kit includes the capability to simulate transactions so you can quickly test your code before you send it to the DON.

```shell
npx hardhat functions-simulate
```

If the simulation is successful, the output includes the simulated on-chain response:

```text
__Simulated On-Chain Response__
Response returned to client contract represented as a hex string: 0x00000000000000000000000000000000000000000000000000000000000f50ed
Decoded as a uint256: 1003757

Gas used by sendRequest: 360602
Gas used by client callback function: 75029
```

## Configure your on-chain resources

After you configure your local environment, configure some on-chain resources to process your requests, receive the responses, and pay for the work done by the DON.

Deploy an on-chain [FunctionsConsumer.sol](https://github.com/smartcontractkit/functions-hardhat-starter-kit/blob/main/contracts/FunctionsConsumer.sol) contract, create a [subscription](/chainlink-functions/resources/subscriptions), and fund the subscription with LINK to pay for requests after they are fulfilled.

1. Deploy the consumer contract and record the contract address. For now, you can skip on-chain contract verification with the `--verify false` flag. If you do need to [verify](https://blog.chain.link/how-to-verify-a-smart-contract-on-etherscan) your contract on-chain, sign up for a [free Polygonscan API key](https://polygonscan.com/login) and include the key in your `.env` file under the `POLYGONSCAN_API_KEY=` variable.

   ```shell
   npx hardhat functions-deploy-client --network mumbai --verify false
   ```

   Find the contract address in the output:

   ```text
   FunctionsConsumer contract deployed to 0x97795d62905a66f5cf0384075c64c95795d443c3 on mumbai
   ```

1. Create a Chainlink Functions subscription and add your contract as an approved consumer contract. You can do this with a single transaction if you include your contract address with the `--contract` flag. For this example, 1 LINK is more than enough on the Mumbai testnet. You can always get more LINK from [faucets.chain.link](https://faucets.chain.link/mumbai) and later add it to the subscription. See the [Subscriptions Management](/chainlink-functions/resources/subscriptions#fund-a-subscription) page for details.

   ```shell
   npx hardhat functions-sub-create --network mumbai --amount 1 --contract YOUR_CONSUMER_CONTRACT_ADDRESS
   ```

   Record the subscription ID from the output.

   ```text
   Created subscription with ID: 7
   Owner: 0xc26881a0aF2ea1c7e8135e6ac47af756265432e8
   Balance: 1.0 LINK
   1 authorized consumer contract:
   [ '0x97795d62905a66f5cf0384075c64c95795d443c3' ]
   ```

## Run the example request

After your environment and on-chain resources are configured, you can send your request to the DON to execute the request defined in the [Functions-request-config.js](https://github.com/smartcontractkit/functions-hardhat-starter-kit/blob/main/Functions-request-config.js) file. Run `npx hardhat functions-request`. Specify your subscription ID with the `--subid` flag and specify your consumer contract address with the `--contract` flag.

```shell
npx hardhat functions-request --subid YOUR_SUBSCRIPTION_ID --contract YOUR_CONSUMER_CONTRACT_ADDRESS --network mumbai
```

The on-chain transaction and request will take longer to complete than the simulation. The output includes a decoded response:

```text
Transmission cost: 0.000078762053689013 LINK
Base fee: 0.0 LINK
Total cost: 0.000078762053689013 LINK

Request 0x7294990c570bd46a24ec488f73e088be56e45a2fa43fa761f161d26b32b7b435 fulfilled!
Response returned to client contract represented as a hex string: 0x00000000000000000000000000000000000000000000000000000000000f50ed
Decoded as a uint256: 1003757
```

If your request is successful, you have all the tools that you need to make your own custom requests. Edit the [Functions-request-config.js](https://github.com/smartcontractkit/functions-hardhat-starter-kit/blob/main/Functions-request-config.js) file and the [calculation-example.js](https://github.com/smartcontractkit/functions-hardhat-starter-kit/blob/main/calculation-example.js) source file with your own code. Then, run the `npx hardhat functions-request` command again to test your changes. You can re-use the same consumer contract and subscription for different requests.

Chainlink Functions is capable of much more than just computation. Try one of the [Tutorials](/chainlink-functions/tutorials/) to see examples that can GET and POST to public APIs, securely handle API secrets, handle custom responses, and query multiple APIs.
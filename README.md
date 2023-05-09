# Creating an Educational DAO on the Celo Blockchain with Solidity and Hardhat.md

## Introduction
Blockchain technology has opened up new possibilities for decentralized finance, and smart contracts play a crucial role in making this possible. In this tutorial, we will be creating a simple Educational DAO contract on the Celo blockchain using Solidity and the Hardhat development framework. This contract will allow users to contribute educational content and earn on the Celo blockchain.

### Learning Objective
By the end of this tutorial, you will be able to:
* Write and deploy a smart contract on the Celo blockchain using Solidity and Hardhat
* Interact with the deployed contract using the ContractKit library

### Tech Stack
* [Solidity](https://docs.soliditylang.org/en/v0.8.19/)
* [Hardhat](https://hardhat.org/hardhat-runner/docs/getting-started#overview)
* [ContractKit library](https://docs.celo.org/developer/contractkit)
* Celo blockchain (specifically, the Alfajores testnet)

### Prerequisites and Previous Knowledge
* Basic understanding of blockchain technology and smart contracts
* Familiarity with JavaScript and Node.js
* A text editor such as Visual Studio Code
* An understanding of how to use the command line interface (CLI)

### Time Required
This tutorial should take approximately 1-2 hours to complete, depending on your level of experience with blockchain development.

## Tutorial

### Step 1 - Create a new Hardhat project
First, you'll need to create a new Hardhat project. To do this, open a terminal and run the following commands:
```
mkdir Educational-dao
cd Educational-dao
npm init -y
npm install --save-dev hardhat
```
This will create a new directory called **Educational-dao**, initialize a new **package.json** file, and install the Hardhat development environment as a dev dependency.

### Step 2 - Install the necessary dependencies
Next, you'll need to install some additional dependencies that we'll be using to build our Educational DAO. In your terminal, run the following command:
````
npm install --save-dev @openzeppelin/contracts @celo/contractkit dotenv
````
This will install the OpenZeppelin Contracts library, the Celo ContractKit library, and the dotenv library for managing environment variables.

### Step 3 - Set up your .env file
Next, create a new file called **.env** in your project directory, and add the following lines:
````
MNEMONIC="<your mnemonic here>"
CELO_RPC="<your Celo RPC endpoint here>"
````
Replace **<your mnemonic here>** with your 12-word mnemonic phrase, and **<your Celo RPC endpoint here>** with the URL of your Celo RPC endpoint. This information can be obtained from your Celo wallet provider or node operator.
  
### Write your Educational DAO contract
Now it's time to write your Educational DAO contract! Open a new file called **EducationalDAO.sol** in the **contracts/** directory of your project, and add the following code:

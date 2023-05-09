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

``` bash
mkdir Educational-dao
cd Educational-dao
npm init -y
npm install --save-dev hardhat
```
This will create a new directory called **Educational-dao**, initialize a new **package.json** file, and install the Hardhat development environment as a dev dependency.

### Step 2 - Install the necessary dependencies
Next, you'll need to install some additional dependencies that we'll be using to build our Educational DAO. In your terminal, run the following command:

```` scss
npm install --save-dev @openzeppelin/contracts @celo/contractkit dotenv
````
This will install the OpenZeppelin Contracts library, the Celo ContractKit library, and the dotenv library for managing environment variables.

### Step 3 - Set up your .env file
Next, create a new file called **.env** in your project directory, and add the following lines:

```` makefile
MNEMONIC="<your mnemonic here>"
CELO_RPC="<your Celo RPC endpoint here>"
````
Replace **<your mnemonic here>** with your 12-word mnemonic phrase, and **<your Celo RPC endpoint here>** with the URL of your Celo RPC endpoint. This information can be obtained from your Celo wallet provider or node operator.
  
### Step 4 - Write your Educational DAO contract
Now it's time to write your Educational DAO contract! Open a new file called **EducationalDAO.sol** in the **contracts/** directory of your project, and add the following code:
  
``` sol
  // SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract EducationalDAO is Ownable {
    struct Content {
        string title;
        string description;
        string url;
        uint256 reward;
        bool approved;
        bool rejected;
    }

    IERC20 public celoToken;

    mapping(address => bool) public contributors;
    mapping(uint256 => Content) public contents;

    event ContentSubmitted(
        uint256 indexed id,
        string title,
        string description,
        string url,
        uint256 reward
    );

    event ContentApproved(uint256 indexed id);

    event ContentRejected(uint256 indexed id);

    constructor(address _celoTokenAddress) {
        celoToken = IERC20(_celoTokenAddress);
    }

    function submitContent(
        string memory _title,
        string memory _description,
        string memory _url,
        uint256 _reward
    ) external {
        require(contributors[msg.sender], "Not a contributor");

        uint256 id = uint256(keccak256(abi.encodePacked(_title, _description, _url)));
        contents[id] = Content({
            title: _title,
            description: _description,
            url: _url,
            reward: _reward,
            approved: false,
            rejected: false
        });

        emit ContentSubmitted(id, _title, _description, _url, _reward);
    }

    function approveContent(uint256 _id) external onlyOwner {
        Content storage content = contents[_id];
        require(!content.approved && !content.rejected, "Content already processed");

        content.approved = true;
        celoToken.transfer(msg.sender, content.reward);

        emit ContentApproved(_id);
   }

    function addContributor(address _contributor) external onlyOwner {
        contributors[_contributor] = true;
    }

    function removeContributor(address _contributor) external onlyOwner {
        contributors[_contributor] = false;
    }
}
````
  
This code defines a simple EducationalDAO contract that allows contributors to submit educational content to the DAO, which can then be approved or rejected by the DAO owner. The approved content is rewarded with Celo tokens. The contract also allows the DAO owner to add or remove contributors.
  
### Step 5 - Write a Hardhat test script
Now that we've written our EducationalDAO contract, let's write a test script to ensure that everything is working as expected. Open a new file called **EducationalDAOTest.js** in the **test/** directory of your project, and add the following code:

  ```` js
  const { expect } = require("chai");
const { ethers } = require("hardhat");
const { time } = require("@openzeppelin/test-helpers");

describe("EducationalDAO", function () {
  let dao;
  let celoToken;

  beforeEach(async function () {
    const CeloToken = await ethers.getContractFactory("MockCeloToken");
    celoToken = await CeloToken.deploy();

    const EducationalDAO = await ethers.getContractFactory("EducationalDAO");
    dao = await EducationalDAO.deploy(celoToken.address);

    await celoToken.mint(dao.address, ethers.utils.parseEther("1000000"));

    const [owner, contributor1, contributor2] = await ethers.getSigners();
    await dao.addContributor(contributor1.address);
    await dao.addContributor(contributor2.address);

    await celoToken.connect(contributor1).approve(dao.address, ethers.utils.parseEther("1000"));
    await celoToken.connect(contributor2).approve(dao.address, ethers.utils.parseEther("1000"));
  });

  it("should allow contributors to submit content", async function () {
    const [_, contributor1, contributor2] = await ethers.getSigners();

    await dao.connect(contributor1).submitContent("Title 1", "Description 1", "URL 1", ethers.utils.parseEther("100"));
    await dao.connect(contributor2).submitContent("Title 2", "Description 2", "URL 2", ethers.utils.parseEther("200"));

    expect(await dao.contents(0)).to.deep.equal({
      title: "Title 1",
      description: "Description 1",
      url: "URL 1",
      reward: ethers.utils.parseEther("100"),
      approved: false,
      rejected: false,
    });

    expect(await dao.contents(1)).to.deep.equal({
      title: "Title 2",
      description: "Description 2",
      url: "URL 2",
      reward: ethers.utils.parseEther("200"),
      approved: false,
      rejected: false,
    });
  });

    it("should allow the owner to approve content and reward contributors", async function () {
        const [owner, contributor1, contributor2] = await ethers.getSigners();

        await dao.connect(contributor1).submitContent("Title 1", "Description 1", "URL 1", ethers.utils.parseEther("100"));
        await dao.connect(contributor2).submitContent("Title 2", "Description 2", "URL 2", ethers.utils.parseEther("200"));

        // Wait for 2 days
        await time.increase(time.duration.days(2));

        await dao.approveContent(0);
        await dao.approveContent(1);

        expect(await dao.contents(0)).to.deep.equal({
            title: "Title 1",
            description: "Description 1",
            url: "URL 1",
            reward: ethers.utils.parseEther("100"),
            approved: true,
            rejected: false,
        });

        expect(await dao.contents(1)).to.deep.equal({
            title: "Title 2",
            description: "Description 2",
            url: "URL 2",
            reward: ethers.utils.parseEther("200"),
            approved: true,
            rejected: false,
        });

        const initialBalance1 = await celoToken.balanceOf(contributor1.address);
        const initialBalance2 = await celoToken.balanceOf(contributor2.address);

        await dao.rewardContentCreator(0);
        await dao.rewardContentCreator(1);

        expect(await celoToken.balanceOf(contributor1.address)).to.equal(initialBalance1.add(ethers.utils.parseEther("100")));
        expect(await celoToken.balanceOf(contributor2.address)).to.equal(initialBalance2.add(ethers.utils.parseEther("200")));
    });

    it("should allow the owner to reject content", async function () {
        const [owner, contributor1] = await ethers.getSigners();

        await dao.connect(contributor1).submitContent("Title 1", "Description 1", "URL 1", ethers.utils.parseEther("100"));

        await dao.rejectContent(0);

        expect(await dao.contents(0)).to.deep.equal({
            title: "Title 1",
            description: "Description 1",
            url: "URL 1",
            reward: ethers.utils.parseEther("100"),
            approved: false,
            rejected: true,
        });
    });

    it("should not allow non-contributors to submit content", async function () {
        const [_, nonContributor] = await ethers.getSigners();

        await expect(dao.connect(nonContributor).submitContent("Title", "Description", "URL", ethers.utils.parseEther("100"))).to.be.revertedWith("Not a contributor");
    });
});
  
````
This test script creates an instance of the EducationalDAO contract, mints some Celo tokens and adds some contributors. It then tests whether contributors can submit content, the owner can approve or reject content and reward contributors for approved content, and non-contributors are prevented from submitting content.
  
### Step 6 - Deploy the contract
To deploy our EducationalDAO contract, we will use the hardhat-deploy plugin. This plugin makes it easy to deploy contracts to the Celo blockchain. First, we need to install the plugin:
  
```` scss
  npm install --save-dev @nomiclabs/hardhat-ethers @nomiclabs/hardhat-waffle hardhat-deploy
````
  
Next, we need to configure Hardhat to use the hardhat-deploy plugin. In your hardhat.config.js file, add the following code:
  
```` js
 require("@nomiclabs/hardhat-waffle");
require("hardhat-deploy");

const { privateKey } = require('./secrets.json');

module.exports = {
  defaultNetwork: "hardhat",
  networks: {
    hardhat: {},
    alfajores: {
      url: "https://alfajores-forno.celo-testnet.org",
      accounts: [privateKey],
    },
    mainnet: {
      url: "https://forno.celo.org",
      accounts: [privateKey],
    },
  },
  solidity: {
    version: "0.8.4",
    settings: {
      optimizer: {
        enabled: true,
        runs: 200,
      },
    },
  },
};
 
````
  
This code configures Hardhat to use the **hardhat-deploy** plugin and sets up some networks for us to deploy our contract to. The privateKey variable should be set to the **private key** of your Celo account.

We also need to create a **secrets.json** file in the root directory of our project with the following contents:

```` json
{
  "privateKey": "<your-private-key>"
}
````
  
Replace **<your-private-key>** with the private key of your Celo account.

Now we can deploy our contract to the Alfajores testnet by running the following command:
  
```` 
  npx hardhat deploy --network alfajores
```` 
  
This will deploy the contract to the Alfajores testnet and print out the address of the deployed contract.
  
### Step 7 - Interact with the contract
  
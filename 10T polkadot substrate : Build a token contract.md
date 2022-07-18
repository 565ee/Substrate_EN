• [Setting up a project](#index1)  
• [Sample Hardhat project](#index2)  
• [Testing](#index3)  
• [External networks](#index4)  
• [Plugins and dependencies](#index5)  
• [hardhat Tutorials , hardhat 教程](#index98)   
• [Contact 联系方式](#index99)

# <span id='index1'>• Setting up a project</span>  
Hardhat projects are Node.js projects with the `hardhat` package installed and a `hardhat.config.js` file.

To initialize a Node.js project you can use [npm](https://docs.npmjs.com/cli/v8) or [yarn](https://classic.yarnpkg.com/). We recommend using npm 7 or later:
```
npm init -y
```

Then you need to install Hardhat:
```
npm install --save-dev hardhat
```

If you run `npx hardhat` now, you will be shown some options to facilitate project creation:

```
$ npx hardhat
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

Welcome to Hardhat v2.10.0

? What do you want to do? …
▸ Create a JavaScript project
  Create a TypeScript project
  Create an empty hardhat.config.js
  Quit
```

If you select _Create an empty hardhat.config.js_, Hardhat will create a `hardhat.config.js` like the following:

```
/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: "0.8.9",
};
```

And this is enough to run Hardhat using a default project structure.

# <span id='index2'>• Sample Hardhat project</span>  
If you select _Create a JavaScript project_, a simple project creation wizard will ask you some questions. After that, the wizard will create some directories and files and installthe necessary dependencies. The most important of these dependencies is the [Hardhat Toolbox](/hardhat-runner/plugins/nomicfoundation-hardhat-toolbox), a plugin that bundles all the things you need to start working with Hardhat.

The initialized project has the following structure:

```
contracts/
scripts/
test/
hardhat.config.js
```

These are the default paths for a Hardhat project.

- `contracts/` is where the source files for your contracts should be.
- `test/` is where your tests should go.
- `scripts/` is where simple automation scripts go.

If you need to change these paths, take a look at the [paths configuration section](../config/index.md#path-configuration).

# <span id='index3'>• Testing</span>  
When it comes to testing your contracts, the sample project comes with some useful functionality:

- The built-in [Hardhat Network](/hardhat-network/docs) as the development network to test on, along with the [Hardhat Network Helpers](/hardhat-network-helpers) library to manipulate this network.
- [Mocha](https://mochajs.org/) as the test runner, [Chai](https://chaijs.com/) as the assertion library, and the [Hardhat Chai Matchers](/hardhat-chai-matchers) to extend Chai with contracts-related functionality.
- The [`ethers.js`](https://docs.ethers.io/v5/) library to interact with the network and with contracts.

As well as other useful plugins. You can learn more about this in the [Testing contracts guide](./test-contracts.md).

# <span id='index4'>• External networks</span>  
If you need to use an external network, like an Ethereum testnet, mainnet or some other specific node software, you can set it up using the `networks` configuration entries in the exported object in `hardhat.config.js`, which is how Hardhat projects manage settings.

You can make use of the `--network` CLI parameter to quickly change the network.

Take a look at the [networks configuration section](../config/index.md#networks-configuration) to learn more about setting up different networks.

# <span id='index5'>• Plugins and dependencies</span>  
Most of Hardhat's functionality comes from plugins, so check out the [plugins section](/hardhat-runner/plugins) for the official list and see if there are any ones of interest to you.

To use a plugin, the first step is always to install it using npm or yarn, followed by requiring it in your config file:

```
require("@nomicfoundation/hardhat-toolbox");

module.exports = {
  solidity: "0.8.9",
};
```

Plugins are **essential** to Hardhat projects, so make sure to check out all the available ones and also build your own!

### Setting up your editor

[Hardhat for Visual Studio Code](/hardhat-vscode) is the official Hardhat extension that adds advanced support for Solidity to VSCode. If you use Visual Studio Code, give it a try!

# <span id='index98'>• hardhat Tutorials , hardhat 教程</span>  
CN 中文 Github  [hardhat 教程 : github.com/565ee/hardhat_CN](https://github.com/565ee/hardhat_CN)  
CN 中文 CSDN    [hardhat 教程 : blog.csdn.net/wx468116118](https://blog.csdn.net/wx468116118/category_11923442.html)  
EN 英文 Github  [hardhat Tutorials : github.com/565ee/hardhat_EN](https://github.com/565ee/hardhat_EN)  

# <span id='index99'>• Contact 联系方式</span>  
Homepage   : [565.ee](https://565.ee)  
GitHub     : [github.com/565ee](https://github.com/565ee)  
Email      : 565.eee@gmail.com  
Facebook   : [facebook.com/565.ee](https://facebook.com/565.ee)  
Twitter    : [twitter.com/565_eee](https://twitter.com/565_eee)  
Telegram   : [t.me/ee_565](https://t.me/ee_565)

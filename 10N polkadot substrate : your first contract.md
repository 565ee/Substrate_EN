• [introduce](#index1)  
• [Tutorial objectives](#index2)  
• [Update your Rust environment](#index3)  
• [Install the Substrate contracts node](#index4)  
• [Create a new smart contract project](#index5)  
• [Test the default contract](#index6)  
• [Build the contract](#index7)  
• [Start the Substrate smart contracts node](#index8)  
• [Deploy the contract](#index9)  
• [Create an instance on the blockchain](#index10)  
• [Call the smart contract](#index11)  
• [Substrate Tutorials , Substrate 教程](#index98)   
• [Contact 联系方式](#index99)

# <span id='index1'>• introduce</span>  
As you learned in Blockchain basics decentralized applications are most often written as smart contracts. Although Substrate is primarily a framework and toolkit for building custom blockchains, it can also provide a platform for smart contracts. This tutorial demonstrates how to build a basic smart contract to run on a Substrate-based chain. In this tutorial, you'll explore using ink! as a programming language for writing Rust-based smart contracts.

# <span id='index2'>• Tutorial objectives</span>  
By completing this tutorial, you will accomplish the following objectives:

Learn how to create a smart contract project.
Build and test a smart contract using the ink! smart contract language.
Deploy a smart contract on a local Substrate node.
Interact with a smart contract through a browser.

# <span id='index3'>• Update your Rust environment</span>  
For this tutorial, you need to add some Rust source code to your Substrate development environment.

To update your development environment:

• Open a terminal shell on your computer.
• Change to the root directory where you compiled the Substrate node template.
• Update your Rust environment by running the following command:
```
rustup component add rust-src --toolchain nightly
```

• Verify that you have the WebAssembly target installed by running the following command:
```
rustup target add wasm32-unknown-unknown --toolchain nightly
```

# <span id='index4'>• Install the Substrate contracts node</span>  
To simplify this tutorial, you can download a precompiled Substrate node for Linux or macOS. The precompiled binary includes the FRAME pallet for smart contracts by default. Alternatively, you can build the preconfigured contracts-node manually by running cargo install contracts-node on your local computer.

If you can't download the precompiled node, you can compile it locally with a command similar to the following:
```
cargo install contracts-node --git https://github.com/paritytech/substrate-contracts-node.git --tag <latest-tag> --force --locked
```

# <span id='index5'>• Create a new smart contract project</span>  
You are now ready to start developing a new smart contract project.

To generate the files for a smart contract project:

• Open a terminal shell on your computer.
• Create a new project folder named flipper by running the following command:
```
cargo contract new flipper
```

• Change to the new project folder by running the following command:
```
cd flipper/
```

# <span id='index6'>• Test the default contract</span>  
At the bottom of the lib.rs source code file, there are simple test cases to verify the functionality of the contract. You can test whether this code is functioning as expected using the offchain test environment.

To test the contract:

• Open a terminal shell on your computer, if needed.
• Verify that you are in the flipper project folder, if needed.
• Use the test subcommand and nightly toolchain to execute the default tests for the flipper contract by running the following command:
```
cargo +nightly test
```

The command should display output similar to the following to indicate successful test completion:
```
running 2 tests
test flipper::tests::it_works ... ok
test flipper::tests::default_works ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

# <span id='index7'>• Build the contract</span>  
After testing the default contract, you are ready to compile this project to WebAssembly.

To build the WebAssembly for this smart contract:

• Open a terminal shell on your computer, if needed.
• Verify that you are in the flipper project folder.
• Compile the flipper smart contract by running the following command:
```
cargo +nightly contract build
```

This command builds a WebAssembly binary for the flipper project, a metadata file that contains the contract Application Binary Interface (ABI), and a .contract file that you use to deploy the contract. For example, you should see output similar to the following:

# <span id='index8'>• Start the Substrate smart contracts node</span>  
If you have successfully installed substrate-contracts-node, you can start a local blockchain node for your smart contract.

To start the preconfigured contracts-node:

• Open a terminal shell on your computer, if needed.
• Start the contracts node in local development mode by running the following command:
```
substrate-contracts-node --dev
```

• Navigate to the Contracts UI in a web browser, then click Yes allow this application access.
• Select Local Node.

# <span id='index9'>• Deploy the contract</span>  
However, deploying a smart contract on Substrate is a little different than deploying on traditional smart contract platforms. For most smart contract platforms, you must deploy a completely new blob of the smart contract source code each time you make a change. For example, the standard ERC20 token has been deployed to Ethereum thousands of times. Even if a change is minimal or only affects some initial configuration setting, each change requires a full redeployment of the code. Each smart contract instance consume blockchain resources equivalent to the full contract source code, even if no code was actually changed.

In Substrate, the contract deployment process is split into two steps:

• Upload the contract code to the blockchain.
• Create an instance of the contract.

Upload the contract code
For this tutorial, you use the Contracts UI front-end to deploy the flipper contract on the Substrate chain.

To upload the smart contract source code:

• Open to the Contracts UI in a web browser.
• Verify that you are connected to the Local Node.
• Click Add New Contract.
• Click Upload New Contract Code.
• Select an Account to use to create a contract instance.

You can select any existing account, including a predefined account such as alice.

• Type a descriptive Name for the smart contract, for example, Flipper Contract.
• Browse and select or drag and drop the flipper.contract file that contains the bundled Wasm blob and metadata into the upload section.
![image](https://user-images.githubusercontent.com/28084126/178223805-fe470c80-934a-4f57-ac6e-6041136a1f69.png)

# <span id='index10'>• Create an instance on the blockchain</span>  
Smart contracts exist as an extension of the account system on the Substrate blockchain. When you create an instance of this smart contract, Substrate creates a new AccountId to store any balance managed by the smart contract and to allow you to interact with the contract.

After you upload the smart contract and click Next, the Contracts UI displays information about the content of the smart contract.

To create the instance:

• Review and accept the default Deployment Constructor options for the initial version of the smart contract.
• Review and accept the default Max Gas Allowed of 200000.
![image](https://user-images.githubusercontent.com/28084126/178224124-58da7b9a-161d-458c-8286-70b7e9b1dfcf.png)

• Click Next.

The transaction is now queued. If you needed to make changes, you could click Go Back to modify the input.
![image](https://user-images.githubusercontent.com/28084126/178224236-ad6a3e5b-d5f7-42cf-aa56-20fc11e4e3ed.png)

• Click Upload and Instantiate.

Depending on the account you used, you might be prompted for the account password. If you used a predefined account, you won't need to provide a password.
![image](https://user-images.githubusercontent.com/28084126/178224321-7c1860f3-8371-4013-9454-7b1d1700d9e8.png)

# <span id='index11'>• Call the smart contract</span>  
Now that your contract has been deployed on the blockchain, you can interact with it. The default flipper smart contract has two functions—flip() and get()—and you can use the Contracts UI to try them out.

get() function
You set the initial value of the flipper contract value to false when you instantiated the contract. You can use the get() function to verify the current value is false.

To test the get() function:

• • Select any account from the Account list.

• This contract doesn't place restrictions on who is allowed to send the get() request.

• Select get(): bool from the Message to Send list.
• Click Read.
• Verify that the value false is returned in the Call Results.
![image](https://user-images.githubusercontent.com/28084126/178225245-eff8ebe3-6e38-4e05-96e3-7514ae1bf61d.png)


flip() function
The flip() function changes the value from false to true.

To test the flip() function:

• • Select any predefined account from the Account list.

• The flip() function is a transaction that changes the chain state and requires an account with funds to be used to execute the call. Therefore, you should select an account that has a predefined account balance, such as the alice account.

• Select flip() from the Message to Send list.
• Click Call.
• Verify that the transaction is successful in the Call Results.
• Verify the new value is true in the Call Results.
![image](https://user-images.githubusercontent.com/28084126/178225529-235b7e4a-8312-439c-9de7-f41831a30662.png)

# <span id='index98'>• Substrate Tutorials , Substrate 教程</span>  
CN 中文 Github  [Substrate 教程 : github.com/565ee/Substrate_CN](https://github.com/565ee/Substrate_CN)  
CN 中文 CSDN    [Substrate 教程 : blog.csdn.net/wx468116118](https://blog.csdn.net/wx468116118/category_11846056.html)  
EN 英文 Github  [Substrate Tutorials : github.com/565ee/Substrate_EN](https://github.com/565ee/Substrate_EN)  
EN 英文 dev.to  [Substrate Tutorials : dev.to/565ee](https://dev.to/565ee/substrate-tutorials-5n4)  

# <span id='index99'>• Contact 联系方式</span>  
Homepage   : [565.ee](https://565.ee)  
GitHub     : [github.com/565ee](https://github.com/565ee)  
Email      : 565.eee@gmail.com  
Facebook   : [facebook.com/565.ee](https://facebook.com/565.ee)  
Twitter    : [twitter.com/565_eee](https://twitter.com/565_eee)  
Telegram   : [t.me/ee_565](https://t.me/ee_565)

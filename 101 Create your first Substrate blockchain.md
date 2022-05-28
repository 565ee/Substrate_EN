# Catalog    
• [What is Substrate?](#index1)  
• [Install Rust and the Rust toolchain](#index2)  
• [Prepare a Substrate node using the node template](#index3)  
• [Install the front-end template](#index4)  
• [Start the local Substrate node](#index5)  
• [Start the front-end template](#index6)  
• [Transfer funds from one account to another](#index7)  
• [Stop the local node](#index8)  

# <span id='index1'>• What is Substrate?</span>  
Substrate is an open source, modular, and extensible framework for building blockchains.

Substrate has been designed from the ground up to be flexible and allow innovators to design and build a blockchain network that meets their needs. It provides all the core components you need to build a customized blockchain node.

To get you started, the Substrate Developer Hub provides an out-of-the-box working Substrate-based node template. Without making any changes, you can use this node template to create a working blockchain network with some predefined user accounts and funds.

# <span id='index2'>• Install Rust and the Rust toolchain</span>  
To install and configure Rust manually:

1. Install rustup by running the following command:
```
curl https://sh.rustup.rs -sSf | sh
```

2. Configure your current shell to reload your PATH environment variable so that it includes the Cargo bin directory by running the following command:
```
source ~/.cargo/env
```

3. Configure the Rust toolchain to default to the latest stable version by running the following commands:
```
rustup default stable
rustup update
```

4. Add the nightly release and the nightly WebAssembly (wasm) targets by running the following commands:
```
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
```

5. Verify your installation by running the following commands:
```
rustc --version
rustup show
```

The previous steps walked you through the installation and configuration of Rust and the Rust toolchain so that you could see the full process for yourself.

# <span id='index3'>• Prepare a Substrate node using the node template</span>  
The Substrate node template provides a working development environment so that you can start building on Substrate right away.

To compile the Substrate node template:

1. Clone the node template repository using the version latest branch by running the following command:
```
git clone https://github.com/substrate-developer-hub/substrate-node-template
```

2. Change to the root of the node template directory by running the following command:
```
cd substrate-node-template
git checkout latest
```

3. Compile the node template by running the following command:
```
cargo build --release
```
You should always use the --release flag to build optimized artifacts.

# <span id='index4'>• Install the front-end template</span>  
1. Clone the front-end template repository by running the following command:
```
git clone https://github.com/substrate-developer-hub/substrate-front-end-template
```

2. Change to the root of the front-end template directory by running the following command:
```
cd substrate-front-end-template
git checkout latest
```

3. Install the dependencies for the front-end template by running the following command:
```
yarn install
```

# <span id='index5'>• Start the local Substrate node</span>  
1. Change to the root directory where you compiled the Substrate node template.
Start the node in development mode by running the following command:
```
./target/release/node-template --dev
```

2. Verify your node is up and running successfully by reviewing the output displayed in the terminal.

The terminal should display output similar to this:
![本地节点](https://user-images.githubusercontent.com/28084126/170838886-089b6d90-6b0e-4201-a465-e78aaa9e896e.png)

If the number after finalized is increasing, your blockchain is producing new blocks and reaching consensus about the state they describe.

We'll look into the details of what's reported in the log output in a later tutorial. For now, it's only important to know that your node is running and producing blocks.

3. Keep the terminal that displays the node output open to continue.

# <span id='index6'>• Start the front-end template</span>  
The Substrate front-end template consists of user interface components to enable you to interact with the Substrate node and perform a few common tasks.

To use the front-end template:

1. Open a new terminal shell on your computer, change to the root directory where you installed the front-end template.

2. Start the Front-end template by running the following command:
```
yarn start
```

3. Open http://localhost:8000 in a browser to view the front-end template.

The top section has an Account selection list for selecting the account to work with when you want to perform on-chain operations. The top section of the template also displays information about the chain to which you're connected.
![账号](https://user-images.githubusercontent.com/28084126/170839012-c0354af4-2d45-44cf-9918-a198cd022d69.png)

# <span id='index7'>• Transfer funds from one account to another</span>  
Now that you have a blockchain node running on your local computer and you have a front-end template available for performing on-chain operations, you are ready to explore different ways to interact with the blockchain.

By default, the front-end template includes several components that allow you to try different common tasks. For this tutorial, you can perform a simple transfer operation that moves funds from one account to another.

To transfer funds to an account:
![Transfer](https://user-images.githubusercontent.com/28084126/170839066-b1cfcc57-749f-405e-af21-8603580d1037.png)

# <span id='index8'>• Stop the local node</span>  
After a successful transfer, you can continue to explore the front-end template components or stop the local Substrate node. Because you specified the --dev option when you started the node, stopping the local node stops the blockchain and purges all persistent block data so that you can start with a clean state next time you start the node.

To stop the local Substrate node:

1. Return to the terminal shell where the node output is displayed.

2. Press Control-c to terminate the running process.

3. Verify your terminal returns to the terminal prompt in the substrate-node-template directory.

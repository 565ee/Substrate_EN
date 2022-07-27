• [introduce](#index1)  
• [Matching versions are critical](#index2)  
• [Build the relay chain node](#index3)  
• [Relay chain specification](#index4)  
• [Start your relay chain](#index5)  
• [Substrate Tutorials , Substrate 教程](#index98)   
• [Contact 联系方式](#index99)

# <span id='index1'>• introduce</span>  
In this tutorial, you will configure a local relay chain and connect a parachain template for use in a local testing environment.

Tutorial objectives
Identify software requirements.
Set up your parachain build environment.
Prepare a relay chain specification.
Start a relay chain locally.

# <span id='index2'>• Matching versions are critical</span>  
You must use the exact versions set forth in this tutorial. Parachains are very tightly coupled with the relay chain codebase that they connect to because they share so many common dependencies. Be sure to use the corresponding version of Polkadot with any other software when working on any examples throughout the Substrate documentation. You must stay synchronized with relay chain upgrades for your parachain to continue running successfully. If you don't keep up with relay chain upgrades, it's likely that your network will stop producing blocks.

All tutorials in the docs have been tested to work with:
Polkadot v0.9.24
Substrate Parachain Template polkadot-v0.9.24

# <span id='index3'>• Build the relay chain node</span>  
A slightly modified version of Polkadot's built in rococo-local network configuration will serve as the relay chain for this tutorial.
```
# Clone the Polkadot Repository, with correct version
git clone --depth 1 --branch release-v0.9.24 https://github.com/paritytech/polkadot.git

# Switch into the Polkadot directory
cd polkadot

# Build the relay chain Node
cargo b -r
```

Compiling the node can take 15 to 60 minuets to complete.
```
# Check if the help page prints to ensure the node is built correctly
./target/release/polkadot --help
```
If the help page is printed, you have succeeded in building a Polkadot node.

# <span id='index4'>• Relay chain specification</span>  
You will need a chain specification for your relay chain network. As we want to use a local testnet, a custom configuration may be needed to set development or custom keys for validators, boot node addresses, etc.

A relay chain must have one more validator nodes running than the total of connected parachain collators. For testing these typically must be hard coded into your chain specs. For example, if you want to connect two parachains with a single collator, run three or more relay chain validator nodes, and ensure they all are specified in your chain spec.

Whichever chain spec file you choose to use we will refer to the file simply as chain-spec.json in the instructions below. You will need to supply the proper path to the chain spec you are using.

This tutorial includes a sample chain specification file with two validator relay chain nodes—Alice and Bob—as authorities. You can use this sample chain specification without modification for a local test network and a single parachain. This is useful for registering a single parachain:

Plain rococo-local relay chain spec
https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/cumulus/chain-specs/rococo-custom-2-plain.json

Raw rococo-local relay chain spec
https://github.com/substrate-developer-hub/substrate-docs/blob/main/static/assets/tutorials/cumulus/chain-specs/rococo-custom-2-raw.json

You can read and edit the plain chain specification file. However, the chain specification file must be converged to the SCALE-encoded raw format before it can be used to start a node. For information about converting a chain specification to use the raw format, see the Customize a chain specification guide.

The sample chain specification is only valid for a single parachain with two validator nodes. If you add other validators, add additional parachains to your relay chain, or want to use custom non-development keys, you'll need to create a custom chain specification that fit your needs.

# <span id='index5'>• Start your relay chain</span>  
Before you can start block production for a parachains, you need to launch a relay chain for them to connect to. This section describes how to start both nodes using the two-validator raw chain spec. The steps are similar for starting additional nodes.

Start the alice validator
# Start Relay `Alice` node
```
./target/release/polkadot \
--alice \
--validator \
--base-path /tmp/relay/alice \
--chain <path to spec json> \
--port 30333 \
--ws-port 9944
```

The port (port) and websocket port (ws-port) specified in this command use default values and can be omitted. However, the values are included here as a reminder to always check these values. After the node starts, no other nodes on the same local machine can use these ports.
```
🏷 Local node identity is: 12D3KooWGjsmVmZCM1jPtVNp6hRbbkGBK3LADYNniJAKJ19NUYiq
```
When the node starts you will see several log messages, including the node's Peer ID. Take note of this, as you will need it when connecting other nodes to it.

Start the bob validator
The command to start the second node is similar to the command to start the first node with a few important differences.
```
./target/release/polkadot \
--bob \
--validator \
--base-path /tmp/relay-bob \
--chain <path to spec json> \
--bootnodes /ip4/<Alice IP>/tcp/30333/p2p/<Alice Peer ID> \
--port 30334 \
--ws-port 9945
```

Notice that this command uses a a different base path ( /tmp/relay-bob), validator key (--bob), and ports (30334 and 9945).

The command to start the second node also includes the --bootnodes command-line option to specify the IP address and peer identifier of the first node. The bootnodes option is not strictly necessary if you are running the entire network on a single local machine, but it is necessary when using a connection to a non-local network without any specified bootnodes in the chain spec, as is the case with the rococo-custom-2-plain.json example we are using.

For this tutorial, your final chain spec filename must start with rococo or the node will not know what runtime logic to include.• [introduce](#index1)  
• [Matching versions are critical](#index2)  
• [Build the relay chain node](#index3)  
• [Relay chain specification](#index4)  
• [Start your relay chain](#index5) 

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

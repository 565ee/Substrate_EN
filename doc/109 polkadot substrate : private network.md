• [introduce](#index1)  
• [Generate your own keys](#index2)  
• [Generate a second set of keys](#index3)  
• [Modify an existing chain specification](#index4)  
• [Convert the chain specification to use the raw format](#index5)  
• [Start the first node](#index6)  
• [Add keys to the keystore](#index7)  
• [Enable other participants to join](#index8)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)

# <span id='index1'>• introduce</span>  
This tutorial illustrates how you can start a small, standalone blockchain network with an authority set of private validators.

All blockchains require the nodes in the network to agree on the set of messages and their order to successfully create blocks and progress from one block to the next. Each block represents the state of data at a specific point in time and the nodes' agreement on the state is called consensus. There are several different algorithms used to reach consensus, including the following:

• Proof of work consensus depends on the computational work done by validator nodes to add valid blocks to the chain.

• Proof of stake consensus selects the validators to add valid blocks to the chain based on the cryptocurrency holdings that they have locked up as a stake in the network.

• Proof of authority consensus relies on a set of approved account identities to act as validators. The nodes associated with approved accounts have the authority to put transactions into blocks.

The Substrate node template uses a proof of authority consensus model also referred to as authority round or Aura consensus. The Aura consensus protocol limits block production to a rotating list of authorized accounts—authorities—that create blocks in a round robin fashion.

In this tutorial, you'll see how this consensus model works in practice, first, by using the predefined accounts that are part of the node template, then by adding a new authority to the approved set.

# <span id='index2'>• Generate your own keys</span>  
Now that you know how to start and connect running nodes as peers using command-line options, you are ready to generate your own secret keys instead of using the predefined account keys. It's important to remember that each participant in the blockchain network is responsible for generating unique keys.

You have already used some command-line options to start your local blockchain node using the predefined alice and bob accounts. You can also use command-line options to generate random keys to use with Substrate.

As a best practice, you should use an air-gapped computer that has never been connected to the internet when you generate keys for a production blockchain. At a minimum, you should disconnect from the internet before you generate any keys you intend to use on a public or private blockchain that is not under your control.

For this tutorial, however, you can use node template key subcommand to generate random keys locally and can remain connected to the internet.

• Open a terminal shell on your computer.

• Change to the root directory where you compiled the Substrate node template.

• Generate a random secret phrase and keys by running the following command:
```
./target/release/node-template key generate --scheme Sr25519 --password-interactive
```

• Type a password for the generated keys.

The command generates keys and displays output similar to the following:
```
Secret phrase:  pig giraffe ceiling enter weird liar orange decline behind total despair fly
Secret seed:       0x0087016ebbdcf03d1b7b2ad9a958e14a43f2351cd42f2f0a973771b90fb0112f
Public key (hex):  0x1a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
Account ID:        0x1a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
Public key (SS58): 5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW
SS58 Address:      5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW
```

• Use the secret phrase for the account you just generated to derive keys using the Ed25519 signature scheme.

For example, run a command similar to the following:
```
./target/release/node-template key inspect --password-interactive --scheme Ed25519 "pig giraffe ceiling enter weird liar orange decline behind total despair fly"
```

• Type the password you used to generate keys.

The command displays output similar to the following:
```
Secret phrase `pig giraffe ceiling enter weird liar orange decline behind total despair fly` is account:
Secret seed:       0x0087016ebbdcf03d1b7b2ad9a958e14a43f2351cd42f2f0a973771b90fb0112f
Public key (hex):  0x2577ba03f47cdbea161851d737e41200e471cd7a31a5c88242a527837efc1e7b
Public key (SS58): 5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN
Account ID:        0x2577ba03f47cdbea161851d737e41200e471cd7a31a5c88242a527837efc1e7b
SS58 Address:      5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN
```

# <span id='index3'>• Generate a second set of keys</span>  
• For this tutorial, the private network consists of just two nodes, so you need two sets of keys. You have a few options to continue the tutorial:

You can use the keys for one of the predefined accounts.

You can repeat the steps in the previous section using a different identity on your local computer to generate a second key pair.

You can derive a child key pair to simulate a second identity on your local computer.

You can recruit other participants to generate the keys required to join your private network.

• For illustration purposes, the second set of keys used in this tutorial are:

Sr25519: 5EJPj83tJuJtTVE2v7B9ehfM7jNT44CBFaPWicvBwYyUKBS6 for aura.
Ed25519: 5FeJQsfmbbJLTH1pvehBxrZrT5kHvJFj84ZaY5LK7NU87gZS for grandpa.

# <span id='index4'>• Modify an existing chain specification</span>  
After you generate the keys to use with your blockchain, you are ready to create a custom chain specification using those key pairs then share your custom chain specification with trusted network participants called validators.

To enable others to participate in your blockchain network, you should ensure that they generate their own keys. If other participants have generated their key pairs, you can create a custom chain specification to replace the local chain specification that you used previously.

This tutorial illustrates how to create a two-node network. You can follow the same steps to add more nodes to your network

Previously, you added nodes to the blockchain using the predefined local chain specification using the --chain local command-line option. Instead of writing a completely new chain specification, you can modify the one that you have used before.

To create a new chain specification based on an existing specification:

• Open a terminal shell on your computer.

• Change to the root directory where you compiled the Substrate node template.

• Export the local chain specification to a file named customSpec.json by running the following command:
```
./target/release/node-template build-spec --disable-default-bootnode --chain local > customSpec.json
```

• Preview the first few fields in the customSpec.json file by running the following command:
```
head customSpec.json
```
The command displays the first fields from the file. For example:
```
{
  "name": "Local Testnet",
  "id": "local_testnet",
  "chainType": "Local",
  "bootNodes": [],
  "telemetryEndpoints": null,
  "protocolId": null,
  "properties": null,
  "consensusEngine": null,
  "codeSubstitutes": {},
```

• Preview the last fields in the customSpec.json file by running the following command:
```
tail -n 80 customSpec.json
```
This command displays the last sections following the Wasm binary field, including the details for several of the pallets—such as the sudo and balances pallets—that are used in the runtime.

• Open the customSpec.json file in a text editor.

• Modify aura field to specify the nodes with the authority to create blocks by adding the Sr25519 SS58 address keys for each network participant.
```
"aura": {
    "authorities": [
      "5CfBuoHDvZ4fd8jkLQicNL8tgjnK8pVG9AiuJrsNrRAx6CNW",
      "5EJPj83tJuJtTVE2v7B9ehfM7jNT44CBFaPWicvBwYyUKBS6"
    ]
  },
```

• Modify the grandpa field to specify the nodes with the authority to finalize blocks by adding the Ed25519 SS58 address keys for each network participant.
```
"grandpa": {
    "authorities": [
      [
        "5CuqCGfwqhjGzSqz5mnq36tMe651mU9Ji8xQ4JRuUTvPcjVN",
        1
      ],
      [
        "5FeJQsfmbbJLTH1pvehBxrZrT5kHvJFj84ZaY5LK7NU87gZS",
        1
      ]
    ]
  },
```

# <span id='index5'>• Convert the chain specification to use the raw format</span>  
After you prepare a chain specification with the information you want to use, you must convert it into a raw specification before it can be used. The raw chain specification includes the same information as the unconverted specification. However, the raw chain specification also contains encoded storage keys that the node uses to reference the data in its local storage. Distributing a raw chain specification ensures that each node stores the data using the proper storage keys.

To convert a chain specification to use the raw format:

Open a terminal shell on your computer.

Change to the root directory where you compiled the Substrate node template.

Convert the customSpec.json chain specification to the raw format with the file name customSpecRaw.json by running the following command:

```
./target/release/node-template build-spec --chain=customSpec.json --raw --disable-default-bootnode > customSpecRaw.json
```

# <span id='index6'>• Start the first node</span>  
As the first participant in the private blockchain network, you are responsible for starting the first node, called the bootnode.

To start the first node:

• Open a terminal shell on your computer.

• Change to the root directory where you compiled the Substrate node template.

• Start the first node using the custom chain specification by running a command similar to the following:
```
./target/release/node-template \
--base-path /tmp/node01 \
--chain ./customSpecRaw.json \
--port 30333 \
--ws-port 9945 \
--rpc-port 9933 \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--validator \
--rpc-methods Unsafe \
--name MyNode01
```

# <span id='index7'>• Add keys to the keystore</span>  
After you start the first node, no blocks are yet produced. The next step is to add two types of keys to the keystore for each node in the network.

For each node:

Add the aura authority keys to enable block production.

Add the grandpa authority keys to enable block finalization.


To insert keys into the keystore:

• Open a terminal shell on your computer.

• Change to the root directory where you compiled the Substrate node template.

• Insert the aura secret key generated from the key subcommand by running a command similar to the following:
```
./target/release/node-template key insert --base-path /tmp/node01 \
--chain customSpecRaw.json \
--scheme Sr25519 \
--suri <your-secret-seed> \
--password-interactive \
--key-type aura
```

• Insert the grandpa secret key generated from the key subcommand by running a command similar to the following:
```
./target/release/node-template key insert --base-path /tmp/node01 \
--chain customSpecRaw.json \
--scheme Ed25519 \
--suri <your-secret-key> \
--password-interactive \
--key-type gran
```

• Verify that your keys are in the keystore for node01 by running the following command:
```
ls /tmp/node01/chains/local_testnet/keystore
```
The command displays output similar to the following:
```
617572611441ddcb22724420b87ee295c6d47c5adff0ce598c87d3c749b776ba9a647f04
6772616e1441ddcb22724420b87ee295c6d47c5adff0ce598c87d3c749b776ba9a647f04
```

# <span id='index8'>• Enable other participants to join</span>  
You can now allow other validators to join the network using the --bootnodes and --validator command-line options.

To add a second validator to the private network:

• Open a terminal shell on a second computer.

• Change to the root directory where you compiled the Substrate node template.

• Start a second blockchain node by running the following command:
```
./target/release/node-template \
--base-path /tmp/node02 \
--chain ./customSpecRaw.json \
--port 30334 \
--ws-port 9946 \
--rpc-port 9934 \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--validator \
--rpc-methods Unsafe \
--name MyNode02 \
--bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWLmrYDLoNTyTYtRdDyZLWDe1paxzxTw5RgjmHLfzW96SX \
--password-interactive
```

• Add the aura secret key generated from the key subcommand by running a command similar to the following:
```
./target/release/node-template key insert --base-path /tmp/node02 \
--chain customSpecRaw.json \
--scheme Sr25519 \
--suri <second-participant-secret-seed> \
--password-interactive \
--key-type aura
```

• Add the grandpa secret key generated from the key subcommand to the local keystore by running a command similar to the following:
```
./target/release/node-template key insert --base-path /tmp/node02 \
--chain customSpecRaw.json \
--scheme Ed25519 \
--suri <second-participant-secret-seed> \
--password-interactive \
--key-type gran
```

• Verify that your keys are in the keystore for node02 by running the following command:
```
ls /tmp/node02/chains/local_testnet/keystore
```
The command displays output similar to the following:
```
617572611a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
6772616e1a4cc824f6585859851f818e71ac63cf6fdc81018189809814677b2a4699cf45
```
Substrate nodes require a restart after inserting a grandpa key, so you must shut down and restart nodes before you see blocks being finalized.

• Shut down the node by pressing Control-c.

• Restart the second blockchain node by running the following command:
```
./target/release/node-template \
--base-path /tmp/node02 \
--chain ./customSpecRaw.json \
--port 30334 \
--ws-port 9946 \
--rpc-port 9934 \
--telemetry-url "wss://telemetry.polkadot.io/submit/ 0" \
--validator \
--rpc-methods Unsafe \
--name MyNode02 \
--bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWLmrYDLoNTyTYtRdDyZLWDe1paxzxTw5RgjmHLfzW96SX \
--password-interactive
```

After both nodes have added their keys to their respective keystores—located under /tmp/node01 and /tmp/node02—and been restarted, you should see the same genesis block and state root hashes.

You should also see that each node has one peer (1 peers), and they have produced a block proposal (best: #2 (0xe111…c084)). After a few seconds, you should see new blocks being finalized on both nodes.

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

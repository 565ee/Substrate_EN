• [Introduction](#index1)  
• [Learning outcomes](#index2)  
• [Build the node template](#index3)  
• [Add the node-authorization pallet](#index4)  
• [Add genesis storage for our pallet](#index5)  
• [Obtaining Node Keys and PeerIDs](#index6)  
• [Alice and Bob Start the Network](#index7)  
• [Add charlie Node](#index8)  
• [Add Dave as a Sub-Node to Charlie](#index9)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)  

# <span id='index1'>• Introduction</span>  
In this tutorial, you will learn how to build a permissioned network with Substrate by using the node-authorization pallet. This tutorial should take you about 1 hour to complete.

You are probably already familiar with public or permissionless blockchain, where everyone is free to join the network by running a node. In a permissioned network, only authorized nodes are allowed to perform specific activities, like validate blocks and propagate transactions. Some examples of where permissioned blockchains may be desired:

• Private (or consortium) networks
• Highly regulated data environments (healthcare, B2B ledgers, etc.)
• Testing pre-public networks at scale

Before you start, we expect that:

• You have completed the Build a PoE Decentralized Application Tutorial.
• You are conceptually familiar with P2P Networking in Substrate. We recommend completing the Private Network Tutorial to get experience with this first.

# <span id='index2'>• Learning outcomes</span>  
• Learn how to use the node-authorization pallet in your runtime
• Learn how to create a permissioned network consisting of multiple nodes

# <span id='index3'>• Build the node template</span>  
• Clone the node template.
```
git clone https://github.com/substrate-developer-hub/substrate-node-template
# We want to use the `latest` tag throughout all of this tutorial
git checkout latest
```

• Build the node template.
```
cd substrate-node-template/
cargo build --release
```

# <span id='index4'>• Add the node-authorization pallet</span>  
• First we must add the pallet to our runtime dependencies:
runtime/Cargo.toml

```
pallet-node-authorization = { default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-v0.9.23" }

#--snip--
[features]
default = ['std']
std = [
    #--snip--
    'pallet-node-authorization/std',
    #--snip--
]
```

• We need to simulate the governance in our simple blockchain, so we just let a sudo admin rule, configuring the pallet's interface to EnsureRoot. In a production environment we should want to have governance based checking implemented here. 
runtime/src/lib.rs

```

/* --snip-- */

use frame_system::EnsureRoot;

/* --snip-- */

parameter_types! {
	pub const MaxWellKnownNodes: u32 = 8;
	pub const MaxPeerIdLength: u32 = 128;
}

impl pallet_node_authorization::Config for Runtime {
	type Event = Event;
	type MaxWellKnownNodes = MaxWellKnownNodes;
	type MaxPeerIdLength = MaxPeerIdLength;
	type AddOrigin = EnsureRoot<AccountId>;
	type RemoveOrigin = EnsureRoot<AccountId>;
	type SwapOrigin = EnsureRoot<AccountId>;
	type ResetOrigin = EnsureRoot<AccountId>;
	type WeightInfo = ();
}

/* --snip-- */
```

• Finally, we are ready to put our pallet in construct_runtime macro with following extra line of code:
runtime/src/lib.rs

```
construct_runtime!(
	pub enum Runtime where
		Block = Block,
		NodeBlock = opaque::Block,
		UncheckedExtrinsic = UncheckedExtrinsic
	{
		/* --snip-- */
		NodeAuthorization: pallet_node_authorization, // <-- add this line
		/* --snip-- */
	}
);
```

# <span id='index5'>• Add genesis storage for our pallet</span>  
• PeerId is encoded in bs58 format, so we need a new library bs58 in node/Cargo.toml to decode it to get its bytes.
node/cargo.toml
```
[dependencies]
#--snip--
bs58 = "0.4.0"
#--snip--
```

• Now we add a proper genesis storage in node/src/chain_spec.rs. Similarly, import the necessary dependencies:
node/src/chain_spec.rs
```
/* --snip-- */
use sp_core::OpaquePeerId; // A struct wraps Vec<u8>, represents as our `PeerId`.
use node_template_runtime::NodeAuthorizationConfig; // The genesis config that serves for our pallet.
/* --snip-- */
```

• Adding our genesis config in the helper function testnet_genesis,
node/src/chain_spec.rs
```
/// Configure initial storage state for FRAME modules.
fn testnet_genesis(
	wasm_binary: &[u8],
	initial_authorities: Vec<(AuraId, GrandpaId)>,
	root_key: AccountId,
	endowed_accounts: Vec<AccountId>,
	_enable_println: bool,
) -> GenesisConfig {

		/* --snip-- */

	/*** Add This Block Item ***/
		node_authorization: NodeAuthorizationConfig {
			nodes: vec![
				(
					OpaquePeerId(bs58::decode("12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2").into_vec().unwrap()),
					endowed_accounts[0].clone()
				),
				(
					OpaquePeerId(bs58::decode("12D3KooWQYV9dGMFoRzNStwpXztXaBUjtPqi6aU76ZgUriHhKust").into_vec().unwrap()),
					endowed_accounts[1].clone()
				),
			],
		},

	/* --snip-- */

}
```

# <span id='index6'>• Obtaining Node Keys and PeerIDs</span>  
• For Alice's well known node:
```
# Node Key
c12b6d18942f5ee8528c8e2baf4e147b5c5c18710926ea492d09cbd9f6c9f82a

# Peer ID, generated from node key
12D3KooWBmAwcd4PJNJvfV89HwE48nwkRmAgo8Vy3uQEyNNHBox2

# BS58 decoded Peer ID in hex:
0x0024080112201ce5f00ef6e89374afb625f1ae4c1546d31234e87e3c3f51a62b91dd6bfa57df
```

• For Bob's well known node:
```
# Node Key
6ce3be907dbcabf20a9a5a60a712b4256a54196000a8ed4050d352bc113f8c58

# Peer ID, generated from node key
12D3KooWQYV9dGMFoRzNStwpXztXaBUjtPqi6aU76ZgUriHhKust

# BS58 decoded Peer ID in hex:
0x002408011220dacde7714d8551f674b8bb4b54239383c76a2b286fa436e93b2b7eb226bf4de7
```

• For Charlie's NOT well known node:
```
# Node Key
3a9d5b35b9fb4c42aafadeca046f6bf56107bd2579687f069b42646684b94d9e

# Peer ID, generated from node key
12D3KooWJvyP3VJYymTqG7eH4PM5rN4T2agk5cdNCfNymAqwqcvZ

# BS58 decoded Peer ID in hex:
0x002408011220876a7b4984f98006dc8d666e28b60de307309835d775e7755cc770328cdacf2e
```

• For Dave's sub-node
```
# Node Key
a99331ff4f0e0a0434a6263da0a5823ea3afcfffe590c9f3014e6cf620f2b19a

# Peer ID, generated from node key
12D3KooWPHWFrfaJzxPnqnAYAoRUyAHHKqACmEycGTVmeVhQYuZN

# BS58 decoded Peer ID in hex:
0x002408011220c81bc1d7057a1511eb9496f056f6f53cdfe0e14c8bd5ffca47c70a8d76c1326d
```

The nodes of Alice and Bob are already configured in genesis storage and serve as well known nodes. We will later add Charlie's node into the set of well known nodes. Finally we will add the connection between Charlie's node and Dave's node without making Dave's node as a well known node.

# <span id='index7'>• Alice and Bob Start the Network</span>  
• Let's start Alice's node first:
```
./target/release/node-template \
--chain=local \
--base-path /tmp/validator1 \
--alice \
--node-key=c12b6d18942f5ee8528c8e2baf4e147b5c5c18710926ea492d09cbd9f6c9f82a \
--port 30333 \
--ws-port 9944
```

• Start Bob's node:
```
# In a new terminal, leave Alice running
./target/release/node-template \
--chain=local \
--base-path /tmp/validator2 \
--bob \
--node-key=6ce3be907dbcabf20a9a5a60a712b4256a54196000a8ed4050d352bc113f8c58 \
--port 30334 \
--ws-port 9945
```

After both nodes are started, you should be able to see new blocks authored and finalized in bother terminal logs. Now let's use the polkadot.js apps and check the well known nodes of our blockchain. Don't forget to switch to one of our local nodes running: 127.0.0.1:9944 or 127.0.0.1:9945.

Then, let's go to Developer page, Chain State sub-tab, and check the data stored in the nodeAuthorization pallet, wellKnownNodes storage. You should be able to see the peer ids of Alice and Bob's nodes, prefixed with 0x to show its bytes in hex format.

We can also check the owner of one node by querying the storage owners with the peer id of the node as input, you should get the account address of the owner.

# <span id='index8'>• Add charlie Node</span>  
• Let's start Charlie's node
```
./target/release/node-template \
--chain=local \
--base-path /tmp/validator3 \
--name charlie  \
--node-key=3a9d5b35b9fb4c42aafadeca046f6bf56107bd2579687f069b42646684b94d9e \
--port 30335 \
--ws-port=9946 \
--offchain-worker always
```

• Go to Developer page, Sudo tab, in apps and submit the nodeAuthorization - add_well_known_node call with the peer id in hex of Charlie's node and the owner is Charlie, of course. Note Alice is the valid sudo origin for this call.
<img width="1257" alt="add_well_known_node" src="https://user-images.githubusercontent.com/28084126/173120725-3d749bd2-4e52-45c9-b18b-54594d91a220.png">

# <span id='index9'>• Add Dave as a Sub-Node to Charlie</span>  
• Let's add Dave's node, not as a well-known node, but a "sub-node" of Charlie. Dave will only be able to connect to Charlie to access the network. This is a security feature: as Charlie is therefore solely responsible for any connected sub-node peer. There is one point of access control for David in case they need to be removed or audited.

Start Dave's node with following command:
```
./target/release/node-template \
--chain=local \
--base-path /tmp/validator4 \
--name dave \
--node-key=a99331ff4f0e0a0434a6263da0a5823ea3afcfffe590c9f3014e6cf620f2b19a \
--port 30336 \
--ws-port 9947 \
--offchain-worker always
```

After it was started, there is no available connections. This is a permissioned network, so first, Charlie needs to configure his node to allow the connection from Dave's node.

In the Developer Extrinsics page, get Charlie to submit an addConnections extrinsic. The first PeerId is the peer id in hex of Charlie's node. The connections is a list of allowed peer ids for Charlie's node, here we only add Dave's.
<img width="1263" alt="charlie_add_connections" src="https://user-images.githubusercontent.com/28084126/173121156-735a540d-9e98-4dc7-900c-34777dfd9003.png">

Then, Dave needs to configure his node to allow the connection from Charlie's node. But before he adds this, Dave needs to claim his node, hopefully it's not too late!
<img width="1257" alt="dave_claim_node" src="https://user-images.githubusercontent.com/28084126/173121383-8e1ce7da-5c9d-4849-bbaf-3849d7916c71.png">

Similarly, Dave can add connection from Charlie's node.
<img width="1261" alt="dave_add_connections" src="https://user-images.githubusercontent.com/28084126/173121872-15a051de-4226-4117-8311-e53772cb72af.png">

You should now see Dave is catching up blocks and only has one peer which belongs to Charlie! Restart Dave's node in case it's not connecting with Charlie right away.

You are at the end of this tutorial and are already learned about how to build a permissioned network. You can also play with other dispatchable calls like remove_well_known_node, remove_connections.

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

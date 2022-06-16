• [Introduction](#index1)  
• [Learning outcomes](#index2)  
• [Start the Template Node](#index3)  
• [Use sudo to dispatch](#index4)  
• [Add the Scheduler Pallet](#index5)  
• [Build the Upgraded Runtime](#index6)  
• [Prepare an Upgraded Runtime](#index7)  
• [Upgrade the Runtime](#index8)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)  

# <span id='index1'>• Introduction</span>  
One of the defining features of the Substrate blockchain development framework is its support for forkless runtime upgrades. Forkless upgrades are a means of enhancing a blockchain runtime in a way that is supported and protected by the capabilities of the blockchain itself. A blockchain's runtime defines the state the blockchain can hold and also defines the logic for effecting changes to that state.
![node-diagram](https://user-images.githubusercontent.com/28084126/174053349-8f22cbb5-6dca-44aa-9e53-c0237e37785b.png)

Substrate makes it possible to deploy enhanced runtime capabilities (including breaking changes(!)) without a hard fork. Because the definition of the runtime is itself an element in a Substrate chain's state, network participants may update this value by way of an extrinsic, specifically the set_code function. Since updates to runtime state are bound by the blockchain's consensus mechanisms and cryptographic guarantees, network participants can use the blockchain itself to trustlessly distribute updated or extended runtime logic without needing to fork the chain or even release a new blockchain client.

This tutorial will use the Substrate Developer Hub node template to explore two mechanisms for forkless upgrades of FRAME-based runtimes. First, the sudo_unchecked_weight function from the Sudo pallet will be used to perform an upgrade that adds the Scheduler pallet. Then, the schedule function from the Scheduler pallet will be used to perform an upgrade that increases the existential (minimum) balance for network accounts.

# <span id='index2'>• Learning outcomes</span>  
• Understand some basic governance features in Substrate chains
• Gain experience with how a Substrate chain can upgrade it's runtime
• Learn how to use the scheduler pallet in your runtime

# <span id='index3'>• Start the Template Node</span>  
Since forkless runtime upgrades do not require network participants to restart their blockchain clients, the first step of this tutorial is to start the template node as-is. Build and start the unmodified node template.
```
cargo run --release -- --dev
```
By default, the well-known Alice account is configured as the holder of the Sudo pallet's key in the development_config function of the template node's chain specification file - this is the configuration that is used when the node is launched with the --dev flag. This means that Alice's account will be the one used to perform runtime upgrades throughout this tutorial.

# <span id='index4'>• Use sudo to dispatch</span>  
As the name of the Sudo pallet implies, it provides capabilities related to the management of a single sudo ("superuser do") administrator. In FRAME, the Root Origin is used to identify the runtime administrator; some of FRAME's features, including the ability to update the runtime by way of the set_code function, are only accessible to this administrator. The Sudo pallet maintains a single storage item: the ID of the account that has access to the pallet's dispatchable functions. The Sudo pallet's sudo function allows the holder of this account to invoke a dispatchable as the Root origin.

The following pseudo-code demonstrates how this is achieved, refer to the Sudo pallet's source code to learn more.
```
fn sudo(origin, call) -> Result {
	// Ensure caller is the account identified by the administrator key
	let sender = ensure_signed(origin)?;
	ensure!(sender == Self::key(), Error::RequireSudo);

	// Dispatch the specified call as the Root origin
	let res = call.dispatch(Origin::Root);
	Ok()
}
```

# <span id='index5'>• Add the Scheduler Pallet</span>  
Because the template node doesn't come with the Scheduler pallet included in its runtime, the first runtime upgrade performed in this tutorial will add that pallet. First, add the Scheduler pallet as a dependency in the template node's runtime Cargo file.

runtime/Cargo.toml
```
pallet-scheduler = { default-features = false, git = "https://github.com/paritytech/substrate.git", branch = "polkadot-v0.9.23" }

#--snip--

[features]
default = ['std']
std = [
    #--snip--
    'pallet-scheduler/std',
    #--snip--
]
```

The final step to preparing an upgraded FRAME runtime is to increment its spec_version, which is a member of the RuntimeVersion struct:

runtime/src/lib.rs
```
pub const VERSION: RuntimeVersion = RuntimeVersion {
	spec_name: create_runtime_str!("node-template"),
	impl_name: create_runtime_str!("node-template"),
	authoring_version: 1,
	spec_version: 101,  // *Increment* this value, the template uses 100 as a base
	impl_version: 1,
	apis: RUNTIME_API_VERSIONS,
	transaction_version: 1,
};
```

# <span id='index6'>• Build the Upgraded Runtime</span>  
```
cargo build --release -p node-template-runtime
```
Here the --release flag will result in a longer compile time, but also generate a smaller build artifact that is better suited for submitting to the blockchain network: storage minimization and optimizations are critical for any blockchain.

As we are only building the runtime, Cargo looks in runtime cargo.toml file for requirements and only executes these. Notice the runtime/build.rs file that cargo looks for builds the Wasm of your runtime that is specified in runtime/src/lib.rs.

When the --release flag is specified, build artifacts are output to the target/release directory; when the flag is omitted they will be sent to target/debug

open the Polkadot JS Apps UI and automatically configure the UI to connect to the local node.
Use Alice's account to invoke the sudoUncheckedWeight function and use the setCode function from the system pallet as its parameter. In order to supply the build artifact that was generated by the previous build step, toggle the "file upload" switch on the right-hand side of the "code" input field for the parameter to the setCode function. Click the "code" input field, and select the Wasm binary that defines the upgraded runtime: target/release/wbuild/node-template-runtime/node_template_runtime.compact.wasm. Leave the value for the _weight parameter at the default of 0. Click "Submit Transaction" and then "Sign and Submit".
![sudo-upgrade](https://user-images.githubusercontent.com/28084126/174057535-c260250e-a8c9-458b-bf74-6ac6c0a002e0.png)

After the transaction has been included in a block, the version number in the upper-left-hand corner of Polkadot JS Apps UI should reflect that the runtime version is now 101.

If you still see your node producing blocks in the terminal it's running and reported on the UI, you have performed a successful forkless runtime upgrade! Congrats!!!

# <span id='index7'>• Prepare an Upgraded Runtime</span>  
This upgrade is more straightforward than the previous one and only requires updating a single value in runtime/src/lib.rs aside from the runtime's spec_version.

```
pub const VERSION: RuntimeVersion = RuntimeVersion {
	spec_name: create_runtime_str!("node-template"),
	impl_name: create_runtime_str!("node-template"),
	authoring_version: 1,
	spec_version: 102,  // *Increment* this value.
	impl_version: 1,
	apis: RUNTIME_API_VERSIONS,
	transaction_version: 1,
};

/*** snip ***/

parameter_types! {
	pub const ExistentialDeposit: u128 = 1000;  // Update this value.
	pub const MaxLocks: u32 = 50;
}

/*** snip ***/

```

This change increases the value of the Balances pallet's ExistentialDeposit - the minimum balance needed to keep an account alive from the point-of-view of the Balances pallet.

```
cargo build --release -p node-template-runtime
```
This will override any previous build artifacts! So if you want to have a copy on hand of your last runtime Wasm build files, be sure to copy them somewhere else."

# <span id='index8'>• Upgrade the Runtime</span>  
In the previous section, the Scheduler pallet was configured with the Root origin as its ScheduleOrigin, which means that the sudo function (not sudo_unchecked_weight) can be used to invoke the schedule function. Use this link to open the Polkadot JS Apps UI's Sudo tab: https://polkadot.js.org/apps/#/sudo?rpc=ws://127.0.0.1:9944.

Wait until all the other fields have been filled in before providing the when parameter. Leave the maybe_periodic parameter empty and the priority parameter at its default value of 0. Select the System pallet's set_code function as the call parameter and provide the Wasm binary as before. Leave the "with weight override" option deactivated. Once all the other fields have been filled in, use a block number about 10 blocks (1 minute) in the future to fill in the when parameter and quickly submit the transaction.
![scheduled-upgrade](https://user-images.githubusercontent.com/28084126/174058307-2996f385-1b0f-4c53-b30a-525d8a907e7c.png)

After the target block has been included in the chain, the version number in the upper-left-hand corner of Polkadot JS Apps UI should reflect that the runtime version is now 102.

You can then observe the specific changes that were made in the upgrade by using the Polkadot JS Apps UI Chain State app to query the existentialDeposit constant value from the Balances pallet.

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

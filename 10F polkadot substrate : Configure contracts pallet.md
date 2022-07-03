• [introduce](#index1)  
• [Add the pallet dependencies](#index2)  
• [Implement the Contracts configuration trait](#index3)  
• [Expose the Contracts API](#index4)  
• [Update the outer node](#index5)  
• [Start the local Substrate node](#index6)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)

# <span id='index1'>• introduce</span>  
If you completed the Build a local blockchain tutorial, you already know that the Substrate node template provides a working runtime that includes some pallets to get you started. In Add a pallet to the runtime, you learned the basic common steps for adding a new pallet to the runtime. However, each pallet requires you to configure specific parameters and types. To see what that entails, this tutorial demonstrates adding a more complex pallet to the runtime. In this tutorial, you'll add the Contracts pallet so that you can support smart contract development for your blockchain.

# <span id='index2'>• Add the pallet dependencies</span>  
Any time you add a pallet to the runtime, you need to import the appropriate crate and update the dependencies for the runtime. For the Contracts pallet, you need to import the pallet-contracts crate.

• Open the runtime/Cargo.toml configuration file in a text editor.

• Import the pallet-contracts crate to make it available to the node template runtime by adding it to the list of dependencies.
```
[dependencies.pallet-contracts]
default-features = false
git = 'https://github.com/paritytech/substrate.git'
tag = 'latest'
version = '4.0.0-dev'

[dependencies.pallet-contracts-primitives]
default-features = false
git = 'https://github.com/paritytech/substrate.git'
tag = 'latest'
version = '4.0.0-dev'
```

• Add the Contracts pallet to the list of std features so that its features are included when the runtime is built as a native Rust binary.

```
[features]
default = ['std']
std = [
	#--snip--
	'pallet-contracts/std',
	'pallet-contracts-primitives/std',
	#--snip--
 ]
```

• Save your changes and close the runtime/Cargo.toml file.

# <span id='index3'>• Implement the Contracts configuration trait</span>  
Now that you have successfully imported the Contracts pallet crate, you are ready to add it to the runtime. If you have explored other tutorials, you might already know that every pallet has a configuration trait—called Config—that the runtime must implement.

To implement the Config trait for the Contracts pallet in the runtime:

• Open the runtime/src/lib.rs file in a text editor.

• Locate the pub use frame_support block and add Nothing to the list of traits.
```
pub use frame_support::{
	construct_runtime, parameter_types,
	traits::{KeyOwnerProofSystem, Randomness, StorageInfo, Nothing},
	weights::{
	constants::{BlockExecutionWeight, ExtrinsicBaseWeight, RocksDbWeight, WEIGHT_PER_SECOND},
	IdentityFee, Weight,
},
StorageValue,
};
```

• Add the WeightInfo property to the runtime.
```
/* After this line */
use pallet_transaction_payment::CurrencyAdapter;
/*** Add this line ***/
use pallet_contracts::weights::WeightInfo;
```

• Add the constants required by the Contracts pallet to the runtime.
```
/* After this block */
// Time is measured by number of blocks.
pub const MINUTES: BlockNumber = 60_000 / (MILLISECS_PER_BLOCK as BlockNumber);
pub const HOURS: BlockNumber = MINUTES * 60;
pub const DAYS: BlockNumber = HOURS * 24;

/* Add this block */
// Contracts price units.
pub const MILLICENTS: Balance = 1_000_000_000;
pub const CENTS: Balance = 1_000 * MILLICENTS;
pub const DOLLARS: Balance = 100 * CENTS;

const fn deposit(items: u32, bytes: u32) -> Balance {
	items as Balance * 15 * CENTS + (bytes as Balance) * 6 * CENTS
}

/// Assume ~10% of the block weight is consumed by `on_initialize` handlers.
/// This is used to limit the maximal weight of a single extrinsic.
const AVERAGE_ON_INITIALIZE_RATIO: Perbill = Perbill::from_percent(10);
/*** End Added Block ***/
```

• Add the parameter types and implementation for the Config trait to the runtime.
```
impl pallet_timestamp::Config for Runtime {
/* --snip-- */
}

/*** Add this block ***/
parameter_types! {
	pub TombstoneDeposit: Balance = deposit(
		1,
		<pallet_contracts::Pallet<Runtime>>::contract_info_size()
	);
	pub DepositPerContract: Balance = TombstoneDeposit::get();
	pub const DepositPerStorageByte: Balance = deposit(0, 1);
	pub const DepositPerStorageItem: Balance = deposit(1, 0);
	pub RentFraction: Perbill = Perbill::from_rational(1u32, 30 * DAYS);
	pub const SurchargeReward: Balance = 150 * MILLICENTS;
	pub const SignedClaimHandicap: u32 = 2;
	pub const MaxValueSize: u32 = 16 * 1024;
	// The lazy deletion runs inside on_initialize.
	pub DeletionWeightLimit: Weight = AVERAGE_ON_INITIALIZE_RATIO *
	 BlockWeights::get().max_block;
	// The weight needed for decoding the queue should be less or equal than a fifth
	// of the overall weight dedicated to the lazy deletion.
	pub DeletionQueueDepth: u32 = ((DeletionWeightLimit::get() / (
		<Runtime as pallet_contracts::Config>::WeightInfo::on_initialize_per_queue_item(1) -
		<Runtime as pallet_contracts::Config>::WeightInfo::on_initialize_per_queue_item(0)
	 )) / 5) as u32;

	pub Schedule: pallet_contracts::Schedule<Runtime> = Default::default();
}

impl pallet_contracts::Config for Runtime {
	type Time = Timestamp;
	type Randomness = RandomnessCollectiveFlip;
	type Currency = Balances;
	type Event = Event;
	type RentPayment = ();
	type SignedClaimHandicap = SignedClaimHandicap;
	type TombstoneDeposit = TombstoneDeposit;
	type DepositPerContract = DepositPerContract;
	type DepositPerStorageByte = DepositPerStorageByte;
	type DepositPerStorageItem = DepositPerStorageItem;
	type RentFraction = RentFraction;
	type SurchargeReward = SurchargeReward;
	type WeightPrice = pallet_transaction_payment::Module<Self>;
	type WeightInfo = pallet_contracts::weights::SubstrateWeight<Self>;
	type ChainExtension = ();
	type DeletionQueueDepth = DeletionQueueDepth;
	type DeletionWeightLimit = DeletionWeightLimit;
	type Call = Call;
	/// The safest default is to allow no calls at all.
	///
	/// Runtimes should whitelist dispatchables that are allowed to be called from contracts
	/// and make sure they are stable. Dispatchables exposed to contracts are not allowed to
	/// change because that would break already deployed contracts. The `Call` structure itself
	/// is not allowed to change the indices of existing pallets, too.
	type CallFilter = Nothing;
	type Schedule = Schedule;
	type CallStack = [pallet_contracts::Frame<Self>; 31];
}
/*** End added block ***/
```

• Add the types exposed in the Contracts pallet to the construct_runtime! macro.
```
construct_runtime!(
	pub enum Runtime where
	 Block = Block,
	 NodeBlock = opaque::Block,
	 UncheckedExtrinsic = UncheckedExtrinsic
	{
	 /* --snip-- */
	 /*** Add this ine ***/
	 Contracts: pallet_contracts::{Pallet, Call, Storage, Event<T>},
	}
);
```

• Save your changes and close the runtime/src/lib.rs file.

• Check that your runtime compiles correctly by running the following command:
```
cargo check -p node-template-runtime
```
Although the runtime should compile, you cannot yet compile the entire node.

# <span id='index4'>• Expose the Contracts API</span>  
Some pallets, including the Contracts pallet, expose custom runtime APIs and RPC endpoints. You are not required to enable the RPC calls on the Contracts pallet to use it on chain. However, it is useful to expose the APIs and endpoints for the Contracts pallet because doing so enables you to perform the following tasks:

Read contract state from off chain.

Make calls to node storage without making a transaction.

To expose the Contracts RPC API:

• Open the runtime/Cargo.toml file in a text editor and add a dependencies section to import the Contracts RPC endpoints runtime API.
```
[dependencies.pallet-contracts-rpc-runtime-api]
default-features = false
git = 'https://github.com/paritytech/substrate.git'
tag = 'latest'
version = '4.0.0-dev'
```

• Add the Contracts RPC API to the list of std features so that its features are included when the runtime is built as a native Rust binary.
```
[features]
default = ['std']
std = [
	#--snip--
	'pallet-contracts-rpc-runtime-api/std',
]
```

• Save your changes and close the runtime/Cargo.toml file.

• Open the runtime/src/lib.rs file and implement the contracts runtime API in the impl_runtime_apis! macro near the end of the runtime lib.rs file.
```
impl_runtime_apis! {
	/* --snip-- */
	/*** Add this block ***/
	impl pallet_contracts_rpc_runtime_api::ContractsApi<Block, AccountId, Balance, BlockNumber, Hash>
	for Runtime {
	 fn call(
			origin: AccountId,
			dest: AccountId,
			value: Balance,
			gas_limit: u64,
			input_data: Vec<u8>,
	 ) -> pallet_contracts_primitives::ContractExecResult {
			let debug = true;
			Contracts::bare_call(origin, dest, value, gas_limit, input_data, debug)
	}

		fn instantiate(
			origin: AccountId,
			endowment: Balance,
			gas_limit: u64,
			code: pallet_contracts_primitives::Code<Hash>,
			data: Vec<u8>,
			salt: Vec<u8>,
	 ) -> pallet_contracts_primitives::ContractInstantiateResult<AccountId, BlockNumber> {
			let compute_rent_projection = true;
			let debug = true;
			Contracts::bare_instantiate(origin, endowment, gas_limit, code, data, salt, compute_rent_projection, debug)
	 }

	 fn get_storage(
			address: AccountId,
			key: [u8; 32],
	 ) -> pallet_contracts_primitives::GetStorageResult {
			Contracts::get_storage(address, key)
	 }

	 fn rent_projection(
			address: AccountId,
	 ) -> pallet_contracts_primitives::RentProjectionResult<BlockNumber> {
			Contracts::rent_projection(address)
	 }
	}
	/*** End added block ***/
}
```

• Save your changes and close the runtime/src/lib.rs file.

• Check that your runtime compiles correctly by running the following command:
```
cargo check -p node-template-runtime
```

# <span id='index5'>• Update the outer node</span>  
At this point, you have finished adding the Contracts pallet to the runtime. Now, you need to consider whether the outer node requires any corresponding updates. For the Contracts pallet to take advantage of the RPC endpoint API, you need to add the custom RPC endpoint to the node configuration.

To add the RPC API extension to the outer node:

Open the node/Cargo.toml file and add the dependencies sections to import the Contracts and Contracts RPC crates.
```
[dependencies]
jsonrpc-core = '15.1.0'
structopt = '0.3.8'
#--snip--
# *** Add the following lines ***

[dependencies.pallet-contracts]
git = 'https://github.com/paritytech/substrate.git'
tag = 'latest'
version = '4.0.0-dev'

[dependencies.pallet-contracts-rpc]
git = 'https://github.com/paritytech/substrate.git'
tag = 'latest'
version = '4.0.0-dev'
```

• Open the node/src/rpc.rs file in a text editor.

Substrate provides an RPC to interact with the node. However, it does not contain access to the Contracts pallet by default. To interact with the Contracts pallet, you must extend the existing RPC and add the Contracts pallet and its API.
```
use node_template_runtime::{opaque::Block, AccountId, Balance, Index, BlockNumber, Hash}; // NOTE THIS IS AN ADJUSTMENT TO AN EXISTING LINE
use pallet_contracts_rpc::{Contracts, ContractsApi};
   /* --snip-- */

/// Instantiate all full RPC extensions.
pub fn create_full<C, P>(
   deps: FullDeps<C, P>,
) -> jsonrpc_core::IoHandler<sc_rpc::Metadata> where
   /* --snip-- */
   C: Send + Sync + 'static,
   C::Api: substrate_frame_rpc_system::AccountNonceApi<Block, AccountId, Index>,
   /*** Add This Line ***/
   C::Api: pallet_contracts_rpc::ContractsRuntimeApi<Block, AccountId, Balance, BlockNumber, Hash>,
   /* --snip-- */
{
   /* --snip-- */
   io.extend_with(
		TransactionPaymentApi::to_delegate(TransactionPayment::new(client.clone()))
   );
   /*** Add this block ***/
   // Contracts RPC API extension
   io.extend_with(
		ContractsApi::to_delegate(Contracts::new(client.clone()))
   );
   /*** End added block ***/
   io
}
```

• Save your changes and close the node/src/rpc.rs file.

• Check that your runtime compiles correctly by running the following command:
```
cargo check -p node-template-runtime
```

• Compile the node in release mode by running the following command:
```
cargo build --release
```

# <span id='index6'>• Start the local Substrate node</span>  
After your node compiles, you are ready to start the Substrate node that has been enhanced with smart contract capabilities from the Contracts pallet and interact with it using the front-end template.

To start the local node:

• Open a terminal shell, if necessary.

• Change to the root directory of the Substrate node template.

• Start the node in development mode by running the following command:
```
./target/release/node-template --dev
```

• In the new terminal, change to the root directory where you installed the front-end template.

• Start the web server for the front-end template by running the following command:
```
yarn start
```

• Open http://localhost:8000/ in a browser to view the front-end template.

• In the Pallet Interactor component, verify that Extrinsic is selected.

• Select contracts from the list of pallets available to call.
![image](https://user-images.githubusercontent.com/28084126/177032033-3368be4f-a608-41c5-9471-124796add60f.png)

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

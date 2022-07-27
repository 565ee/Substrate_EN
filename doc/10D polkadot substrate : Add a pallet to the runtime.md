• [introduce](#index1)  
• [Tutorial objectives](#index2)  
• [Add the Nicks pallet dependencies](#index3)  
• [Review the configuration trait for the pallet](#index4)  
• [Implement the Config trait for the pallet](#index5)  
• [Start the blockchain node](#index6)  
• [Start the front-end template](#index7)  
• [Set a nickname using the Nicks pallet](#index8)  
• [Query information for an account using the Nicks pallet](#index9)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)

# <span id='index1'>• introduce</span>  
As you saw Build a local blockchain, the Substrate node template provides a working runtime that includes some default FRAME development modules—pallets—to get you started building a custom blockchain.

This tutorial introduces the basic steps for adding a new pallet to the runtime for the node template. The steps are similar any time you want to add a new FRAME pallet to the runtime. However, each pallet requires specific configuration settings—for example, the specific parameters and types required to perform the functions that the pallet implements. For this tutorial, you'll add the Nicks pallet to the runtime for the node template, so you'll see how to configure the settings that are specific to the Nicks pallet. The Nicks pallet allows blockchain users to pay a deposit to reserve a nickname for an account they control. It implements the following functions:

The set_name function to collect a deposit and set the name of an account if the name is not already taken.
The clear_name function to remove the name associated with an account and return the deposit.
The kill_name function to forcibly remove an account name without returning the deposit.

# <span id='index2'>• Tutorial objectives</span>  
By completing this tutorial, you will use the Nicks pallet to accomplish the following objectives:

• Learn how to update runtime dependencies to include a new pallet.

• Learn how to configure a pallet-specific Rust trait.

• See changes to the runtime by interacting with the new pallet using the front-end template.

# <span id='index3'>• Add the Nicks pallet dependencies</span>  
Before you can use a new pallet, you must add some information about it to the configuration file that the compiler uses to build the runtime binary.

For Rust programs, you use the Cargo.toml file to define the configuration settings and dependencies that determine what gets compiled in the resulting binary. Because the Substrate runtime compiles to both a native Rust binary that includes standard library functions and a WebAssembly (Wasm) binary that does not include the standard library, the Cargo.toml file controls two important pieces of information:

The pallets to be imported as dependencies for the runtime, including the location and version of the pallets to import.
The features in each pallet that should be enabled when compiling the native Rust binary. By enabling the standard (std) feature set from each pallet, you can compile the runtime to include functions, types, and primitives that would otherwise be missing when you build the WebAssembly binary.

To add the dependencies for the Nicks pallet to the runtime:

• Open a terminal shell and change to the root directory for the node template.

• Open the runtime/Cargo.toml configuration file in a text editor.

• Import the pallet-nicks crate to make it available to the node template runtime by adding it to the list of dependencies.
```
[dependencies.pallet-nicks]
default-features = false
git = 'https://github.com/paritytech/substrate.git'
tag = 'monthly-2021-10'
version = '4.0.0-dev'
```

• Add the pallet-nicks/std features to the list of features to enable when compiling the runtime.
```
[features]
default = ['std']
std = [
  ...
  'pallet-aura/std',
  'pallet-balances/std',
  'pallet-nicks/std',    # add this line
  ...
]
```

• Check that the new dependencies resolve correctly by running the following command:
```
cargo check -p node-template-runtime
```

# <span id='index4'>• Review the configuration trait for the pallet</span>  
Every pallet has a Rust trait called Config. The Config trait is used to identify the parameters and types that the pallet needs to carry out its functions.

Most of the pallet-specific code required to add a pallet is implemented using the Config trait. You can review what you to need to implement for any pallet by referring to its Rust documentation or the source code for the pallet. For example, to see what you need to implement for the nicks pallet, you can refer to the Rust documentation for pallet_nicks::Config or the trait definition in the Nicks pallet source code.

For this tutorial, you can see that the Config trait in the nicks pallet declares the following types:
```
pub trait Config: frame_system::Config {
	/// The overarching event type.
	type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;

	/// The currency trait.
	type Currency: ReservableCurrency<Self::AccountId>;

	/// Reservation fee.
	#[pallet::constant]
	type ReservationFee: Get<BalanceOf<Self>>;

	/// What to do with slashed funds.
	type Slashed: OnUnbalanced<NegativeImbalanceOf<Self>>;

	/// The origin account that can forcibly set or remove a name. Root can always do this.
	type ForceOrigin: EnsureOrigin<Self::Origin>;

	/// The minimum length for a name.
	#[pallet::constant]
	type MinLength: Get<u32>;

	/// The maximum length for a name.
	#[pallet::constant]
	type MaxLength: Get<u32>;
}
```

After you identify the types your pallet requires, you need to add code to the runtime to implement the Config trait. To learn how to implement the Config trait for a pallet, you can use the Balances pallet—which is already implemented in the node template runtime—as an example.

To review the Config trait for the Balances pallet:

• Open the runtime/src/lib.rs file in a text editor.

• Locate the Balances pallet section.

• Note that the implementation for the Balances pallet consists of two parts:
The parameter_types! block where constant values are defined.
```
parameter_types! {
	// The u128 constant value 500 is aliased to a type named ExistentialDeposit.
	pub const ExistentialDeposit: u128 = 500;
	// A heuristic that is used for weight estimation.
	pub const MaxLocks: u32 = 50;
}
```

The impl block where the types and values defined by the Config interface are configured.
```
impl pallet_balances::Config for Runtime {
	// The previously defined parameter_type is used as a configuration parameter.
	type MaxLocks = MaxLocks;
	// The "Balance" that appears after the equal sign is an alias for the u128 type.
	type Balance = Balance;
	// The empty value, (), is used to specify a no-op callback function.
	type DustRemoval = ();
	// The previously defined parameter_type is used as a configuration parameter.
	type ExistentialDeposit = ExistentialDeposit;
	// The FRAME runtime system is used to track the accounts that hold balances.
	type AccountStore = System;
	// Weight information is supplied to the Balances pallet by the node template runtime.
	// type WeightInfo = (); // old way
	type WeightInfo = pallet_balances::weights::SubstrateWeight<Runtime>;
	// The ubiquitous event type.
	type Event = Event;
}
```

# <span id='index5'>• Implement the Config trait for the pallet</span>  
Now that you have seen an example of how the Config trait is implemented for the Balances pallet, you're ready to implement the Config trait for the Nicks pallet.

To implement the nicks pallet in your runtime:

• Open the runtime/src/lib.rs file in a text editor.

• Locate the last line of the Balances code block.

• Add the following code block for the Nicks pallet:
```
/// Add this code block to your template for Nicks:
parameter_types! {
	// Choose a fee that incentivizes desireable behavior.
	pub const NickReservationFee: u128 = 100;
	pub const MinNickLength: u32 = 8;
	// Maximum bounds on storage are important to secure your chain.
	pub const MaxNickLength: u32 = 32;
}

impl pallet_nicks::Config for Runtime {
	// The Balances pallet implements the ReservableCurrency trait.
	// `Balances` is defined in `construct_runtime!` macro. See below.
	// https://paritytech.github.io/substrate/master/pallet_balances/index.html#implementations-2
	type Currency = Balances;

	// Use the NickReservationFee from the parameter_types block.
	type ReservationFee = NickReservationFee;

	// No action is taken when deposits are forfeited.
	type Slashed = ();

	// Configure the FRAME System Root origin as the Nick pallet admin.
	// https://paritytech.github.io/substrate/master/frame_system/enum.RawOrigin.html#variant.Root
	type ForceOrigin = frame_system::EnsureRoot<AccountId>;

	// Use the MinNickLength from the parameter_types block.
	type MinLength = MinNickLength;

	// Use the MaxNickLength from the parameter_types block.
	type MaxLength = MaxNickLength;

	// The ubiquitous event type.
	type Event = Event;
}
```

• Add Nicks to the construct_runtime! macro.
```
construct_runtime!(
pub enum Runtime where
    Block = Block,
    NodeBlock = opaque::Block,
    UncheckedExtrinsic = UncheckedExtrinsic
  {
    /* --snip-- */
    Balances: pallet_balances::{Pallet, Call, Storage, Config<T>, Event<T>},

    /*** Add This Line ***/
    Nicks: pallet_nicks::{Pallet, Call, Storage, Event<T>},
  }
);
```

• Check that the new dependencies resolve correctly by running the following command:
```
cargo check -p node-template-runtime
```

• Compile the node in release mode by running the following command:
```
cargo build --release
```

# <span id='index6'>• Start the blockchain node</span>  
After your node compiles, you are ready to start the node that has been enhanced with nickname capabilities from the Nicks pallet and interact with it using the front-end template.

To start the local Substrate node:

• Open a terminal shell, if necessary.

• Change to the root directory of the Substrate node template.

• Start the node in development mode by running the following command:
```
./target/release/node-template --dev
```
In this case, the --dev option specifies that the node runs in developer mode using the predefined development chain specification. By default, this option also deletes all active data—such as keys, the blockchain database, and networking information—when you stop the node by pressing Control-c. Using the --dev option ensures that you have a clean working state any time you stop and restart the node.

• Verify your node is up and running successfully by reviewing the output displayed in the terminal.

If the number after finalized is increasing in the console output, your blockchain is producing new blocks and reaching consensus about the state they describe.

• Keep the terminal that displays the node output open to continue.

# <span id='index7'>• Start the front-end template</span>  
Now that you have added a new pallet to your runtime, you can use the Substrate front-end template to interact with the node template and access the Nicks pallet.

To start the front-end template:

• Open a new terminal shell on your computer.

• In the new terminal, change to the root directory where you installed the front-end template.

• Start the web server for the front-end template by running the following command:
```
yarn start
```
• Open http://localhost:8000/ in a browser to view the front-end template.

# <span id='index8'>• Set a nickname using the Nicks pallet</span>  
After you start the front-end template, you can use it to interact with the Nicks pallet you just added to the runtime.

To set a nickname for an account:

Check the account selection list to verify that the Alice account is currently selected.

In the Pallet Interactor component, verify that Extrinsic is selected.

Select nicks from the list of pallets available to call.

Select the setName dispatchable as the function to call from the nicks pallet.

Type a name that is longer than the MinNickLength (8 characters) and no longer than the MaxNickLength (32 characters).

Click Signed to execute the function.
![image](https://user-images.githubusercontent.com/28084126/176255805-f174b9bf-f7ea-4a5c-8f1b-fef11be39a1c.png)

Set a name

Observe the status of the call and the events emitted by the Nicks pallet.

# <span id='index9'>• Query information for an account using the Nicks pallet</span>  
Next, you can use Query capability to read the value of Alice's nickname from the runtime storage for the Nicks pallet.

To return the information stored for Alice:

• In the Pallet Interactor component, select Query.

• Select nicks from the list of pallets available to query.

• Select the nameOf.

• Copy and paste the address for the alice account in the Account field, then click Query.
![image](https://user-images.githubusercontent.com/28084126/176256222-9ad62806-7848-4949-a6e9-d9fd83e49451.png)
The return type is a tuple that contains two values:

The hex-encoded nickname for the Alice account.

The amount that was reserved from Alice's account to secure the nickname.

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

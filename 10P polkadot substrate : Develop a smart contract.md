• [introduce](#index1)  
• [Create a new smart contract project](#index2)  
• [Store simple values](#index3)  
• [Constructors](#index4)  
• [Add a function to get a storage value](#index5)  
• [Add a function to modify the storage value](#index6)  
• [Build the WebAssembly for the contract](#index7)  
• [Deploy and test the smart contract](#index8)  
• [Substrate Tutorials , Substrate 教程](#index98)   
• [Contact 联系方式](#index99)

# <span id='index1'>• introduce</span>  
In Prepare your first contract, you learned the basic steps for building and deploying a smart contract on a Substrate-based blockchain using a default first project. For this tutorial, you'll develop a new smart contract that increments a counter value each time you execute a function call.

By completing this tutorial, you will accomplish the following objectives:

Learn how to use a smart contract template.
Store simple values using a smart contract.
Increment and retrieve stored values using a smart contract.
Add public and private functions to a smart contract.

# <span id='index2'>• Create a new smart contract project</span>  
Smart contracts that run on Substrate start as projects. You create projects using cargo contract commands.

For this tutorial, you'll create a new project for the incrementer smart contract. Creating a new project adds a new project directory and default starter files—also called template files—to the project directory. You will modify these starter template files to build the smart contract logic for the incrementer project.

To create a new project for your smart contract:

• Open a terminal shell on your local computer, if you don’t already have one open.
• Create a new project named incrementer by running the following command:
```
cargo contract new incrementer
```

• Change to the new project directory by running the following command:
```
cd incrementer/
```

• Open the lib.rs file in a text editor.

By default, the template lib.rs file contains the source code for the flipper smart contract with instances of the flipper contract name renamed incrementer.

• Replace the default template source code with new incrementer source code.
• Save the changes to the lib.rs file, then close the file.
• Open the Cargo.toml file in a text editor and review the dependencies for the contract.
• Save changes to the Cargo.toml file, then close the file.
• Verify that the program compiles and passes the trivial test by running the following command:
```
cargo +nightly test
```

• Verify that you can build the WebAssembly for the contract by running the following command:
```
cargo +nightly contract build
```
If the program compiles successfully, you are ready to start programming.

# <span id='index3'>• Store simple values</span>  
Now that you have some starter source code for the incrementer smart contract, you can introduce some new functionality. For example, this smart contract requires storage of simple values. The following code illustrates how to store simple values for this contract using the #[ink(storage)] attribute macro:
```
#[ink(storage)]
pub struct MyContract {
	// Store a bool
	my_bool: bool,
	// Store a number
	my_number: u32,
}
```

# <span id='index4'>• Constructors</span>  
Every ink! smart contract must have at least one constructor that runs when the contract is created. However, a smart contract can have multiple constructors, if needed. The following code illustrates using multiple constructors:
```
use ink_lang as ink;

#[ink::contract]
mod mycontract {

	#[ink(storage)]
	pub struct MyContract {
		number: u32,
	}

	impl MyContract {
		/// Constructor that initializes the `u32` value to the given `init_value`.
		#[ink(constructor)]
		pub fn new(init_value: u32) -> Self {
			Self {
				number: init_value,
			}
		}

		/// Constructor that initializes the `u32` value to the `u32` default.
		///
		/// Constructors can delegate to other constructors.
		#[ink(constructor)]
		pub fn default() -> Self {
			Self {
				number: Default::default(),
			}
		}
	/* --snip-- */
	}
}
```

# <span id='index5'>• Add a function to get a storage value</span>  
Now that you have created and initialized a storage value, you can interact with it using public and private functions. For this tutorial, you add a public function to get a storage value. Note that all public functions must use the #[ink(message)] attribute macro.

To add the public function to the smart contract:

• Open the lib.rs file in a text editor.
• Update the get public function to return the data for the value storage item that has the i32 data type.
```
#[ink(message)]
pub fn get(&self) -> i32 {
   self.value
   }
}
```

• Replace the Test Your Contract comment in the private default_works function with code to test the get function.
```
fn default_works() {
   let contract = Incrementer::default();
   assert_eq!(contract.get(), 0);
}
```

• Save your changes and close the file.
• Use the test subcommand and nightly toolchain to test your work by running the following command:
```
cargo +nightly test
```

# <span id='index6'>• Add a function to modify the storage value</span>  
At this point, the smart contract does not allow users modify the storage. To enable users to modify storage items, you must explicitly mark value as a mutable variable.

To add a function for incrementing the stored value:

• Open the lib.rs file in a text editor.
• Add a new inc public function to increment the value stored using the by parameter that has data type of i32.
```
#[ink(message)]
pub fn inc(&mut self, by: i32) {
   self.value += by;
   }
}
```

• Add a new test to the source code to verify this function.
```
#[ink::test]
   fn it_works() {
       let mut contract = Incrementer::new(42);
       assert_eq!(contract.get(), 42);
       contract.inc(5);
       assert_eq!(contract.get(), 47);
       contract.inc(-50);
       assert_eq!(contract.get(), -3);
}
```

• Save your changes and close the file.
• Use the test subcommand and nightly toolchain to test your work by running the following command:
```
cargo +nightly test
```

# <span id='index7'>• Build the WebAssembly for the contract</span>  
After you test the incrementer contract, you are ready to compile this project to WebAssembly. After you compile the smart contract to WebAssembly, you can use the Contracts UI to deploy and test the smart contract on your local contracts node.

To build the WebAssembly for this smart contract:

• Open a terminal shell on your computer, if needed.
• Verify that you are in the incrementer project folder.
• Compile the incrementer smart contract by running the following command:
```
cargo +nightly contract build
```

# <span id='index8'>• Deploy and test the smart contract</span>  
If you have the substrate-contracts-node node installed locally, you can start a local blockchain node for your smart contract, then use the Contracts UI to deploy and test the smart contract.

To deploy on the local node:

• Open a terminal shell on your computer, if needed.
• Start the contracts node in local development mode by running the following command:
```
substrate-contracts-node --dev
```

• • Open the Contracts UI and verify that it is connected to the local node.
• Click Add New Contract.
• Click Upload New Contract Code.
• Select the incrementer.contract file, then click Next.
• Click Upload and Instantiate.
• Explore and interact with the smart contract using the Contracts UI.

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

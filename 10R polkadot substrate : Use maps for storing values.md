• [introduce](#index1)  
• [Initialize a mapping](#index2)  
• [Identifying the contract caller](#index3)  
• [Using the contract caller](#index4)  
• [Add mapping to the smart contract](#index5)  
• [Insert, update, or remove values](#index6)  
• [Substrate Tutorials , Substrate 教程](#index98)   
• [Contact 联系方式](#index99)

# <span id='index1'>• introduce</span>  
In Develop a smart contract, you developed a smart contract for storing and retrieving a single numeric value. This tutorial illustrates how you can extend the functionality of your smart contract to manage one number per user. To add this functionality, you'll use the Mapping type.

the following code illustrates mapping a user to a number:
```
#[ink(storage)]
pub struct MyContract {
	// Store a mapping from AccountIds to a u32
	my_number_map: ink_storage::Mapping<AccountId, u32>,
}
```

With the Mapping data type, you can store a unique instance of the storage value for each key. For this tutorial, each AccountId represents a key that maps to one and only one stored numeric my_value. Each user can only store, increment, and retrieve the value associated with his or her own AccountId.

# <span id='index2'>• Initialize a mapping</span>  
The first step is to initialize the mapping between an AccountId and a stored value. You must always initialize a mapping before you use it in your contract to avoid mapping errors and inconsistencies. To initialize a mapping, you need to do the following:

Add the SpreadAllocate trait on the storage structure.
• • Specify the mapping key and the value mapped to it.
• Call the ink_lang::utils::initalize_contract function to initialize the mapping for the contract.
The following example illustrates how to initialize a mapping and retrieve a value:
```
#![cfg_attr(not(feature = "std"), no_std)]

use ink_lang as ink;

#[ink::contract]
mod mycontract {
    use ink_storage::traits::SpreadAllocate;

    #[ink(storage)]
    #[derive(SpreadAllocate)]
    pub struct MyContract {
        // Store a mapping from AccountIds to a u32
        map: ink_storage::Mapping<AccountId, u32>,
    }

    impl MyContract {
        #[ink(constructor)]
        pub fn new(count: u32) -> Self {
            // This call is required to correctly initialize the
            // Mapping of the contract.
            ink_lang::utils::initialize_contract(|contract: &mut Self| {
                let caller = Self::env().caller();
                contract.map.insert(&caller, &count);
            })
        }

        #[ink(constructor)]
        pub fn default() -> Self {
            ink_lang::utils::initialize_contract(|_| {})
        }

        // Get the number associated with the caller's AccountId, if it exists
        #[ink(message)]
        pub fn get(&self) -> u32 {
            let caller = Self::env().caller();
            self.map.get(&caller).unwrap_or_default()
        }
    }
}
```

# <span id='index3'>• Identifying the contract caller</span>  
In the preceding example, you might have noticed the self.env().caller() function call. This function is available throughout the contract logic and always returns the contract caller. It is important to note that the contract caller is not the same as the origin caller. If a user accesses a contract that then calls a subsequent contract, the self.env().caller() in the second contract is the address of the first contract, not the original user.

# <span id='index4'>• Using the contract caller</span>  
There are many scenarios where having the contract caller available is useful. For example, you can use self.env().caller() to create an access control layer that only allows users to access their own values. You can also use self.env().caller() to save the contract owner during contract deployment. For example:
```
#![cfg_attr(not(feature = "std"), no_std)]

use ink_lang as ink;

#[ink::contract]
mod mycontract {

	#[ink(storage)]
	pub struct MyContract {
		// Store a contract owner
		owner: AccountId,
	}

	impl MyContract {
		#[ink(constructor)]
		pub fn new() -> Self {
			Self {
				owner: Self::env().caller();
			}
		}
		/* --snip-- */
	}
}
```

# <span id='index5'>• Add mapping to the smart contract</span>  
You are now ready to introduce a storage map to the incrementer contract.

To add a storage map to the incrementer contract:
• Open a terminal shell on your computer, if needed.
• Verify that you are in the incrementer project folder.
• Open the lib.rs file in a text editor.
• Import the SpreadAllocate trait and derive it for your contract.
```
#[ink::contract
mod incrementer {
   use ink_storage::traits::SpreadAllocate;

   #[ink(storage)]
   #[derive(SpreadAllocate)]
```

• Add the mapping key from AccountId to the i32 data type stored as my_value.
```
pub struct Incrementer {
   value: i32,
   my_value: ink_storage::Mapping<AccountId, i32>,
}
```

• Use the initialize_contract function to set an initial value and my_value for the new function in the contract.
```
#[ink(constructor)]
pub fn new(init_value: i32) -> Self {
   ink_lang::utils::initialize_contract(|contract: &mut Self| {
       contract.value = init_value;
       let caller = Self::env().caller();
       contract.my_value.insert(&caller, &0);
   })
}
```

• Use the initialize_contract function to set an initial value for the default function in the contract.
#[ink(constructor)]
```
#[ink(constructor)]
pub fn default() -> Self {
   ink_lang::utils::initialize_contract(|contract: &mut Self| {
       contract.value = Default::default();
   })
}
```

• Save your changes and close the file.
• Use the test subcommand and nightly toolchain to test your work by running the following command:
```
cargo +nightly test
```

# <span id='index6'>• Insert, update, or remove values</span>  
The final step in the Incrementer contract is to allow users to update their own values. You can use calls to the Mapping API to provide this functionality in the smart contract.

The ink_storage Mapping provides direct access to storage items. For example, you can replace a previous value held for a storage item by calling Mapping::insert() with an existing key. You can also update values by first reading them from storage using Mapping::get(), then update the value with Mapping::insert(). If there is no existing value at a given key, Mapping::get() returns None.

Because the Mapping API provides direct access to storage, you can use the Mapping::remove() method to remove the value at a given key from storage.

To add insert and remove functions to the contract:
• Verify that you are in the incrementer project folder.
• Open the lib.rs file in a text editor.
• Add an inc_mine() function that allows the contract caller to get the my_value storage item and insert an incremented value into the mapping.
```
#[ink(message)]
pub fn inc_mine(&mut self, by: i32) {
   let caller = self.env().caller();
   let my_value = self.get_mine();
   self.my_value.insert(caller, &(my_value + by));
}
```

• Add a remove_mine() function that allows the contract caller to get the clear the my_value storage item from storage.
```
#[ink(message)]
pub fn remove_mine(&self) {
   self.my_value.remove(&self.env().caller())
}
```

• Add a new test to verify that the inc_mine() functions works as expected.
```
#[ink::test]
fn inc_mine_works() {
   let mut contract = Incrementer::new(11);
   assert_eq!(contract.get_mine(), 0);
   contract.inc_mine(5);
   assert_eq!(contract.get_mine(), 5);
   contract.inc_mine(5);
   assert_eq!(contract.get_mine(), 10);
}
```

• Add a new test to verify that the remove_mine() functions works as expected.
```
#[ink::test]
fn remove_mine_works() {
   let mut contract = Incrementer::new(11);
   assert_eq!(contract.get_mine(), 0);
   contract.inc_mine(5);
   assert_eq!(contract.get_mine(), 5);
   contract.remove_mine();
   assert_eq!(contract.get_mine(), 0);
}
```

• Use the test subcommand and nightly toolchain to test your work by running the following command:
```
cargo +nightly test
```

The command should display output similar to the following to indicate successful test completion:
```
running 5 tests
test incrementer::tests::default_works ... ok
test incrementer::tests::it_works ... ok
test incrementer::tests::remove_mine_works ... ok
test incrementer::tests::inc_mine_works ... ok
test incrementer::tests::my_value_works ... ok

test result: ok. 5 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

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

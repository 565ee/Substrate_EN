• [introduce](#index1)  
• [Prepare](#index2) 
• [Target](#index3)  
• [Design the application](#index4)  
• [Set up scaffolding for your pallet](#index5)  
• [Configure the pallet to emit events](#index6)  
• [Implement pallet events](#index7)  
• [pallet errors](#index8)  
• [Implement a storage map for stored items](#index9)  
• [Implement callable functions](#index10)  
• [Include MaxBytesInHash runtime configuration](#index11)  
• [Build the runtime with your new pallet](#index12)  
• [Add your custom react component](#index13)  
• [Submit a proof](#index14)  
• [Substrate Tutorials , Substrate 教程](#index98)  
• [Contact 联系方式](#index99)  

# <span id='index1'>• introduce</span>  
This tutorial illustrates how to create a custom proof-of-existence (PoE) service using the Substrate blockchain development framework and the FRAME library.

Proof of existence is an approach to validating the authenticity and ownership of a digital object by using the object information stored on the blockchain. Because the blockchain associates a timestamp and signature with the object, the blockchain record can be used to verify—to serve as proof—that a particular object existed at a specific date and time. It can also verify who the owner of a record was at that date and time.

Instead of individual files, the blockchain stores digital records using a cryptographic hash. The hash enables the blockchain to store files of arbitrary size efficiently by using a small and unique hash value. Because any change to a file would result in a different hash, users can prove the validity of a file by computing the hash and comparing that hash with the hash stored on chain.

Blockchains use public keys to map digital identities to accounts that have private keys. The blockchain records the account you use to store the hash for a digital object as part of the transaction. Because the account information is stored as part of the transaction, the controller of the account can later prove ownership as the person who initially uploaded the file.

# <span id='index2'>• Prepare</span>  
You have configured your environment for Substrate development by installing Rust and the Rust toolchain.

You have completed the Create your first Substrate blockchain and have the node and front-end templates installed.

You are generally familiar with software development and use command-line interfaces.


# <span id='index3'>• Target</span>  
Learn the basic structure of a custom pallet.

See examples of how Rust macros simplify the code you need to write.

Start a blockchain node that contains a custom pallet.

Add front-end code that exposes the proof-of-existence pallet.

# <span id='index4'>• Design the application</span>  
The proof of existence application exposes the following callable functions:

create_claim() allows a user to claim the existence of a file by uploading a hash.

revoke_claim() allows the current owner of a claim to revoke ownership.

These functions only require you to store information about the proofs that have been claimed, and who made those claims.

# <span id='index5'>• Set up scaffolding for your pallet</span>  
The Substrate node template has a FRAME-based runtime. FRAME is a library of code that allows you to build a Substrate runtime by composing modules called "pallets". You can think of the pallets as individual pieces of logic that define what your blockchain can do. Substrate provides you with a number of pre-built pallets for use in FRAME-based runtimes.

This tutorial demonstrates how to create a custom pallet from scratch. Therefore, the first step is to remove some files and content from the files in the node template directory.

• Open a terminal shell and navigate to the root directory for the node template.

• Change to the pallets/template/src directory by running the following command:
```
cd pallets/template/src
```

• Remove the following files:
```
benchmarking.rs
mock.rs
tests.rs
```

• Open the pallets/template/src/lib.rs file in a text editor.

This file contains code that you can use as a template for a new pallet. You won't be using the template code in this tutorial. However, you can review the template code to see what it provides before you delete it.

• Replace all contents of pallets/template/src/lib.rs with the following skeleton code that includes a few minimal macros for FRAME V2:
```
  #![cfg_attr(not(feature = "std"), no_std)]

  pub use pallet::*;

  #[frame_support::pallet]
  pub mod pallet {
      use frame_support::pallet_prelude::*;
      use frame_system::pallet_prelude::*;

      // The struct on which we build all of our Pallet logic.
      #[pallet::pallet]
      #[pallet::generate_store(pub(super) trait Store)]
      pub struct Pallet<T>(_);

      /* Placeholder for defining custom types. */

      // TODO: Update the `config` block below
      #[pallet::config]
      pub trait Config: frame_system::Config {
          type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;
      }

      // TODO: Update the `event` block below
      #[pallet::event]
      #[pallet::generate_deposit(pub(super) fn deposit_event)]
      pub enum Event<T: Config> {}

      // TODO: Update the `error` block below
      #[pallet::error]
      pub enum Error<T> {}

      // TODO: add #[pallet::storage] block

      // TODO: Update the `call` block below
      #[pallet::call]
      impl<T: Config> Pallet<T> {}
  }
```

• Save your changes.

• (Optionally) check that your code compiles by running the following command:
```
# Quick Check the template works *only*
cargo check -p node-template-runtime
# --- AND/OR ---
# Full release build of the node template, inclusive of the template
cargo build -r
```

# <span id='index6'>• Configure the pallet to emit events</span>  
Every pallet has a Rust "trait" called Config. This trait is used to set the interface for the FRAME system and sets the required associated types to be concretely defined in a runtime that includes this pallet. For this tutorial, the configuration setting only enables the pallet to emit events, as almost every pallet does.

To define the Config trait for the proof-of-existence pallet, open the pallets/template/src/lib.rs file in a text editor and update the #[pallet::config] block with to match the following code block:
```
	/// Configure the pallet by specifying the parameters and types on which it depends.
	#[pallet::config]
	pub trait Config: frame_system::Config {
		/// Because this pallet emits events, it depends on the runtime's definition of an event.
		type Event: From<Event<Self>> + IsType<<Self as frame_system::Config>::Event>;
		/// For constraining the maximum bytes of a hash used for any proof
		type MaxBytesInHash: Get<u32>;
	}
```

# <span id='index7'>• Implement pallet events</span>  
To implement the pallet events, update the #[pallet::event] block to match the following code block:
```
	// Pallets use events to inform users when important changes are made.
	// Event documentation should end with an array that provides descriptive names for parameters.
	// https://docs.substrate.io/v3/runtime/events-and-errors
	#[pallet::event]
	#[pallet::generate_deposit(pub(super) fn deposit_event)]
	pub enum Event<T: Config> {
		/// Event emitted when a proof has been claimed. [who, claim]
		ClaimCreated(T::AccountId, BoundedVec<u8, T::MaxBytesInHash>),
		/// Event emitted when a claim is revoked by the owner. [who, claim]
		ClaimRevoked(T::AccountId, BoundedVec<u8, T::MaxBytesInHash>),
	}
```

# <span id='index8'>• pallet errors</span>  
To implement the errors for the proof-of-existence pallet, replace the // TODO: add #[pallet::error] block line with the following code block:
```
	#[pallet::error]
	pub enum Error<T> {
		/// The proof has already been claimed.
		ProofAlreadyClaimed,
		/// The proof does not exist, so it cannot be revoked.
		NoSuchProof,
		/// The proof is claimed by another account, so caller can't revoke it.
		NotProofOwner,
	}
```

• [Implement a storage map for stored items](#index9)  
To implement storage for the proof-of-existence pallet, replace the // TODO: add #[pallet::storage] block line with the following code block:
```
	#[pallet::storage]
	/// Maps each proof to its owner and block number when the proof was made
	pub(super) type Proofs<T: Config> = StorageMap<
		_,
		Blake2_128Concat,
		BoundedVec<u8, T::MaxBytesInHash>,
		(T::AccountId, T::BlockNumber),
		OptionQuery,
	>;
```

# <span id='index10'>• Implement callable functions</span>  
To implement this logic in the proof-of-existence pallet, replace the // TODO: add #[pallet::call] block line with the following code block:
```
	// Dispatchable functions allow users to interact with the pallet and invoke state changes.
	// These functions materialize as "extrinsics", which are often compared to transactions.
	// Dispatchable functions must be annotated with a weight and must return a DispatchResult.
	#[pallet::call]
	impl<T: Config> Pallet<T> {
		#[pallet::weight(1_000)]
		pub fn create_claim(
			origin: OriginFor<T>,
			proof: BoundedVec<u8, T::MaxBytesInHash>,
		) -> DispatchResult {
			// Check that the extrinsic was signed and get the signer.
			// This function will return an error if the extrinsic is not signed.
			// https://docs.substrate.io/v3/runtime/origins
			let sender = ensure_signed(origin)?;

			// Verify that the specified proof has not already been claimed.
			ensure!(!Proofs::<T>::contains_key(&proof), Error::<T>::ProofAlreadyClaimed);

			// Get the block number from the FRAME System pallet.
			let current_block = <frame_system::Pallet<T>>::block_number();

			// Store the proof with the sender and block number.
			Proofs::<T>::insert(&proof, (&sender, current_block));

			// Emit an event that the claim was created.
			Self::deposit_event(Event::ClaimCreated(sender, proof));

			Ok(())
		}

		#[pallet::weight(10_000)]
		pub fn revoke_claim(
			origin: OriginFor<T>,
			proof: BoundedVec<u8, T::MaxBytesInHash>,
		) -> DispatchResult {
			// Check that the extrinsic was signed and get the signer.
			// This function will return an error if the extrinsic is not signed.
			// https://docs.substrate.io/v3/runtime/origins
			let sender = ensure_signed(origin)?;

			// Verify that the specified proof has been claimed.
			ensure!(Proofs::<T>::contains_key(&proof), Error::<T>::NoSuchProof);

			// Get owner of the claim.
			// Panic condition: there is no way to set a `None` owner, so this must always unwrap.
			let (owner, _) = Proofs::<T>::get(&proof).expect("All proofs must have an owner!");

			// Verify that sender of the current call is the claim owner.
			ensure!(sender == owner, Error::<T>::NotProofOwner);

			// Remove claim from storage.
			Proofs::<T>::remove(&proof);

			// Emit an event that the claim was erased.
			Self::deposit_event(Event::ClaimRevoked(sender, proof));
			Ok(())
		}
	}
```

At this point you have a completed pallet! Now to use the pallet, we must correctly configure it in your runtime.

# <span id='index11'>• Include MaxBytesInHash runtime configuration</span>  
You should be curious that the proof-of-existence pallet uses the BoundedVec<u8, T::MaxBytesInHash> type for proofs, but thus far we have no concrete notion of what MaxBytesInHash is. This constant should be set in the runtime to something reasonable for use in your blockchain. One very typical hash type used in many web3 applications is a CID, and the V1 instance of these is typically less than 64 bytes in length. So here we specify MaxBytesInHash to be this length (or less) in the runtime:

• Open the runtime/src/lib.rs file in a text editor.

• Update the pallet_template::Config block to include:
```
/// Configure the pallet-template in pallets/template.
impl pallet_template::Config for Runtime {
	type Event = Event;
	type MaxBytesInHash = frame_support::traits::ConstU32<64>;
}
```

• Save your changes and close the file.

• (Optionally) check that your code compiles by running the following command:
```
cargo check -p node-template-runtime
```

• [Build the runtime with your new pallet](#index12)  
After you've copied all of the parts of the proof-of-existence pallet into the pallets/template/lib.rsfile, you are ready to compile and start the node.

To compile and start the updated Substrate node:

• Open a terminal shell.

• Change to the root directory for the node template.

• Compile the node template by running the following command:
```
cargo build --release
```

• Start the node in development mode by running the following command:
```
./target/release/node-template --dev
```
The --dev option starts the node using the predefined development chain specification. Using the --dev option ensures that you have a clean working state any time you stop and restart the node.

• Verify the node produces blocks.

# <span id='index13'>• Add your custom react component</span>  
• Open a new terminal shell on your computer, then change to the root directory where you installed the front-end template.

• Open the src/TemplateModule.js file in a text editor.

• Delete the entire contents of that file.

• Copy and paste the following code into thesrc/TemplateModule.js file:
```
import React, { useEffect, useState } from 'react'
import { Form, Input, Grid, Message } from 'semantic-ui-react'

// Pre-built Substrate front-end utilities for connecting to a node
// and making a transaction.
import { useSubstrateState } from './substrate-lib'
import { TxButton } from './substrate-lib/components'

// Polkadot-JS utilities for hashing data.
import { blake2AsHex } from '@polkadot/util-crypto'

// Main Proof Of Existence component
function Main(props) {
  // Establish an API to talk to the Substrate node.
  const { api, currentAccount } = useSubstrateState()
  // React hooks for all the state variables we track.
  // Learn more at: https://reactjs.org/docs/hooks-intro.html
  const [status, setStatus] = useState('')
  const [digest, setDigest] = useState('')
  const [owner, setOwner] = useState('')
  const [block, setBlock] = useState(0)

  // Our `FileReader()` which is accessible from our functions below.
  let fileReader;
  // Takes our file, and creates a digest using the Blake2 256 hash function
  const bufferToDigest = () => {
    // Turns the file content to a hexadecimal representation.
    const content = Array.from(new Uint8Array(fileReader.result))
      .map(b => b.toString(16).padStart(2, '0'))
      .join('');
    const hash = blake2AsHex(content, 256);
    setDigest(hash);
  };

  // Callback function for when a new file is selected.
  const handleFileChosen = file => {
    fileReader = new FileReader();
    fileReader.onloadend = bufferToDigest;
    fileReader.readAsArrayBuffer(file);
  };

  // React hook to update the owner and block number information for a file
  useEffect(() => {
    let unsubscribe;
    // Polkadot-JS API query to the `proofs` storage item in our pallet.
    // This is a subscription, so it will always get the latest value,
    // even if it changes.
    api.query.templateModule
      .proofs(digest, result => {
        // Our storage item returns a tuple, which is represented as an array.
        if (result.inspect().inner) {
          let [tmpAddress, tmpBlock] = result.toHuman()
          setOwner(tmpAddress)
          setBlock(tmpBlock)
        } else {
          setOwner('')
          setBlock(0)
        }
      })
      .then(unsub => {
        unsubscribe = unsub;
      });
    return () => unsubscribe && unsubscribe();
    // This tells the React hook to update whenever the file digest changes
    // (when a new file is chosen), or when the storage subscription says the
    // value of the storage item has updated.
  }, [digest, api.query.templateModule])

  // We *assume* a file digest is claimed if the stored block number is not 0
  function isClaimed() {
    return block !== 0
  }

  // The actual UI elements which are returned from our component.
  return (
    <Grid.Column>
      <h1>Proof of Existence</h1>
      {/* Show warning or success message if the file is or is not claimed. */}
      <Form success={!!digest && !isClaimed()} warning={isClaimed()}>
        <Form.Field>
          {/* File selector with a callback to `handleFileChosen`. */}
          <Input
            type="file"
            id="file"
            label="Your File"
            onChange={e => handleFileChosen(e.target.files[0])}
          />
          {/* Show this message if the file is available to be claimed */}
          <Message success header="File Digest Unclaimed" content={digest} />
          {/* Show this message if the file is already claimed. */}
          <Message
            warning
            header="File Digest Claimed"
            list={[digest, `Owner: ${owner}`, `Block: ${block}`]}
          />
        </Form.Field>
        {/* Buttons for interacting with the component. */}
        <Form.Field>
          {/* Button to create a claim. Only active if a file is selected, and not already claimed. Updates the `status`. */}
          <TxButton
            label="Create Claim"
            type="SIGNED-TX"
            setStatus={setStatus}
            disabled={isClaimed() || !digest}
            attrs={{
              palletRpc: 'templateModule',
              callable: 'createClaim',
              inputParams: [digest],
              paramFields: [true]
            }}
          />
          {/* Button to revoke a claim. Only active if a file is selected, and is already claimed. Updates the `status`. */}
          <TxButton
            label="Revoke Claim"
            type="SIGNED-TX"
            setStatus={setStatus}
            disabled={!isClaimed() || owner !== currentAccount.address}
            attrs={{
              palletRpc: 'templateModule',
              callable: 'revokeClaim',
              inputParams: [digest],
              paramFields: [true]
            }}
          />
        </Form.Field>
        {/* Status message about the transaction. */}
        <div style={{ overflowWrap: 'break-word' }}>{status}</div>
      </Form>
    </Grid.Column>
  );
}

export default function TemplateModule(props) {
  const { api } = useSubstrateState()
  return api.query.templateModule ? <Main {...props} /> : null

}
```

• Save your changes and close the file.

• Start the front-end template by running the following commands:
```
nvm install # use the correct node version
yarn        # instal deps
yarn start  # start a dev server
```

# <span id='index14'>• Submit a proof</span>  
To test the proof-of-existence pallet using the new front-end component:

• Find the component at the bottom of the page.

• Click Choose file and select any file on your computer.

The proof-of-existence pallet generates the hash for the selected file and displays it in the File Digest field.

Because the file does not have an owner or block number, it is available to claim.

• Click Create Claim to take ownership of the file.
![poe-component](https://user-images.githubusercontent.com/28084126/172207586-c091fa6f-60df-434c-a7a3-4fac2a482694.png)

Clicking Create Claim calls the create_claim function in the custom proof-of-existence pallet. The front-end component displays the file digest, account identifier, and block number for the completed transaction.

• Verify the claim is successful and a new claimCreated event appears in the Events component.
![poe-claimed](https://user-images.githubusercontent.com/28084126/172208054-7f042fbe-ff5f-486c-88f6-b7afcb6bacda.png)

The front-end component recognizes that the file is now claimed, and gives you the option to revoke the claim.

Remember, only the owner can revoke the claim. If you select another user account, the revoke option is disabled.

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

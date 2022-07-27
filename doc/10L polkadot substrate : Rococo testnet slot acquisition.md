• [introduce](#index1)  
• [Set up a wallet](#index2)  
• [Acquire a ParaID](#index3)  
• [Generate parachain genesis and Wasm files](#index4)  
• [Start your collator](#index5)  
• [Register as a parathread](#index6)  
• [Request your parachain slot](#index7)  
• [Substrate Tutorials , Substrate 教程](#index98)   
• [Contact 联系方式](#index99)

# <span id='index1'>• introduce</span>  
Rococo is Parity's public test network for parachains. This will guide you through how you can onboard your parachain to it.

Before you begin
Because Rococo is a shared test network, this tutorial requires additional steps to prepare your environment that aren't required for local testing. Before you begin, verify the following:

You know how to generate and modify chain specification files as described in Add trusted nodes.
You know how to generate and store keys using one of the methods described in Add trusted nodes.
You have completed the parachain tutorial and tested a parachain on your local computer.

# <span id='index2'>• Set up a wallet</span>  
To perform any action on Rococo, you need ROC tokens. To store these, you need a Substrate compatible cryptocurrency wallet. For this operations in any public setting, you can not use development keys and accounts. There are many options available for setting up wallet account. For information about some of these options, see the Build wallets on the Polkadot wiki. If you are just getting started, you can use the polkadot-js extension. After you have an account, be sure to:

Back up your secret seed phrase.
Make note of your accountID that is using the default 42 SS58 prefix for use with Rococo.
To acquire ROC, join the Rococo faucet matrix channel and use the !drip <accountID> command in the faucet channel to get 100 ROC in your wallet.
https://matrix.to/#/#rococo-faucet:matrix.org

# <span id='index3'>• Acquire a ParaID</span>  
With a minimal 5 ROC free balance, register as a parathread on Rococo:

Go to the Polkadot-JS Apps parathreads section for Rococo.
Reserve a unique ParaID. You will be assigned to the next available ID - make note of this.
![image](https://user-images.githubusercontent.com/28084126/178095955-aa9c2a57-523d-4a62-9aeb-ca5ed39bfd90.png)

Note that on Rococo, ParaID numbers 0-999 are reserved for system parachains and 1000-1999 are reserved for public utility parachains. Only numbers 2000 and above that aren't already reserved can be used for community parachains.

# <span id='index4'>• Generate parachain genesis and Wasm files</span>  
The files required to register a parachain must explicitly specify the correct relay chain and the right ParaID. In this tutorial, the relay chain is rococo instead of rococo-local used in the Connect a local parachain tutorial.

• Configure your chain specification to use:

Your Rococo ParaID.
Unique alternatives to the development keys and accounts for your collator nodes. While Alice and such accounts will work you should absolutely not use them!

• Generate the appropriate parachain's genesis state for Rococo.

• Generate the parachain runtime Wasm blob for Rococo.

# <span id='index5'>• Start your collator</span>  
You must have your collator's peering ports for the embedded relay chain and your parachain publicly accessible and discoverable. This way you are able to peer with Rococo validator nodes, otherwise you will not be able to produce blocks! The peering port is set with the --port <collator node>-- --port <relay node> CLI flags, be sure to do this for both nodes separately. It's very likely you want at least your collator's --ws-port <ws port> port open as well to allow for yourself (and others) to connect with it via the Polkadot Apps UI or API calls.

# <span id='index6'>• Register as a parathread</span>  
Before you can become a parachain, you must register as a parathread on Rococo:

• Register as a parathread. using your reserved ParaID.
![image](https://user-images.githubusercontent.com/28084126/178096126-a1662184-059c-46a3-8fa3-e930f5a5cc6c.png)

• Check that you see a registrar.Registered event in the Polkadot-JS application to verify registration succeeded.
![image](https://user-images.githubusercontent.com/28084126/178096145-8a4186c8-8320-42bc-92b8-3ed09bb639ec.png)

• Click Parachains -> Parathreads and verify that your parathread registration is Onboarding:
![image](https://user-images.githubusercontent.com/28084126/178096164-2189a633-d4d7-4a15-aeb9-b3db1e359b2a.png)

# <span id='index7'>• Request your parachain slot</span>  
After the parachain is active as a parathread, the related project team should open a request for either a permanent or a temporary parachain slot on Rococo.

• Permanent slots are parachain slots assigned to teams who currently have a parachain slot on Polkadot (following a successful slot lease auction) and thus have the needs for continuously testing their codebase for compatibility with the latest bleeding edge features in a real-world live environment (Rococo). Only a limited number of permanent slots are available (see note below).

• Temporary slots are parachain slots that are dynamically allocated in a continuous, round-robbin style roation. At the start of every lease period, a certain number of parathreads—up to a maximum defined in the relay chain's configuration—are automatically upgraded to parachains for a certain duration. The parachains that were active during the ending lease period are automatically downgraded to parathreads to free the slots for others to use in the subsequent period. The temporary slots are meant to be used by teams who do not have a parachain slot yet on Polkadot, and are aiming for one in the near future.

The goal dynamic allocations is to help teams test their runtimes more often, and in a more streamlined manner.

The Rococo runtime requires sudo access to assign slots (AssignSlotOrigin = EnsureRoot<Self::AccountId>;). The Rococo sudo key is currently controlled by Parity Technologies at this time, so to have the operation required to get a slot, please go to the Subport repo and open a Rococo Slot Request once you complete the above and are ready to connect! A Parity team member will respond once your slot has been activated. Eventually, this process is intended to be community-driven through the Rococo governance framework.

• Using an account with the AssignSlotOrigin origin, follow these steps to assign a temporary slot:
Open Polkadot-JS Apps for Rococo.
Click Developer > Extrinsics.
Select the account you want to use to submit the transaction.
Select the assignedSlots pallet.
Choose the assignTempParachainSlot function.
Insert your reserved ParaID. Make sure you use the ParaID you previously reserved.
Select Current for the LeasePeriodStart. If the current slot is full, you will be assigned the next available slot.
Sign and submit the transaction.

• Given a period lease duration of 1 day, the current settings for assigned parachain slots on Rococo are (at the time of writing):
Permanent slot least duration: 1 year (365 days)
Temporary slot least duration: 3 days
Maximum number of permanent slots: up to 25 permanent slots
Maximum number of temporary slots: up to 20 temporary slots
Maximum temporary slots allocated per leased period: up to 5 temporary slots per 3-day temporary lease periods


After your slot is activated by the Parity team, you can test your parachain on the Rococo test network. Note that when your temporary slot's lease ends, the parachain is automatically downgraded to a parathread. Registered and approved slots are cycled through automatically in a round-robin fashion, so you can expect to come back online as a parachain from time to time.

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

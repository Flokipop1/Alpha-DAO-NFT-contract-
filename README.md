# Alpha DAO: Frontend Integration Guide

​This guide explains how the React frontend interacts with the Tact smart contract.

# ​1. Setup & Connection
​The app must use @tonconnect/ui-react for wallet connections and @ton/ton to communicate with the blockchain.
​Contract Address: [PASTE_COLLECTION_ADDRESS_HERE]
Network: TON Testnet

# ​2. Permissions: Determining Owner/User
​Upon wallet connection, the app must call the contract's get_owner() method.

​Logic Flow:
​Get the connected user's address from TON Connect: const userAddress = useTonAddress();
​Call the contract getter get_owner() (this is a free, read-only call).
​If userAddress == contractOwner, unlock the Admin Dashboard. Otherwise, show the Minting Page.


# ​Code Example (Reading the Owner):

import { TonClient, Address } from "@ton/ton";
import { AlphaDaoContract } from "../wrappers/AlphaDaoContract"; // The file from your build folder

const client = new TonClient({ endpoint: "https://testnet.toncenter.com/api/v2/jsonRPC" });
const contract = client.open(AlphaDaoContract.fromAddress(Address.parse("CONTRACT_ADDRESS")));

const owner = await contract.getOwner();


# 3. Interaction Logic (Sending Messages)

To perform actions, the frontend sends a "Message" with a specific Payload (Body).

A. Public Minting
When a user clicks "Mint," they must send exactly 5.5 TON (5.0 Price + 0.5 Gas).// will be add coded on the script 

Message Type: Mint (defined in the contract).
Payload: Should be the encoded Mint message.
B. Admin: Pause / Resume

Message Type: Simple Comment or Opcode.
Action: send a message with the text "Pause".
Value: Send a small amount of gas (e.g., 0.05 TON).

C. Admin: Withdraw Funds
Message Type: Withdraw
Action: Triggers the contract to send its entire TON balance to the owner's wallet.
Value: Send a small amount of gas (e.g., 0.05 TON).
4. Sending Transactions (TON Connect UI)
use the useTonConnectUI hook to trigger the wallet popup. see transaction example or template you can use

# message Transaction example 



import { useTonConnectUI } from '@tonconnect/ui-react';
import { beginCell, toNano } from '@ton/ton';

export function AdminPanel() {
  const [tonConnectUI] = useTonConnectUI();

  const handleWithdraw = async () => {
    // 1. Create the Payload (What the contract should do)
    const body = beginCell()
        .storeUint(0, 32)            // Indicator for text comment
        .storeStringTail("Withdraw") // The command
        .endCell();

    const base64Payload = body.toBoc().toString("base64");

    // 2. Define the Transaction
    const myTransaction = {
      validUntil: Math.floor(Date.now() / 1000) + 360, // 6 mins
      messages: [
        {
          address: "CONTRACT_ADDRESS_HERE", // The Collection Address
          amount: toNano("0.05").toString(),     // Gas to process the request
          payload: base64Payload,                // The instruction we built above
        },
      ],
    };

    // 3. Trigger the Wallet Popup
    try {
      await tonConnectUI.sendTransaction(myTransaction);
      alert("Withdraw request sent to wallet!");
    } catch (e) {
      console.error("User rejected or error:", e);
    }
  };

  return (
    <button onClick={handleWithdraw}>
       Withdraw Funds
    </button>
  );
}



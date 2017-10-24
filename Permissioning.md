# Overview
In a basic [Proof-of-Work blockchain](https://github.com/paritytech/parity/wiki/Proof-of-Work-Chains) all participants are able to perform all roles within the network: connect, mine (validate), send transactions, inspect the state and see all transactions.

At Parity Technologies we are introducing a number of features which enable the network participants to permission different aspects of a blockchain. Often conflated as simply “permissioned blockchains” we introduce permissions on a number of different layers:

* [Network](Permissioning#network)
* [Transaction type](Permissioning#transaction-type)
* [Validator set](Permissioning#validator-set)
* Gas price (to-be-released)
* Secret transactions and contracts (to-be-released)

Each user can have different permissions on each layer. All permissioning is based on blockchain accounts, which means that permissions always correspond to an address.

# Network
Permissions on this layer determine which nodes can connect to the network and interact with others. In Parity, individual network members can control their network peers. A Network smart contract enables the governing body to impose any network topology and disallow connections from any external parties.

## How it works
A smart contract has to be deployed that regulates if two nodes can connect to each other given their enode IDs.
The contract must be deployed on the corresponding chain and its address added to the chain spec file under `"params"/"transactionPermissionContract"`.

The contract must support the following ABI:
```json
[
  {
    "constant": false,
    "inputs": [
      {
        "name": "sender",
        "type": "address"
      }
    ],
    "name": "allowedTxTypes",
    "outputs": [
      {
        "name": "",
        "type": "uint32"
      }
    ],
    "payable": false,
    "stateMutability": "nonpayable",
    "type": "function"
  }
]
```

Here's a sample contract that implements that ABI:

TO-BE-ADDED

# Transaction type
Even though all network participants can submit transactions it is still up to the validators to include them. Besides that, it is possible to restrict certain addresses or transaction types on the state transition level. 

By transaction types we mean:

* Contract deployment
* Contract interaction
* Simple transfer

The ability of each address to execute any combination of these transaction types can be determined by a contract implementing a special interface.

## How it works
A smart contract has to be deployed that regulates which participant (address) can perform certain types of transactions. The contract must be deployed on the corresponding chain and its address added to the chain spec file under `"TO-BE-ADDED"`.

The contract must support the following ABI:

TO-BE-ADDED

Here's a sample contract that implements that ABI:
```solidity
pragma solidity ^0.4.11;

contract PeerManager {
    mapping(bytes32 => bytes32) peers;
    
    function PeerManager() {
        peers[0x11] = 0x12;
        peers[0x21] = 0x22;
        peers[0x31] = 0x32;
        peers[0x41] = 0x42;
    }
    
    function connectionAllowed(bytes32 sl, bytes32 sh, bytes32 pl, bytes32 ph) constant returns (bool res) {
        if (sl == 0x01 && sh == 0x02) {
	    return true;
	}
	return pl != 0 && ph != 0 && peers[pl] == ph;
    }
}
```
* `sl` is low 32 bytes of peer 1 enode Id
* `sh` is hi 32 bytes of peer 1 enode Id
* `pl` is low 32 bytes of peer 2 enode Id
* `ph` is hi 32 bytes of peer 2 enode Id

# Validator set
This level of permissions is a rather important one. It determines which parties (Validators) are entitled to create new blocks and thereby build the blockchain. Validators need to collect and validate transactions before sealing them into blocks. Rules according to which they interact can be referred to as a consensus engine. Parity currently supports three different consensus engines:

* Ethash (PoW)
* Aura
* Tendermint (PBFT)

More are being implemented.

For each consensus engine, there are two main varieties of permissioned validation: Proof-of-Authority and Proof-of-Stake. In Proof-of-Authority, validators typically represent some real world entities, which prevents Sybil attacks. These authorities can be added and removed according to a set of rules, such as via a voting process. The rules are specified in a smart contract on the blockchain. Proof-of-Stake on the other hand, relies on security deposits. This means that validators are added after submitting a sufficient amount of valuable tokens, which can be taken away in the case of misbehaviour.

In both cases, Parity Ethereum is able to automatically detect faults in the consensus process and respond immediately. Two types of misbehaviour are possible: malicious and benign. When a malicious misbehaviour is detected by a node, a proof of misbehaviour can be provided to the contract. Benign misbehaviour is more speculative: a node can be never sure if it actually occurred (e.g. differentiating between downtime and a network partition).

In Parity Ethereum a Validator Set can be specified using a contract implementing a special interface. Thanks to the smart contract definition authorities can be managed according to any rules suitable to the particular application.

## How it works
Please see the [Validator Set wiki page](https://github.com/paritytech/parity/wiki/Validator-Set).

# Gas price (to-be-released)
In addition to completely disabling certain accounts from making transactions, a way to specify gas prices per account will be possible. This is achieved by managing gas prices in a smart contract and enables one to regulate how much each account has to spend on interactions with the blockchain. Of course, gas prices can also be set to zero.
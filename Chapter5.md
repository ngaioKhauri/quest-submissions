## Chapter 5

### Day 1

**1. Describe what an event is, and why it might be useful to a client.**

An event is when a smart contract tells the outside world that something happened.

It might be useful to a client because a client can determine when something happens and then do something.

For example, let's say a client is writing an NFT and wants to celebrate when an NFT with ID 100 is minted.
The NFT can emit an event every time it mints an NFT (let's say the event contains the ID of the NFT that was just minted).
The client would be able to determine when an NFT with ID 100 is minted and go celebrate :)

**2. Deploy a contract with an event in it, and emit the event somewhere else in the contract indicating that it happened.**

```
pub contract Test {
  pub event NFTDestroyed(id: UInt64)

  pub resource NFT {
    pub var id: UInt64

    pub fun incrementId(): UInt64 {
      self.id = self.id + 1
      return self.id
    }

    init() {
      post {
        self.id != 5     
      }
      self.id = self.uuid
    }

    destroy() {
      emit NFTDestroyed(id: self.id)
    }
  }
}
```

**3. Using the contract in step 2), add some pre conditions and post conditions to your contract to get used to writing them out.**

```
pub contract Test {
  pub event NFTDestroyed(id: UInt64)

  pub resource NFT {
    pub var id: UInt64

    pub fun incrementId(): UInt64 {
      pre {
        self.id != 3
      }
      self.id = self.id + 1
      return self.id
    }

    init() {
      post {
        self.id != 5     
      }
      self.id = self.uuid
    }

    destroy() {
      emit NFTDestroyed(id: self.id)
    }
  }
}
```

**4. For each of the functions below (numberOne, numberTwo, numberThree), follow the instructions.**

`numberOne` logs the name.
`numberTwo` returns a value.
`numberThree` does not return the updated number. The value of `self.number` after it's run is 0.

```
pub contract Test {

  // TODO
  // Tell me whether or not this function will log the name.
  // name: 'Jacob'
  pub fun numberOne(name: String) {
    pre {
      name.length == 5: "This name is not cool enough."
    }
    log(name)
  }

  // TODO
  // Tell me whether or not this function will return a value.
  // name: 'Jacob'
  pub fun numberTwo(name: String): String {
    pre {
      name.length >= 0: "You must input a valid name."
    }
    post {
      result == "Jacob Tucker"
    }
    return name.concat(" Tucker")
  }

  pub resource TestResource {
    pub var number: Int

    // TODO
    // Tell me whether or not this function will log the updated number.
    // Also, tell me the value of `self.number` after it's run.
    pub fun numberThree(): Int {
      post {
        before(self.number) == result + 1
      }
      self.number = self.number + 1
      return self.number
    }

    init() {
      self.number = 0
    }

  }

}
```

### Day 2

**1. Explain why standards can be beneficial to the Flow ecosystem.**

A standard is beneficial to the Flow ecosystem because a client interacting with contracts that meet the standard can interact with the contracts in a single, uniform manner.

**2. What is YOUR favourite food?**

My favorite food is Waakye

**3. Please fix this code (Hint: There are two things wrong):**

The contract interface:
```
pub contract interface ITest {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    pre {
      newNumber >= 0: "We don't like negative numbers for some reason. We're mean."
    }
    post {
      self.number == newNumber: "Didn't update the number to be the new number."
    }
  }

  pub resource interface IStuff {
    pub var favouriteActivity: String
  }

  pub resource Stuff {
    pub var favouriteActivity: String
  }
}
```
The implementing contract:
```
pub contract Test: ITest {
  pub var number: Int
  
  pub fun updateNumber(newNumber: Int) {
    self.number = 5
  }

  pub resource interface IStuff {
    pub var favouriteActivity: String
  }

  pub resource Stuff: IStuff, ITest.IStuff {
    pub var favouriteActivity: String

    init() {
      self.favouriteActivity = "Playing League of Legends."
    }
  }

  init() {
    self.number = 0
  }
}
```

### Day 3

**What does "force casting" with as! do? Why is it useful in our Collection?**

Force casting changes from one type to another type (for example from the generic `@NonFungibleToken.NFT` type to the more specific `@NFT` type (the CryptoPoops NFT type).

This is useful in our Collection because although the deposit function must take in `@NonFungibleToken.NFT`, the force casting to the CryptoPoops NFT type ensures that we can only deposit CryptoPoops NFTs in our collection.

**What does auth do? When do we use it?**

Prepending auth gives us an "authorized" reference to a resource.

We use it when we want to downcast a reference to a resource to another reference. To do this, the origin reference that we have must be an authorized reference.

**This last quest will be your most difficult yet. Take this contract:**
```
import NonFungibleToken from 0x02
pub contract CryptoPoops: NonFungibleToken {
  pub var totalSupply: UInt64

  pub event ContractInitialized()
  pub event Withdraw(id: UInt64, from: Address?)
  pub event Deposit(id: UInt64, to: Address?)

  pub resource NFT: NonFungibleToken.INFT {
    pub let id: UInt64

    pub let name: String
    pub let favouriteFood: String
    pub let luckyNumber: Int

    init(_name: String, _favouriteFood: String, _luckyNumber: Int) {
      self.id = self.uuid

      self.name = _name
      self.favouriteFood = _favouriteFood
      self.luckyNumber = _luckyNumber
    }
  }

  pub resource Collection: NonFungibleToken.Provider, NonFungibleToken.Receiver, NonFungibleToken.CollectionPublic {
    pub var ownedNFTs: @{UInt64: NonFungibleToken.NFT}

    pub fun withdraw(withdrawID: UInt64): @NonFungibleToken.NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
            ?? panic("This NFT does not exist in this Collection.")
      emit Withdraw(id: nft.id, from: self.owner?.address)
      return <- nft
    }

    pub fun deposit(token: @NonFungibleToken.NFT) {
      let nft <- token as! @NFT
      emit Deposit(id: nft.id, to: self.owner?.address)
      self.ownedNFTs[nft.id] <-! nft
    }

    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    pub fun borrowNFT(id: UInt64): &NonFungibleToken.NFT {
      return (&self.ownedNFTs[id] as &NonFungibleToken.NFT?)!
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  pub fun createEmptyCollection(): @NonFungibleToken.Collection {
    return <- create Collection()
  }

  pub resource Minter {

    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0
    emit ContractInitialized()
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```
**and add a function called borrowAuthNFT just like we did in the section called "The Problem" above. Then, find a way to make it publically accessible to other people so they can read our NFT's metadata. Then, run a script to display the NFTs metadata for a certain id.**

**You will have to write all the transactions to set up the accounts, mint the NFTs, and then the scripts to read the NFT's metadata. We have done most of this in the chapters up to this point, so you can look for help there :)**


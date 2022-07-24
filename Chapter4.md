## Chapter 4

### Day 1

**1. Explain what lives inside of an account.**

1. Contract Code - the code of the contract that you deploy to your account
2. Account Storage - your stored data

**2. What is the difference between the /storage/, /public/, and /private/ paths?**

/storage/ - only the account owner can access it. all your data is here
/public/ - everyone can access it
/private/ - only the account owner and people who the owner gives access to it can access it

**3. What does .save() do? What does .load() do? What does .borrow() do?**

`.save()` saves data to a `/storage/` path
`.load()` loads data from a `/storage/` path
`.borrow()` returns a reference to data in a `/storage/` path

**4. Explain why we couldn't save something to our account storage inside of a script.**

A script cannot change data on a blockchain

**5. Explain why I couldn't save something to your account.**

Only I have access to the data in my `/storage/` path

**6. Define a contract that returns a resource that has at least 1 field in it. Then, write 2 transactions:**

```
access(all) contract HelloWorld {

    access(all) resource Ngaio {
        access(all) var hatesJava: Bool
        init() {
            self.hatesJava = true
        }
    }

    // Public function that returns our friendly greeting!
    access(all) fun makeNgaio(): @Ngaio {
        return <- create Ngaio()
    }
}
```

  **- A transaction that first saves the resource to account storage, then loads it out of account storage, logs a field inside the resource, and destroys it.**

```
import HelloWorld from 0x01

transaction {

  prepare(acct: AuthAccount) {
    var ngaio <- HelloWorld.makeNgaio()
    acct.save(<- ngaio, to: /storage/ngaio_kingdom)

    var loaded <- acct.load<@HelloWorld.Ngaio>(from: /storage/ngaio_kingdom)!
    log(loaded.hatesJava)
    destroy loaded
  }

  execute {}
}
```

  **- A transaction that first saves the resource to account storage, then borrows a reference to it, and logs a field inside the resource.**
```
import HelloWorld from 0x01

transaction {

  prepare(acct: AuthAccount) {
    var ngaio <- HelloWorld.makeNgaio()
    acct.save(<- ngaio, to: /storage/ngaio_dungeon)

    var borrowedRef = acct.borrow<&HelloWorld.Ngaio>(from: /storage/ngaio_dungeon)!
    log(borrowedRef.hatesJava)
  }

  execute {}
}
```

### Day 2

**1. What does .link() do?**

Allows us to link a resource in a `/storage/` path to a `/public/` or `/private/` path in order to expose a reference to that resource.

**2. In your own words (no code), explain how we can use resource interfaces to only expose certain things to the /public/ path.**

Ok, so first we have a resource in a `/storage/` path.
Write a resource interface that our resource implements.
We write the resource interface so that it exposes only the "certain things" we would like to expose to the `/public` path.
Use `link` to link the resource in `/storage/` to the `/public/` path, making sure to specify the type as the type of the resource restricted to the resource interface.

Now, the public can only obtain a reference to the resource restricted to the resource interface when using getCapability.
As a result, the public can only view the "certain things" that I, the owner of the resource, exposed in the resource interface.

**3. Deploy a contract that contains a resource that implements a resource interface. Then, do the following:**
```
access(all) contract HelloWorld {

    access(all) resource interface NgaioInterface {
        access(all) var isAustralian: Bool
    }

    access(all) resource Ngaio: NgaioInterface {
        access(all) var hatesJava: Bool
        access(all) var isAustralian: Bool
        init() {
            self.hatesJava = true
            self.isAustralian = true
        }
    }

    // Public function that returns our friendly greeting!
    access(all) fun makeNgaio(): @Ngaio {
        return <- create Ngaio()
    }
}
```
  **- In a transaction, save the resource to storage and link it to the public with the restrictive interface.**
```
import HelloWorld from 0x01

transaction {

  prepare(acct: AuthAccount) {
    var ngaio <- HelloWorld.makeNgaio()
    acct.save(<- ngaio, to: /storage/ngaio_kingdom)

    acct.link<&HelloWorld.Ngaio{HelloWorld.NgaioInterface}>(/public/ngaio_kingdom, target: /storage/ngaio_kingdom)
  }

  execute {}
}
```
  **- Run a script that tries to access a non-exposed field in the resource interface, and see the error pop up.**
```
import HelloWorld from 0x01

pub fun main(): Bool {
  let publicCapability: Capability<&HelloWorld.Ngaio{HelloWorld.NgaioInterface}> =
    getAccount(0x01).getCapability<&HelloWorld.Ngaio{HelloWorld.NgaioInterface}>(/public/ngaio_kingdom)

  let testResource: &HelloWorld.Ngaio{HelloWorld.NgaioInterface} = publicCapability.borrow()!

  return testResource.isAustralian
}
```
  **- Run the script and access something you CAN read from. Return it from the script.**
```
import HelloWorld from 0x01

pub fun main(): Bool {
  let publicCapability: Capability<&HelloWorld.Ngaio{HelloWorld.NgaioInterface}> =
    getAccount(0x01).getCapability<&HelloWorld.Ngaio{HelloWorld.NgaioInterface}>(/public/ngaio_kingdom)

  let testResource: &HelloWorld.Ngaio{HelloWorld.NgaioInterface} = publicCapability.borrow()!

  return testResource.hatesJava
}
```
### Day 3

**1. Why did we add a Collection to this contract? List the two main reasons.**

1. If we have many NFTs, we would have to keep track of all their storage paths, which is inconvenient.
2. Nobody can give us NFTs because only I can store an NFT in my account storage directly.

**2. What do you have to do if you have resources "nested" inside of another resource? ("Nested resources")**

If an "outer" resource consists of "nested" resources, the "outer" resource must have a destroy function that manually destroys the "nested" resources using the `destroy` keyword.

**3. Brainstorm some extra things we may want to add to this contract. Think about what might be problematic with this contract and how we could fix it.**

  **- Idea #1: Do we really want everyone to be able to mint an NFT? 🤔.**

    No, we do not want everyone to be able to mint an NFT. We should create a minter resource that contains the createNFT function. Only the person who deploys the contract should have access to this minter resource, allowing only them to mint NFTs.

  **- Idea #2: If we want to read information about our NFTs inside our Collection, right now we have to take it out of the Collection to do so. Is this good?**

    No, this is not great. It would be better to create a function in the Collection resource that takes in an id and returns a (optional) reference to the NFT to which the id maps inside the Collection.

### Day 4

**Take our NFT contract so far and add comments to every single resource or function explaining what it's doing in your own words**
```
pub contract CryptoPoops {
  pub var totalSupply: UInt64

  // This is an NFT resource that contains a name,
  // favouriteFood, and luckyNumber
  pub resource NFT {
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

  // This is a resource interface that exposes
  // some functions of Collection to the public
  pub resource interface CollectionPublic {
    pub fun deposit(token: @NFT)
    pub fun getIDs(): [UInt64]
    pub fun borrowNFT(id: UInt64): &NFT
  }

  // This is a resource that holds NFTs
  pub resource Collection: CollectionPublic {
    pub var ownedNFTs: @{UInt64: NFT}

    // Takes in an NFT and puts it into the collection
    pub fun deposit(token: @NFT) {
      self.ownedNFTs[token.id] <-! token
    }

    // Given an id, withdraws and returns the corresponding NFT from the collection
    pub fun withdraw(withdrawID: UInt64): @NFT {
      let nft <- self.ownedNFTs.remove(key: withdrawID) 
              ?? panic("This NFT does not exist in this Collection.")
      return <- nft
    }

    // Returns array containing IDs of all NFTs in the collection
    pub fun getIDs(): [UInt64] {
      return self.ownedNFTs.keys
    }

    // Given an id, returns a reference to the correponding NFT in the collection
    pub fun borrowNFT(id: UInt64): &NFT {
      return (&self.ownedNFTs[id] as &NFT?)!
    }

    init() {
      self.ownedNFTs <- {}
    }

    destroy() {
      destroy self.ownedNFTs
    }
  }

  // Creates and returns a new, empty Collection resource
  pub fun createEmptyCollection(): @Collection {
    return <- create Collection()
  }

  // This is a resource that allows a person to mint an NFT
  // Only the deployer of the contract has an instance of this resource
  pub resource Minter {

    // This function creates an NFT
    pub fun createNFT(name: String, favouriteFood: String, luckyNumber: Int): @NFT {
      return <- create NFT(_name: name, _favouriteFood: favouriteFood, _luckyNumber: luckyNumber)
    }

    // This function creates a minter
    pub fun createMinter(): @Minter {
      return <- create Minter()
    }

  }

  init() {
    self.totalSupply = 0
    self.account.save(<- create Minter(), to: /storage/Minter)
  }
}
```

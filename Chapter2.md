## Chapter 2

### Day 1

**1. Deploy a contract to account 0x03 called "JacobTucker". Inside that contract, declare a constant variable named is, and make it have type String. Initialize it to "the best" when your contract gets deployed.**

```
access(all) contract JacobTucker {
    access(all) let is: String

    init() {
        self.is = "the best"
    }
}
```

**2. Check that your variable is actually equals "the best" by executing a script to read that variable. Include a screenshot of the output.**

![output_text](./output.PNG)

### Day 2

**1. Explain why we wouldn't call changeGreeting in a script.**

Scripts cannot change data on the blockchain/in a smart contract.

**2. What does the AuthAccount mean in the prepare phase of the transaction?**

AuthAccount is the person signing the transaction. This is the person who pays for the transaction. When you sign a transaction using the AuthAccount, the transaction can access the data in your account.

**3. What is the difference between the prepare phase and the execute phase in the transaction?**

Purpose of prepare phase: to access the information/data in your account

Purpose of execute phase: to call functions to view/change the data on the blockchain

**4. This is the hardest quest so far, so if it takes you some time, do not worry! I can help you in the Discord if you have questions.**

- **Add two new things inside your contract:**
    - **A variable named myNumber that has type Int (set it to 0 when the contract is deployed)**

    - **A function named updateMyNumber that takes in a new number named newNumber as a parameter that has type Int and updates myNumber to be newNumber**


```
pub contract HelloWorld {
    pub var greeting: String
    pub var myNumber: Int

    pub fun changeGreeting(newGreeting: String) {
        self.greeting = newGreeting
    }

    pub fun updateMyNumber(newNumber: Int) {
      self.myNumber = newNumber
    }

    init() {
        self.greeting = "Hello, World!"
        self.myNumber = 0
    }
}
```

**- Add a script that reads myNumber from the contract**
```
import HelloWorld from 0x01

pub fun main(): Int {
    return HelloWorld.myNumber
}
```


**- Add a transaction that takes in a parameter named myNewNumber and passes it into the updateMyNumber function. Verify that your number changed by running the script again.**

```
import HelloWorld from 0x01

transaction(myNewNumber: Int) {

  prepare(signer: AuthAccount) {}

  execute {
    HelloWorld.updateMyNumber(newNumber: myNewNumber)
  }
}
```

### Day 3

**1. In a script, initialize an array (that has length == 3) of your favourite people, represented as Strings, and log it.**
```
pub fun main(){
    var arr: [String] = ["Jacob", "Agatha", "Maori"]
    log(arr)
}
```

**2. In a script, initialize a dictionary that maps the Strings Facebook, Instagram, Twitter, YouTube, Reddit, and LinkedIn to a UInt64 that represents the order in which you use them from most to least. For example, YouTube --> 1, Reddit --> 2, etc. If you've never used one before, map it to 0!**
```
pub fun main(){
    var dic: {String: Int} = {"Facebook": 0, "Instagram": 0, "Twitter": 0, "YouTube": 0, "Reddit": 0, "LinkedIn": 0}
    log(dic)
}
```


**3. Explain what the force unwrap operator ! does, with an example different from the one I showed you (you can just change the type).**
```
pub fun main(): Int {
    let thing: {Int: Int} = {3: 1, 5: 2, 2: 3}
    return thing[6]!
}
```
The force unwrap forces the script to return an Int (i.e. the script may not return null)

**4. Using this picture below, explain...**

* **What the error message means**: The return type must be `String`, but `String?` is being returned
* **Why we're getting this error** Because the accessing a value in a dictionary always returns that value as an optional type (i.e. `String?` as opposed to `String` here)
* **How to fix it** One way is to return `thing[0x03]!`. Another way is to change the return type to `String?`

### Day 4

**1. Deploy a new contract that has a Struct of your choosing inside of it (must be different than Profile).**
```
pub contract NgaioContract {

    pub struct Ngaio {
        pub let birthday: String

        // You have to pass in 4 arguments when creating this Struct.
        init(_birthday: String) {
            self.birthday = _birthday
        }
    }

    init() {
    }

}
```
**2. Create a dictionary or array that contains the Struct you defined.**

```
pub contract NgaioContract {

    pub var ngaioArmy: [Ngaio]

    pub struct Ngaio {
        pub let birthday: String

        // You have to pass in 4 arguments when creating this Struct.
        init(_birthday: String) {
            self.birthday = _birthday
        }
    }

    init() {
      self.ngaioArmy = []
    }

}
```
**3. Create a function to add to that array/dictionary.**

```
pub contract NgaioContract {

    pub var ngaioArmy: [Ngaio]

    pub struct Ngaio {
        pub let birthday: String

        // You have to pass in 4 arguments when creating this Struct.
        init(_birthday: String) {
            self.birthday = _birthday
        }
    }

    pub fun cloneNgaio(birthday: String) {
      self.ngaioArmy.append(Ngaio(_birthday: birthday))
    }

    init() {
      self.ngaioArmy = []
    }

}
```

**4. Add a transaction to call that function in step 3.**
```
import NgaioContract from 0x01

transaction() {

    prepare(signer: AuthAccount) {}

    execute {
        NgaioContract.cloneNgaio(birthday: "04/12/1995")
    }
}
```
**5. Add a script to read the Struct you defined.**
```
import NgaioContract from 0x01

pub fun main(): [NgaioContract.Ngaio] {
    return NgaioContract.ngaioArmy
}
```

## Chapter 3

### Day 1

**1. In words, list 3 reasons why structs are different from resources.**

Resources cannot be copied. They cannot be lost (or overwritten). They cannot be created whenever you want (can be created only inside smart contracts, not transactions or scripts).

**2. Describe a situation where a resource might be better to use than a struct.**

If you want to buy an expensive virtual item over Flow, you would want to use a resource because a resource cannot be copied. Using a resource for the expensive virtual item ensures that only one instance of it will exist at a time. This way, after I buy the item, I know that the seller can't just re-sell it to other folks.

**3. What is the keyword to make a new resource?**

`create`

**4. Can a resource be created in a script or transaction (assuming there isn't a public function to create one)?**

Nope, only in a smart contract.

**5. What is the type of the resource below?**
```
pub resource Jacob {

}
```

The type is `@Jacob`

**6. Let's play the "I Spy" game from when we were kids. I Spy 4 things wrong with this code. Please fix them.**

```
pub contract Test {
    // Hint: There's nothing wrong here ;)
    pub resource Jacob {
        pub let rocks: Bool
        init() {
            self.rocks = true
        }
    }

    pub fun createJacob(): @Jacob { // there is 1 here
        let myJacob <- create Jacob() // there are 2 here
        return <- myJacob // there is 1 here
    }
}
```
### Day 2

**1. Write your own smart contract that contains two state variables: an array of resources, and a dictionary of resources. Add functions to remove and add to each of them. They must be different from the examples above.**

```
pub contract NgaioIsALoser {
    pub var dict: @{Int: Ngaio}
    pub var arr: @[Ngaio]

    pub resource Ngaio {
        pub let id: Int
        pub let sucks: Bool
        init(_id: Int, _sucks: Bool) {
            self.id = _id
            self.sucks = _sucks
        }
    }

    pub fun addDict(ngaio: @Ngaio) {
        let key = ngaio.id
        let oldNgaio <- self.dict[key] <- ngaio
        destroy oldNgaio
    }

    pub fun removeDict(id: Int): @Ngaio {
        let ngaio <- self.dict.remove(key: id) ?? panic("Could not find Ngaio")
        return <- ngaio
    }

    pub fun addArr(ngaio: @Ngaio) {
        self.arr.append(<- ngaio)
    }

    pub fun removeArr(index: Int): @Ngaio {
        return <- self.arr.remove(at: index)
    }

    init() {
      self.dict <- {}
      self.arr <- []
    }
}
```

### Day 3

**1. Define your own contract that stores a dictionary of resources. Add a function to get a reference to one of the resources in the dictionary.**

```
pub contract NgaioSucks {

    pub var dict: @{Int: Ngaio}

    pub resource Ngaio {
        pub let id: Int
        pub let sucks: Bool
        init(_id: Int, _sucks: Bool) {
            self.id = _id
            self.sucks = _sucks
        }
    }

    pub fun getReference(id: Int): &Ngaio? {
        return &self.dict[id] as &Ngaio?
    }

    init() {
        self.dict <- {
            0: <- create Ngaio(_id: 0, _sucks: true), 
            1: <- create Ngaio(_id: 1, _sucks: true)
        }
    }
}
```

**2. Create a script that reads information from that resource using the reference from the function you defined in part 1.**

```
import NgaioSucks from 0x01

pub fun main(): &NgaioSucks.Ngaio {
    return NgaioSucks.getReference(id: 0)!
}
```

**3. Explain, in your own words, why references can be useful in Cadence.**

To create NFTs! A resource is a unique item on the Blockchain that cannot be copied/duplicated/faked.

### Day 4

**1. Explain, in your own words, the 2 things resource interfaces can be used for (we went over both in today's content)**

A resource interface can be used as a "requirement" that specifies the functions and fields that an interface must have.
A resource interface can restrict access of a resource's functions and fields from certain people.

**2. Define your own contract. Make your own resource interface and a resource that implements the interface. Create 2 functions. In the 1st function, show an example of not restricting the type of the resource and accessing its content. In the 2nd function, show an example of restricting the type of the resource and NOT being able to access its content.**

```
pub contract NgaioContract {

    pub resource interface INgaio {
      pub let sucks: Bool
    }

    pub resource Ngaio: INgaio {
      pub let sucks: Bool
      pub let age: Int
      init() {
        self.sucks = false
        self.age = 0
      }
    }

    // can access age field
    pub fun noInterface() {
      let me: @Ngaio <- create Ngaio()
      log(me.age)

      destroy me
    }

    // cannot access age field bc it is not exposed in INgaio interface
    pub fun yesInterface() {
      let me: @Ngaio{INgaio} <- create Ngaio()
      log(me.age)

      destroy me
    }
}
```

**3. How would we fix this code?**
```
pub contract Stuff {

    pub struct interface ITest {
      pub var greeting: String
      pub var favouriteFruit: String
      pub fun changeGreeting(newGreeting: String): String
    }

    // ERROR:
    // `structure Stuff.Test does not conform 
    // to structure interface Stuff.ITest`
    pub struct Test: ITest {
      pub var greeting: String
      pub var favouriteFruit: String

      pub fun changeGreeting(newGreeting: String): String {
        self.greeting = newGreeting
        return self.greeting // returns the new greeting
      }

      init() {
        self.greeting = "Hello!"
        self.favouriteFruit = "Kiwi"
      }
    }

    pub fun fixThis() {
      let test: Test{ITest} = Test()
      let newGreeting = test.changeGreeting(newGreeting: "Bonjour!") // ERROR HERE: `member of restricted type is not accessible: changeGreeting`
      log(newGreeting)
    }
}
```

### Day 5

For today's quest, you will be looking at a contract and a script. You will be looking at 4 variables (a, b, c, d) and 3 functions (publicFunc, contractFunc, privateFunc) defined in SomeContract. In each AREA (1, 2, 3, and 4), I want you to do the following: for each variable (a, b, c, and d), tell me in which areas they can be read (read scope) and which areas they can be modified (write scope). For each function (publicFunc, contractFunc, and privateFunc), simply tell me where they can be called.

1.
   - Read: a, b, c, d
   - Write: a, b, c, d
   - Functions: publicFunc, contractFunc, privateFunc
2.
   - Read: a, b, c
   - Write: a
   - Functions: publicFunc, contractFunc
3.
   - Read: a, b, c
   - Write: a
   - Functions: publicFunc, contractFunc
4.
   - Read: a, b
   - Write:
   - Functions: publicFunc

```
access(all) contract SomeContract {
    pub var testStruct: SomeStruct

    pub struct SomeStruct {

        //
        // 4 Variables
        //

        pub(set) var a: String

        pub var b: String

        access(contract) var c: String

        access(self) var d: String

        //
        // 3 Functions
        //

        pub fun publicFunc() {}

        access(contract) fun contractFunc() {}

        access(self) fun privateFunc() {}


        pub fun structFunc() {
            /**************/
            /*** AREA 1 ***/
            /**************/
        }

        init() {
            self.a = "a"
            self.b = "b"
            self.c = "c"
            self.d = "d"
        }
    }

    pub resource SomeResource {
        pub var e: Int

        pub fun resourceFunc() {
            /**************/
            /*** AREA 2 ***/
            /**************/
        }

        init() {
            self.e = 17
        }
    }

    pub fun createSomeResource(): @SomeResource {
        return <- create SomeResource()
    }

    pub fun questsAreFun() {
        /**************/
        /*** AREA 3 ****/
        /**************/
    }

    init() {
        self.testStruct = SomeStruct()
    }
}
```
This is a script that imports the contract above:
```
import SomeContract from 0x01

pub fun main() {
  /**************/
  /*** AREA 4 ***/
  /**************/
}
```

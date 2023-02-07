---
title: State Variables & Integers
actions: ['checkAnswer', 'hints']
material:
  editor:
    language: rust
    startingCode: |
      #![no_std]

      multiversx_sc::imports!();
      multiversx_sc::derive_imports!();

      #[multiversx_sc::contract]
      pub trait ZombieFactory {

        #[init]
        fn init(&self) {
          // set up your storage value here
        }

        // write your storage function here

      }

      
    answer: >
      #![no_std]

      multiversx_sc::imports!();
      multiversx_sc::derive_imports!();

      #[multiversx_sc::contract]
      pub trait ZombieFactory {

        #[init]
        fn init(&self) {
          self.dna_digits().set(16u8);
        }

        #[storage_mapper("dna_digits")]
        fn dna_digits(&self) -> SingleValueMapper<u8>;
      }
---

Great job! Now that we've got a shell for our contract, let's learn about how Rust deals with variables.

**_State variables_** are permanently stored in contract storage. This means they're written to the MultiversX blockchain. Think of them like writing to a DB. The **MultiversX** Rust framework provides various storage mappers you can use. Deciding which one to use for every situation is critical for performance.

For this step we will use a `SingleValueMapper`, but feel free to check the documentation https://docs.multiversx.com/developers/developer-reference/storage-mappers/ if curious about other types.

##### Example:
```
#[mx_sc::contract]
pub trait Example {

  #[init]
  fn init(&self) {
    self.my_size().set(100u32);
  }

  #[storage_mapper("my_size")]
  fn my_size(&self) -> SingleValueMapper<u32>;
}
```

In this example contract, we created a storage for a `u32` named `my_size` and than set the value of it to 100. Since we want this to be sent when the contract is deployed we will set it up inside our init.

> Note: You can easy cast any number to a type by adding the type to the end of the number. In this case `100u32` is 100 casted as a `u32`.

## Procedural Macros

You can spot out that we have `#[storage_mapper("dna_digits")]` which is something we call a procedural macro. This element adds on compile a body to the function that defines its behaviour as a storage mapper. Though they look scary, storages in the MultiversX framework are actually simple and are defined by basically a function declarations with the proper procedural macro above it.

## Unsigned Integers: `u8`

The `u8` data type is a 8-bit unsigned integer, meaning **its value must be non-negative**. 

> Note: You can declare unsigned integer types with more bits — `u16`, `u32`, `u64` etc.. But in this case you want to simply use `u8` because we want to store in it a small number representing the number of digits of the zombie dna.

There are also a couple integer data type for signed integers such as `i8`, `i16`, `i32`, etc..

# Put it to the test

Our Zombie DNA is going to be determined by a 16-digit number.

Write a storage mapper of type `SingleValueMapper<u32>` named `dna_digits`, and set it equal to `16`.
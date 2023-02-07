---
title: Keccak256 and Typecasting
actions: ['checkAnswer', 'hints']
material:
  editor:
    language: rust
    startingCode: |
      #![no_std]

      multiversx_sc::imports!();
      multiversx_sc::derive_imports!();

      #[derive(NestedEncode, NestedDecode, TopEncode, TopDecode, TypeAbi)]
      pub struct Zombie<M: ManagedTypeApi> {
          name: ManagedBuffer<M>,
          dna: u64,
      }

      #[multiversx_sc::contract]
      pub trait ZombieFactory {

        #[init]
        fn init(&self) {
          self.dna_digits().set(16u8);
        }

        fn create_zombie(&self, name: ManagedBuffer, dna: u64) {
            self.zombies().insert(Zombie { name, dna });
        }

        #[view]
        fn generate_random_dna(&self) -> u64{
            // start here
        }

        #[endpoint]
        fn create_random_zombie(&self, name: ManagedBuffer){

        }

        #[view]
        #[storage_mapper("dna_digits")]
        fn dna_digits(&self) -> SingleValueMapper<u8>;

        #[view]
        #[storage_mapper("zombies")]
        fn zombies(&self) -> UnorderedSetMapper<Zombie<Self::Api>>;
      }
    answer: >
      #![no_std]

      multiversx_sc::imports!();
      multiversx_sc::derive_imports!();

      #[derive(NestedEncode, NestedDecode, TopEncode, TopDecode, TypeAbi)]
      pub struct Zombie<M: ManagedTypeApi> {
          name: ManagedBuffer<M>,
          dna: u64,
      }

      #[multiversx_sc::contract]
      pub trait ZombieFactory {

        #[init]
        fn init(&self) {
          self.dna_digits().set(16u8);
        }

        fn create_zombie(&self, name: ManagedBuffer, dna: u64) {
            self.zombies().insert(Zombie { name, dna });
        }

        #[view]
        fn generate_random_dna(&self) -> u64{
            let rand_source = RandomnessSource::new();
            let dna_digits = self.dna_digits().get();
            let max_dna_value = u64::pow(10u64, dna_digits as u32);
            rand_source.next_u64_in_range(0u64, max_dna_value)
        }

        #[endpoint]
        fn create_random_zombie(&self, name: ManagedBuffer){

        }

        #[view]
        #[storage_mapper("dna_digits")]
        fn dna_digits(&self) -> SingleValueMapper<u8>;

        #[view]
        #[storage_mapper("zombies")]
        fn zombies(&self) -> UnorderedSetMapper<Zombie<Self::Api>>;
      }
---


## Math functions 


Math in Rust is pretty straightforward. The following operations are the same as in most programming languages:

* Addition: `x + y`
* Subtraction: `x - y`,
* Multiplication: `x * y`
* Division: `x / y`
* Modulus / remainder: `x % y` _(for example, `13 % 5` is `3`, because if you divide 5 into 13, 3 is the remainder)_

For raising to a power the operation is done straigth up from the numeric type method `pow` :

```
let x = u32::pow(5, 2); // equal to 5^2 = 25
```

This can alternatively be written

```
let x = 5u32.pow(2); // equal to 5^2 = 25
```

## Random generation

We want our `generate_random_dna` function to return a random `u64`. How can we accomplish this?

Randomness in the blockchain environment is a challenging task to accomplish. Due to the nature of the environemnt, nodes must all have the same "random" generator to be able to reach consensus. This is solved by using Golang's standard seeded random number generator, directly in the VM: https://cs.opensource.google/go/go/+/refs/tags/go1.17.5:src/math/rand/

```
let mut rand_source = RandomnessSource::new();
let my_rand_u16 = rand_source.next_u16();
```
By what you might guess `RandomnessSource` comes with a function for each numerical type in Rust.

The `RandomnessSource` struct also provides methods for generating numbers within a range, namely the `fn next_usize_in_range(min: usize, max: usize) -> usize`, which generates a random usize in the `[min, max)` range. These methods are available for the rest of the numerical types as well, but for this example, we need usize (in Rust, indexes are usize).
```
let vec_len = my_vec.len();
let mut rand_source = RandomnessSource::new();
for i in 0..vec_len {
    let rand_index = rand_source.next_usize_in_range(i, vec_len);
    let first_item = my_vec.get(i).unwrap();
    let second_item = my_vec.get(rand_index).unwrap();

    my_vec.set(i, &second_item);
    my_vec.set(rand_index, &first_item);
}
```

## Casting types

Most of the numeric types in rust are easy castable between them

```
let my_u32 = 42u32;
let my_u64 = my_u32 as u64;
```

# Put it to the test

Let's fill in the body of our `generate_random_dna` function! Here's what it should do:

1. The first line of code should contain the instantiation of a `RandomnessSource` object.

2. Follwoingwe need to get dna_digits out of the storage in order to use it.

3. Since we will be using `next_u64_in_range` we will need to specify the range of our dna number which will be [0; 10^16). Note that the exponent of the power function needs to be u32 so a cast will need to be added.
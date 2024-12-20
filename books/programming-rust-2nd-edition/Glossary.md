# Glossary

## Type Associated functions

- functions associated with specific type, but doesn't operate on na instance of that type

```rust 
struct Person {}

impl Person {
    fn new() -> Person {
        // ... 
    }
}

fn main() {
    Person::new() // calling associated functions  
}
```

- Event traits can have associated functions

```rust 
trait StringSet {
    /// Return a new empty set.
    fn new() -> Self;

    /// Return a set that contains all the strings in `strings`.
    fn from_slice(strings: &[&str]) -> Self;

    /// Find out if this set contains a particular `value`.
    fn contains(&self, string: &str) -> bool;

    /// Add a string to this set.
    fn add(&mut self, string: &str);
}
```

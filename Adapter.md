## How it works?
1. The adapter gets an interface, compatible with one of the existing objects. 
2. Using this interface, the existing object can safely call the adapter’s methods. 
3. Upon receiving a call, the adapter passes the request to the second object, but in a format and order that the second object expects.

![[Pasted image 20221019193852.png]]

## Structure

![[Pasted image 20221019194501.png]]

1. The Client is a class that contains the existing business logic of the program. 
2. The Client Interface describes a protocol that other classes must follow to be able to collaborate with the client code. 
3. The Service is some useful class (usually 3rd-party or legacy). The client can’t use this class directly because it has an incompatible interface. 
4. The Adapter is a class that’s able to work with both the client and the service: it implements the client interface, while wrapping the service object. The adapter receives calls from the client via the adapter interface and translates them into calls to the wrapped service object in a format it can understand. 
5. The client code doesn’t get coupled to the concrete adapter class as long as it works with the adapter via the client interface. Thanks to this, you can introduce new types of adapters into the program without breaking the existing client code. This can be useful when the interface of the service class gets changed or replaced: you can just create a new adapter class without changing the client code.

## How to Implement

1. Make sure that you have at least two classes with incompatible interfaces: 
	- A useful service class, which you can’t change (often 3rdparty, legacy or with lots of existing dependencies). 
	- One or several client classes that would benefit from using the service class.
2. Declare the client interface and describe how clients communicate with the service.
3. Create the adapter class and make it follow the client interface. Leave all the methods empty for now. 
4. Add a field to the adapter class to store a reference to the service object. The common practice is to initialize this field via the constructor, but sometimes it’s more convenient to pass it to the adapter when calling its methods. 
5. One by one, implement all methods of the client interface in the adapter class. The adapter should delegate most of the real work to the service object, handling only the interface or data format conversion. 
6. Clients should use the adapter via the client interface. This will let you change or extend the adapters without affecting the client code.

## How to implement in [[Rust]] language

In this example, the `trait SpecificTarget` is incompatible with a `call` function which accepts `trait Target` only.

```Rust
fn call(target: impl Target);
```

The adapter helps to pass the incompatible interface to the `call` function.

```Rust
let target = TargetAdapter::new(specific_target);
call(target);
```

#### **adapter.rs**
```rust
use crate::{adaptee::SpecificTarget, Target};

/// Converts adaptee's specific interface to a compatible `Target` output.
pub struct TargetAdapter {
    adaptee: SpecificTarget,
}

impl TargetAdapter {
    pub fn new(adaptee: SpecificTarget) -> Self {
        Self { adaptee }
    }
}

impl Target for TargetAdapter {
    fn request(&self) -> String {
        // Here's the "adaptation" of a specific output to a compatible output.
        self.adaptee.specific_request().chars().rev().collect()
    }
}
```

#### **adaptee.rs**
```rust
pub struct SpecificTarget;

impl SpecificTarget {
    pub fn specific_request(&self) -> String {
        ".tseuqer cificepS".into()
    }
}
```

#### **target.rs**
```rust
pub trait Target {
    fn request(&self) -> String;
}

pub struct OrdinaryTarget;

impl Target for OrdinaryTarget {
    fn request(&self) -> String {
        "Ordinary request.".into()
    }
}
```

#### **main.rs**
```rust
mod adaptee;
mod adapter;
mod target;

use adaptee::SpecificTarget;
use adapter::TargetAdapter;
use target::{OrdinaryTarget, Target};

/// Calls any object of a `Target` trait.
///
/// To understand the Adapter pattern better, imagine that this is
/// a client code, which can operate over a specific interface only
/// (`Target` trait only). It means that an incompatible interface cannot be
/// passed here without an adapter.
fn call(target: impl Target) {
    println!("'{}'", target.request());
}

fn main() {
    let target = OrdinaryTarget;

    print!("A compatible target can be directly called: ");
    call(target);

    let adaptee = SpecificTarget;

    println!(
        "Adaptee is incompatible with client: '{}'",
        adaptee.specific_request()
    );

    let adapter = TargetAdapter::new(adaptee);

    print!("But with adapter client can call its method: ");
    call(adapter);
}
```

### Output
```
A compatible target can be directly called: 'Ordinary request.'
Adaptee is incompatible with client: '.tseuqer cificepS'
But with adapter client can call its method: 'Specific request.'
```
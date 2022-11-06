*Also known as: CoR, Chain of Command*

![[Pasted image 20221106164402.png]]
1. The **Handler** declares the interface, common for all concrete handlers. It usually contains just a single method for handling requests, but sometimes it may also have another method for setting the next handler on the chain.
2. The **Base Handler** is an optional class where you can put the boilerplate code that’s common to all handler classes. Usually, this class defines a field for storing a reference to the next handler. The clients can build a chain by passing a handler to the constructor or setter of the previous handler. The class may also implement the default handling behavior: it can pass execution to the next handler after checking for its existence.
3. **Concrete Handlers** contain the actual code for processing requests. Upon receiving a request, each handler must decide whether to process it and, additionally, whether to pass it along the chain. Handlers are usually self-contained and immutable, accepting all necessary data just once via the constructor.
4. The **Client** may compose chains just once or compose them dynamically, depending on the application’s logic. Note that a request can be sent to any handler in the chain—it doesn’t have to be the first one.

![[Pasted image 20221106164906.png]]

The application’s GUI is usually structured as an object tree. For example, the Dialog class, which renders the main window of the app, would be the root of the object tree. The dialog contains Panels , which might contain other panels or simple low-level elements like Buttons and TextFields.

## How to implement in [[Rust]] language

The example demonstrates processing a patient through a chain of departments. The chain of responsibility is constructed as follows:
`Patient -> Reception -> Doctor -> Medical -> Cashier`

#### **patient.rs:** Request
```rust
#[derive(Default)]
pub struct Patient {
    pub name: String,
    pub registration_done: bool,
    pub doctor_check_up_done: bool,
    pub medicine_done: bool,
    pub payment_done: bool,
}
```

#### **department.rs:** Handlers
```rust
mod cashier;
mod doctor;
mod medical;
mod reception;

pub use cashier::Cashier;
pub use doctor::Doctor;
pub use medical::Medical;
pub use reception::Reception;

use crate::patient::Patient;

/// A single role of objects that make up a chain.
/// A typical trait implementation must have `handle` and `next` methods,
/// while `execute` is implemented by default and contains a proper chaining
/// logic.
pub trait Department {
    fn execute(&mut self, patient: &mut Patient) {
        self.handle(patient);

        if let Some(next) = &mut self.next() {
            next.execute(patient);
        }
    }

    fn handle(&mut self, patient: &mut Patient);
    fn next(&mut self) -> &mut Option<Box<dyn Department>>;
}

/// Helps to wrap an object into a boxed type.
pub(self) fn into_next(
    department: impl Department + Sized + 'static,
) -> Option<Box<dyn Department>> {
    Some(Box::new(department))
}
```

#### **department/cashier.rs**
```rust
use super::{Department, Patient};

#[derive(Default)]
pub struct Cashier {
    next: Option<Box<dyn Department>>,
}

impl Department for Cashier {
    fn handle(&mut self, patient: &mut Patient) {
        if patient.payment_done {
            println!("Payment done");
        } else {
            println!("Cashier getting money from a patient {}", patient.name);
            patient.payment_done = true;
        }
    }

    fn next(&mut self) -> &mut Option<Box<dyn Department>> {
        &mut self.next
    }
}
```

#### **department/doctor.rs**
```rust
use super::{into_next, Department, Patient};

pub struct Doctor {
    next: Option<Box<dyn Department>>,
}

impl Doctor {
    pub fn new(next: impl Department + 'static) -> Self {
        Self {
            next: into_next(next),
        }
    }
}

impl Department for Doctor {
    fn handle(&mut self, patient: &mut Patient) {
        if patient.doctor_check_up_done {
            println!("A doctor checkup is already done");
        } else {
            println!("Doctor checking a patient {}", patient.name);
            patient.doctor_check_up_done = true;
        }
    }

    fn next(&mut self) -> &mut Option<Box<dyn Department>> {
        &mut self.next
    }
}
```

#### **department/medical.rs**
```rust
use super::{into_next, Department, Patient};

pub struct Medical {
    next: Option<Box<dyn Department>>,
}

impl Medical {
    pub fn new(next: impl Department + 'static) -> Self {
        Self {
            next: into_next(next),
        }
    }
}

impl Department for Medical {
    fn handle(&mut self, patient: &mut Patient) {
        if patient.medicine_done {
            println!("Medicine is already given to a patient");
        } else {
            println!("Medical giving medicine to a patient {}", patient.name);
            patient.medicine_done = true;
        }
    }

    fn next(&mut self) -> &mut Option<Box<dyn Department>> {
        &mut self.next
    }
}
```

#### **department/reception.rs**
```rust
use super::{into_next, Department, Patient};

#[derive(Default)]
pub struct Reception {
    next: Option<Box<dyn Department>>,
}

impl Reception {
    pub fn new(next: impl Department + 'static) -> Self {
        Self {
            next: into_next(next),
        }
    }
}

impl Department for Reception {
    fn handle(&mut self, patient: &mut Patient) {
        if patient.registration_done {
            println!("Patient registration is already done");
        } else {
            println!("Reception registering a patient {}", patient.name);
            patient.registration_done = true;
        }
    }

    fn next(&mut self) -> &mut Option<Box<dyn Department>> {
        &mut self.next
    }
}
```

#### **main.rs:** Client code
```rust
mod department;
mod patient;

use department::{Cashier, Department, Doctor, Medical, Reception};
use patient::Patient;

fn main() {
    let cashier = Cashier::default();
    let medical = Medical::new(cashier);
    let doctor = Doctor::new(medical);
    let mut reception = Reception::new(doctor);

    let mut patient = Patient {
        name: "John".into(),
        ..Patient::default()
    };

    // Reception handles a patient passing him to the next link in the chain.
    // Reception -> Doctor -> Medical -> Cashier.
    reception.execute(&mut patient);

    println!("\nThe patient has been already handled:\n");

    reception.execute(&mut patient);
}
```

### Output
```
Reception registering a patient John
Doctor checking a patient John
Medical giving medicine to a patient John
Cashier getting money from a patient John

The patient has been already handled:

Patient registration is already done
A doctor checkup is already done
Medicine is already given to a patient
Payment done
```


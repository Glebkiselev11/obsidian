Using this patter makes sense only when the core model of your app can be represented as a tree.

![[Pasted image 20221022184407.png]]

The composite patter suggest that you work with same common interface.

For item it'd simply return the product's price. For a box, it'd go over each item the box contains, ask its price and then return a total for this box. If one of these items were a smaller box, that box would also start going over its contents and so on, until the prices of all inner components were calculated.

![[Pasted image 20221022185814.png]]

## Structure

![[Pasted image 20221022190120.png]]
1. The **Component** interface describes operations that are common to both simple and complex elements of the tree.
2. The **leaf** is a basic element of the tree that doesn't have sub-elements. Usually, leaf components end up doing most of the real work, since they dont have anyone to delegate their work to.
3. The **Composite** is an element that has sub-elements: leaves or other containters. A container doesn't know the concrete class of its children. It works with all sub-elements only via the composite interface. Upon receiving a request, a container delegates the work to its sub-elements, processes intermediate results and then returns the final result to the client.
4. The **Client** works with all elements through the component interface. As a result, the client can work in the same way with both simple or complex elements of the tree.

## Applicability
- Use the composite pattern when you have to implement a tree-like object structure. 
- Use the pattern when you want the client code to treat both simple and complex elements uniformly. (All elements defined by the composite pattern share a common interface. Using this interface, the client doesn't have to worry about the concrete class of the objects it works with.)

### Cons
- It might be difficult to provide a common interface for classes whose functionality differs too much. In certain scenarios, you’d need to overgeneralize the component interface, making it harder to comprehend.

## How to implement in [[Rust]] language

#### **fs/mod.rs**
```rust
mod file;
mod folder;

pub use file::File;
pub use folder::Folder;

pub trait Component {
    fn search(&self, keyword: &str);
}
```

#### **fs/file.rs**
```rust
use super::Component;

pub struct File {
    name: &'static str,
}

impl File {
    pub fn new(name: &'static str) -> Self {
        Self { name }
    }
}

impl Component for File {
    fn search(&self, keyword: &str) {
        println!("Searching for keyword {} in file {}", keyword, self.name);
    }
}
```

#### **fs/folder.rs**
```rust
use super::Component;

pub struct Folder {
    name: &'static str,
    components: Vec<Box<dyn Component>>,
}

impl Folder {
    pub fn new(name: &'static str) -> Self {
        Self {
            name,
            components: vec![],
        }
    }

    pub fn add(&mut self, component: impl Component + 'static) {
        self.components.push(Box::new(component));
    }
}

impl Component for Folder {
    fn search(&self, keyword: &str) {
        println!(
            "Searching recursively for keyword {} in folder {}",
            keyword, self.name
        );

        for component in self.components.iter() {
            component.search(keyword);
        }
    }
}
```

#### **main.rs**
```rust
mod fs;

use fs::{Component, File, Folder};

fn main() {
    let file1 = File::new("File 1");
    let file2 = File::new("File 2");
    let file3 = File::new("File 3");

    let mut folder1 = Folder::new("Folder 1");
    folder1.add(file1);

    let mut folder2 = Folder::new("Folder 2");
    folder2.add(file2);
    folder2.add(file3);
    folder2.add(folder1);

    folder2.search("rose");
}
```


### Output
```
Searching recursively for keyword rose in folder Folder 2
Searching for keyword rose in file File 2
Searching for keyword rose in file File 3
Searching recursively for keyword rose in folder Folder 1
Searching for keyword rose in file File 1
------------------------------------
```
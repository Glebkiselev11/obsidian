The Proxy pattern suggests that you create a new proxy class with the same interface as an original service object. Then you update your app so that it passes the proxy object to all of the
original object’s clients. Upon receiving a request from a client, the proxy creates a real service object and delegates all the work to it.

![[Pasted image 20221106153612.png]]
But what's the benefit? If you need to execute something either before of after the primary logic of the class, the proxy lets you do this without changing this class. Since the proxy implements the same interface as the original class, it can be passed to any client that expects a real service object. 
![[Pasted image 20221106154116.png]]

1. The **Service Interface** declares the interface of the Service. The proxy must follow this interface to be able to disguise itself as a service object.
2. The **Service** is a class that provides some useful business logic.
3. The **Proxy** class has a reference field that points to a service object. After the proxy finishes its processing (e.g., lazy initialization, logging, access control, caching, etc.), it passes the request to the service object. Usually, proxies manage the full lifecycle of their service objects.
4. The **Client** should work with both services and proxies via the same interface. This way you can pass a proxy into any code that expects a service object.

## How to implement in [[Rust]] language

### Conceptual Example: Nginx Proxy

A web server such as Nginx can act as a proxy for your application server:

-   It provides controlled access to your application server.
-   It can do rate limiting.
-   It can do request caching.


#### **server.rs**
```rust
mod application;
mod nginx;

pub use nginx::NginxServer;

pub trait Server {
    fn handle_request(&mut self, url: &str, method: &str) -> (u16, String);
}
```

####  **server/application.rs**
```rust
use super::Server;

pub struct Application;

impl Server for Application {
    fn handle_request(&mut self, url: &str, method: &str) -> (u16, String) {
        if url == "/app/status" && method == "GET" {
            return (200, "Ok".into());
        }

        if url == "/create/user" && method == "POST" {
            return (201, "User Created".into());
        }

        (404, "Not Ok".into())
    }
}
```

#### **server/nginx.rs**
```rust
use std::collections::HashMap;

use super::{application::Application, Server};

/// NGINX server is a proxy to an application server.
pub struct NginxServer {
    application: Application,
    max_allowed_requests: u32,
    rate_limiter: HashMap<String, u32>,
}

impl NginxServer {
    pub fn new() -> Self {
        Self {
            application: Application,
            max_allowed_requests: 2,
            rate_limiter: HashMap::default(),
        }
    }

    pub fn check_rate_limiting(&mut self, url: &str) -> bool {
        let rate = self.rate_limiter.entry(url.to_string()).or_insert(1);

        if *rate > self.max_allowed_requests {
            return false;
        }

        *rate += 1;
        true
    }
}

impl Server for NginxServer {
    fn handle_request(&mut self, url: &str, method: &str) -> (u16, String) {
        if !self.check_rate_limiting(url) {
            return (403, "Not Allowed".into());
        }

        self.application.handle_request(url, method)
    }
}
```

#### **main.rs**
```rust
mod server;

use crate::server::{NginxServer, Server};

fn main() {
    let app_status = &"/app/status".to_string();
    let create_user = &"/create/user".to_string();

    let mut nginx = NginxServer::new();

    let (code, body) = nginx.handle_request(app_status, "GET");
    println!("Url: {}\nHttpCode: {}\nBody: {}\n", app_status, code, body);

    let (code, body) = nginx.handle_request(app_status, "GET");
    println!("Url: {}\nHttpCode: {}\nBody: {}\n", app_status, code, body);

    let (code, body) = nginx.handle_request(app_status, "GET");
    println!("Url: {}\nHttpCode: {}\nBody: {}\n", app_status, code, body);

    let (code, body) = nginx.handle_request(create_user, "POST");
    println!("Url: {}\nHttpCode: {}\nBody: {}\n", create_user, code, body);

    let (code, body) = nginx.handle_request(create_user, "GET");
    println!("Url: {}\nHttpCode: {}\nBody: {}\n", create_user, code, body);
}
```

### Output
```
Url: /app/status
HttpCode: 200
Body: Ok

Url: /app/status
HttpCode: 200
Body: Ok

Url: /app/status
HttpCode: 403
Body: Not Allowed

Url: /create/user
HttpCode: 201
Body: User Created

Url: /create/user
HttpCode: 404
Body: Not Ok
```


---
title: "Parse Request Data"
weight: 2101
menu:
  book:
    parent: "advanced"
---

### parsed data source

The data in a request contains the following parts:

* ```Router params```: Refers to the obtained parameter list in routing path matching. For example: ```id``` in ```Router::with_path("/users/<id>/")```.
* ```URL queries```: List of parameters to parse from the URL's query string. For example ```123``` in ```http://localhost/user_id=123```.
* ```Headers```: The list of parameters in the request header.
* ```Form```: A list of parameters to submit as ````POST```.

This data can be parsed into strongly typed structures by providing several functions through ```Request```.

* ```parse_params```: parse the requested ```Router params``` into a specific data type;
* ```parse_queries```: parse the requested ```URL queries``` into a specific data type;
* ```parse_headers```: parse the requested ```Headers``` into a specific data type;
* ```parse_json```: Parse the data in the ```body``` part of the request as JSON format to a specific type;
* ```parse_form```: Parse the data in the ```body``` part of the request as a Form form to a specific type;
* ```parse_body```: Parse the data in the ```body``` section to a specific type according to the type of the requested ```content-type```.
* ```parse_data```: can combine different data sources to parse a specific type, acceptable data sources are: ```Router params```, ```URL queries```, ```Headers ```, ```Form```. To prevent unexpected errors, each data source cannot have the same key name. For example: ```http://localhost/users/<id>?id=123`` `, ```<id>``` represents the ```id``` obtained by route matching, and there is also a ```id=123``` in the ```URL queries``` part, in this case it cannot be Merged ``Router params`` and ````URL queries``` to resolve to a specific type.

### Parsing principle

The customized ```serde``` ```Deserializer``` will be pase data similar to ```HashMap<String, String>``` and ```HashMap<String, Vec<String>>``` into a specific data type.

For example: ```URL queries``` is actually parsed as a [MultiMap](https://docs.rs/multimap/latest/multimap/struct.MultiMap.html) type, ```MultiMap``` can think of it as a data structure like ```HashMap<String, Vec<String>>```. If the requested URL is ```http://localhost/users?id=123&id=234```, we provide The target type is:

```rust
#[derive(Deserialize)]
struct User {
  id: i64
}
```

Then the first ```id=123``` will be parsed, and ```id=234``` will be discarded:

```rust
let user: User = req.parse_queries().unwrap();
assert_eq!(user.id, 123);
```

If the type we provide is:

```rust
#[derive(Deserialize)]
struct Users {
  id: Vec<i64>
}
```

Then ```id=123&id=234``` will be parsed:

```rust
let users: Users = req.parse_queries().unwrap();
assert_eq!(user.ids, vec![123, 234]);
```

Multiple data sources can be merged to parse out a specific type. The first is to define the data source:

```rust
use enumflags2::make_bitflags;

let source = make_bitflags!(ParseSource::{Queries|Form});
```

The data parts of ```URL queries``` and ```Form``` are combined here, and then call ```parse_data``` to parse. For specific examples, see: [parse-data](https:/ /github.com/salvo-rs/salvo/blob/main/examples/parse-data/src/main.rs).
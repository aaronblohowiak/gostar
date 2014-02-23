go* (gostar)
=======

Generics in Go through simple AST manipulation.

Goals
=======
* No changes to existing go compiler or toolchain
* Generics should be easily testable
* Generics should be go-gettable & backwards compatible
* No runtime cost
* Easy to debug

Use
=======

2 minute intro [link to screencast]


Let's say you have a package, `person`, and you want to export a `List` type which is a linked-list of people. 

Add a file named list.gos to your person package directory, containing:

`
  specialize "github.com/aaronblohowiak/containers/list.go"
  package person
  ValueType -> Person
`

Make sure gostar is running and its export path is in your GOPATH. Then, run godoc on your package.

Package `person` will now export a `List` type that is a doubly linked-list of people. What we've done is replaced the list's package declaration to be `person` and we have replaced the type `ValueType` with `Person` everywhere it existed in the list source.

No messy temporary files, no compiler or toolchain changes.


Methodology
=======

gostar is a syntax-aware preprocessor that leverages FUSE so it can be used without modifications to the existing toolchain.

The builtin container/list is specified with `interface{}` as the value type for the linked-list nodes and the return type of many of the functions. With gostar you can replace `interface{}` with your explicit type name and include the resulting implementation in your package. Now, you can eliminate all type conversions and get the benefits of static type analysis.

Advantages
=======

gostar 'templates' are valid go source files and can/should be tested on their own. Additionally, since they must be valid go source files, they are usable on their own as well. This means you can write gostar-friendly generic code that is usable by people who have not adopted gostar!

You can also use gostar to rewrite code that was not intended to be used generically.

Since we're frequently replacing `interface{}` with the specific type name, we can reclaim compile-time type checking with reusable code.

Since we're leveraging FUSE, inspecting the generated code is as easy as cat'ing the file.


Limitations
=======

Currently, gostar is only able to replace types a single file at a time.


This implementation of generics can lead to larger compiled source size, as makes copies of the generic code instead of re-using it. If your program is large enough and the same generic code is reused frequently enough, this could lead to worse runtime performance by thrashing the cpu cache.

Gostar does not offer C++ style template specialization. While gostar can be used to define new types that your package exports, it cannot be used to create new packages on-the-fly.

gostar currently only works on FUSE-compatible operating systems (OSX/*nix. sorry, windows.)


Previous Work
=======

gotgo was a brilliant attempt at generics by generating type-specialized versions of packages. Unfortunately, it turns out that most of what you want generics for is to reuse implementations *within* a package instead of importing a separate package that has been specialized. From the author of gotgo: https://groups.google.com/forum/#!msg/golang-nuts/auLWUqourlA/VkZ10YJ0rBoJ



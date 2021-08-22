# is_a?

The pseudo-method `is_a?` determines whether an expression's runtime type is a subtype of another type. For example:

```crystal
a = 1
a.is_a?(Int32)          # => true
a.is_a?(String)         # => false
a.is_a?(Number)         # => true
a.is_a?(Int32 | String) # => true
```

It is a pseudo-method because the compiler knows about it and it can affect type information, as explained in [if var.is_a?(...)](if_varis_a.md). Also, it accepts a [type](type_grammar.md) that must be known at compile-time as its argument.

## Subtyping relation

If any value of type `T` can be used in a context that expects a value of type `U`, we say that `T` is a subtype of `U`. The subtyping relation fully defines the type hierarchy in Crystal, and is available as either [`Class#<=`](https://crystal-lang.org/api/master/Class.html#%3C=(other:T.class):BoolforallT-instance-method) or [`TypeNode#<=`](https://crystal-lang.org/api/master/Crystal/Macros/TypeNode.html#%3C=(other:TypeNode):BoolLiteral-instance-method). The following are the cases where the subtyping relation holds:

* Idempotency: every type is a subtype of itself.

  ```crystal
  1.is_a?(Int32)   # => true
  "".is_a?(String) # => true
  ```

* Transitivity: if `T` is a subtype of `U` and `U` is a subtype of `V`, then `T` is also a subtype of `V`.

  ```crystal
  # `Array(Int32)` is a subtype of `Indexable(Int32)`, and
  # `Indexable(Int32)` is a subtype of `Indexable`
  [1, 2].is_a?(Indexable) # => true
  ```

* A class or struct type is a subtype of its superclass.

  ```crystal
  class Foo
  end

  class Bar < Foo
  end

  Bar.new.is_a?(Foo)       # => true
  Foo.new.is_a?(Reference) # => true
  ```

* A type is a subtype of the modules it includes. A class type is a subtype of the modules its instance type extends.

  ```crystal
  module Foo
  end

  class Bar
    include Foo
    extend Foo
  end

  Bar.new.is_a?(Foo) # => true
  Bar.is_a?(Foo)     # => true
  ```

* A generic instance type is a subtype of its corresponding uninstantiated generic type.

  ```crystal
  [1, 2].is_a?(Array)                  # => true
  {"foo" => 3, "bar" => 4}.is_a?(Hash) # => true
  ```

* If `T` and `U` are instances of `Tuple`, then `T` is a subtype of `U` if the two types have the same number of elements and each element type in `T` is a subtype of the corresponding element type in `U`.

  ```crystal
  {1, [""]}.is_a?({Int32 | String, Enumerable(String)}) # => true
  {1, 2}.is_a?({Int32})                                 # => false
  ```

* If `T` and `U` are instances of `NamedTuple`, then `T` is a subtype of `U` if the two types have the same keys and each value type in `T` is a subtype of the corresponding value type in `U`.

  ```crystal
  {x: 1, y: [""]}.is_a?({x: Int32 | String, y: Enumerable(String)}) # => true
  {x: 1, y: 2}.is_a?({a: Int32})                                    # => false

  # The order of keys does not matter
  {x: 'a', y: 'b'}.is_a?({y: Char, x: Char}) # => true
  ```

* A type `T` is a subtype of a union if `T` is a subtype of at least one variant type of that union.

  ```crystal
  1.is_a?(Int32 | String) # => true
  1.is_a?(Int32 | Char)   # => true
  1.is_a?(Char | String)  # => false
  ```

* A union type is a subtype of type `T` if all variant types of that union are subtypes of `T`.

  ```crystal
  (1 || 2_u8).is_a?(Int)   # => true
  (1 || 2_u8).is_a?(Int32) # => false

  # `Array(Int32)` is a subtype of `Enumerable(Int32)`
  # `Hash(Int32, Int32)` is a subtype of `Enumerable({Int32, Int32})`
  ([1, 2] || {3 => 4}).is_a?(Enumerable(Int32) | Enumerable({Int32, Int32})) # => true
  ```

* An alias type is a subtype of the type it refers to, and vice-versa.

  ```crystal
  alias Foo = Int32
  Foo.new(2).is_a?(Int32) # => true
  1.is_a?(Foo)            # => true
  ```

* `T.class` is a subtype of `U.class` if `T` is a subtype of `U`.

  ```crystal
  Int32.is_a?(Int.class)        # => true
  String.is_a?(Reference.class) # => true
  ```

* [`NoReturn`](return_types.md#noreturn-return-type) is a subtype of any type.

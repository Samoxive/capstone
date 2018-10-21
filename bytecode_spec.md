Unnamed language is a stack machine, it has a stack like python, it is a hashmap with keys as variable names.

```rust
enum Inst {
    PushInt(name: String, value: i32),
    PushFloat(name: String, value: f32),
    PushBoolean(name: String, value: bool),
    PopObjectValue(pop_to_name: String, object_name: String, key_name: String),
    PushObjectValue(object_name: String, key_name: String, value_name: String),
    Call(name: String, arguments: Vec<String>, this: Option<String>),
    PushCallResult(name: String),
    Pop(name: String),
    Label(name: String),
    GoTo(name: String), // here be dragons
    Branch(name: String, true_label: Option<String>, false_label: Option<String>),
    Return(name: String)
}
```

An example code would get converted to this list of instructions.

```
fibonacci = fn(n) {
    if (n == 0 || n == 1) {
        return n;
    } else {
        return fibonacci(n - 1) + fibonacci(n - 2);
    }
}
```

```rust
PushInt("0", 0)
PushInt("1", 1)
PopObjectValue("n#__eq__", "n", "__eq__")
Call("n#__eq__", vec!["0"], "n")
PushCallResult("eq_0")
Call("n#__eq__", vec!["1"], "n")
PushCallResult("eq_1")
PopObjectValue("eq_0#__or__", "eq_0", "__or__")
Call("eq_0#__or__", vec!["eq_1"], "eq_0")
PushCallResult("if_0_result")
Branch("if_0_result", "terminate", "recurse")

Label("terminate")
Return("n")

Label("recurse")
PushInt("2", 2)
PopObjectValue("n#__sub__", "n", "__sub__")
Call("n#__sub__", vec!["1"], "n")
PushCallResult("n_minus_1")
Call("n#__sub__", vec!["2"], "n")
PushCallResult("n_minus_2")
Call("fibonacci", vec!["n_minus_1"], None)
PushCallResult("fib_n_minus_1")
Call("fibonacci", vec!["n_minus_2"], None)
PushCallResult("fib_n_minus_2")
PopObjectValue("fib_n_minus_1#__add__", "fib_n_minus_1", "__add__")
Call("fib_n_minus_1#__add__", vec!["fib_n_minus_2"], "fib_n_minus_1")
PushCallResult("result")
Return("result")
```

Unnamed language is a stack machine, it has a stack like python, it is a hashmap with keys as variable names.

```rust
enum Inst {
    PushInt(name: String, value: i32),
    PushFloat(name: String, value: f32),
    PushBoolean(name: String, value: bool),
    PushFunction(name: String, instructions: Vec<Inst>),
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

```rust
type InstBlock = Vec<Inst>;
type Stack = Arc<HashMap<String, Value>>; // reference counting pointer to a hash map

enum Value {
    Boolean(bool),
    Integer(i64),
    Float(f64),
    String(String),
    List(Vec<Value>),
    Object(HashMap<String, Value>),
    Function
}

struct ExecutionContext {
    program_counter: usize;
    label_points: HashMap<String, usize>;
    program: InstBlock;
    stack: Stack;
    parent_context: &ExecutionContext;
    call_result: Option<Value>;
}

```

Variables shall be reference counted, when an execution context is done, it should clean up its stack.

Contexes can be stored in event loops or in other contexes, when a function call occurs, it's the callee's responsibility to revive the calling function, in which case callee can place a call result value into caller's context and resume it.

An example code would get converted to this list of instructions.
```
fibonacci = fn(n) {
    if (n == 0 || n == 1) {
        return n;
    } else {
        return fibonacci(n - 1) + fibonacci(n - 2);
    }
};
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

```
// sum numbers from 1 to n
n = 5;
sum = 0;
i = 0;
while (i < n) {
    a = i + 1;
    sum = sum + a;
}
print(sum);
```

```rust
PushInteger("n", 5)
PushInteger("sum", 0)
PushInteger("i", 0)

Label("loop")
PopObjectValue("i#__lt__", "i", "__lt__")
Call("i#__lt__", vec!["n"], "i")
PushCallResult("i_comp")
Branch("i_comp", "terminate_loop", None)
PopObjectValue("i#__add__", "i", "__add__")
PushInteger("1", 1)
Call("i#__add__", vec!["1"], "i")
PushCallResult("a")
PopObjectValue("sum#__add__", "sum", "__add__")
Call("sum#__add__", vec!["a"], "sum")
PushCallResult("sum")
GoTo("loop")

Label("terminate_loop")
Call("print", vec!["sum"], None);
```

```
// x = [1, 2, 3, 4];
for (i : x) {
    print(i);
}
```

```rust
PopObjectValue("x#__iter__", "x", "__iter__")
Call("x#__iter__", vec![], "x")
PushCallResult("x_iterator")
PopObjectValue("x_iterator#next", "x_iterator", "next")
PopObjectValue("x_iterator#hasNext", "x_iterator", "hasNext")

Label("loop")
Call("x_iterator#hasNext", vec![], "x_iterator")
PushCallResult("x_iterator_hasNext_result")
Branch("x_iterator_hasNext_result", None, "terminate_loop")
Call("x_iterator#next", vec![], "x_iterator")
PushCallResult("i")
Call("print", vec!["i"], None)
GoTo("loop")

Label("terminate_loop")
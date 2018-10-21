Unnamed Language
===
The language shall be dynamically typed, with python-like variable model.


Variables
---------
```
foo = 3;
bar = 'hello';
pi = 3.14
isLanguage = true;
isDelicious = (pi == 3.14 && isLanguage)
// || for `or` operations
```

Branching
---------
Only bools can be used as predicates.
```
if (foo == 3) {
    doThing();
} else if (foo == 4) {
    doIt();
} else {
    doOtherThing();
}
```

Loops
-----
```
for (element : list) {
    processElement(element);
}
```

```
i = 0
while (i >= 0) {
    i = i + 1;
}
```

Functions
---------
Functions are first class citizens, they are just another value.

```
sum = fn(x, y) {
    return x + y;
}

sumWith5 = fn(x) {
    return sum(5, x);
}

currySum = fn(x) {
    return fn(y) {
        return x + y;
    }
}

currySumWith5 = curry(5);
```

Collections
-----------
Lists are dynamically sized containers that can contain any value.
```
numberList = [1, 2, 3, 4]
```

Objects are key-value maps that can have a key of string value and value of any value.
```
myObject = {
    hello: 'world',
    foo: 'bar',
    uno: 1,
    random: fn() {
        return 4;
    },
    printFoo: fn() {
        print(this.foo);
    }
};

myObject.uno = 2;
print(myObject.foo);
myObject.printFoo();
```

Calling an object's property will set `this` to the object.

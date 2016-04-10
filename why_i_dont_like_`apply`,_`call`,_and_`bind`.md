# Why I don't like `apply`, `call`, and `bind`

When I was a JS beginner, it confused me a lot when seeing `apply`, `call` and `bind`.
In fact, most developers from strong-typed languages would do, 
no matter he came from Imperative styled languages or functional ones.

To be honest, I don't understand why they use `apply`, `call` and `bind`. 
And I avoid using them, always if I could.

In most occasions, __choosing such functions means you're practicing an anti-pattern.__

###What do these functions do?
They set the `this` value for a function. For example:
```javascript
var max = {
 name: 'Max',
 introduce: function () {
   return 'wow, I am ' + this.name
 }
}

var tom = {
 name: 'Tom',
 introduce: function () {
  return 'hi, I am ' + this.name
 }
}

max.introduce() // wow, I am Max
tom.introduce() // hi, I am Tom

max.introduce.apply(bar) //wow, I am Tom
```

Real world magic, isn't it?

If you have a `tim` who has a `name` but no `introduce` function. 
How can `tim` make a self-introduction?
```javascript
var tim = {
  name: 'Tim'
}

max.introduce.apply(tim) // wow, I am Tim
```

And `call` and `bind` just work similar way. You can find them on MDN.

###Everything works well, but why?
Many developers take it for granted to do such hacks when they need to "borrow" a method from another Object. 

__But why we do it this way?__

If you want to share a method, the usual solution is "inherit" rather than "borrow".

In Javascript, you can do it this way:
```javascript
var tim = {
 name: 'Tim'
}
tim.__proto__ = max

tim.introduce() // wow, I am Tim
```

It seems OK. But there are cases `inherit` makes it painful.

* `max` has dozens of methods and props;
* `tim` has already had its `__proto__` to another object
* You don't want to append the method to `tim`

Multiple inheritance is always painful,


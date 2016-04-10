# Why I don't like `apply`, `call`, and `bind`

When I was a JS beginner, it confused me a lot with `apply`, `call` and `bind`.
In fact, most developers from strong-typed languages would do.
No matter they came from Imperative styled languages or functional ones.

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

Are those Sufficient Conditions for using `apply`, `bind` and `call` for reusing?

### What is it actually about when you set `this`?
What do developers of strong-typed languages do in cases above, since they don't have `apply`?

Actually, they do have their own `apply`. It works differently, but it does exactly what `apply` enables JS hackers. 
Yes, __they use type-casting before they "borrow" a method__. (There seems to be some bad smell, right?)

I'm not crazy, we use type-casting and apply for the same purpose and get similar result:

>We tell the compiler to treat an instance of A as B without check. 

Type-casting would raise run time exceptions if they fail, which is hard to figure before they happen.

If you can assure the conversion will succeed, it's OK to use type-casting. But you have better choices. That's why they say "type-casting is bad".

Things would go even worse with `apply`, because it often depends on concrete implementations.
That is, we know `introduce` calls a `name` property rather than `_name` in the above example. Then, What will happen if someone changes `max.introduce` to call `$name`?

### What's your alternative solution if you do not cast?
In my opinion, that's why they introduce the concept of `interface` in Java and C#.
An `interface` works as a contract to define what fields an object should have.

That means, if `tim` and `max` implement the same `interface`, they should share some common fields. 
For example (C# styled code), if I define a interface:
```csharp
interface IPerson {
 string name;
}
```
And declare:
```csharp
class Max: IPerson {
 // name is required, or an error would raise in compile time
 public string name = "max"; 
}

class Tim: IPerson {
 public string name = "tim";
}
```
You do know an instance of Max has a field `name`.

And if I add a method to `IPerson`:
```csharp
interface IPerson {
 string name;
 string introduce();
}
```
Max and Tim should each have an `introduce` method returns a `string`, or the compiler will raise a compile error.Then we just need to call `tim.introduce()`.

Or I can use a static method as generic introduction:
```csharp
public static string introduce(IPerson person){
 return "Hey, I am " + person.name;
}
```
It is sure to work well on both `max` and `tim`, for they both implement `IPerson`.

### It is cool! But JS do not support interface!
That's why they re-invented C# and named it Typescript (just a joke :))
 
To some degree, I think weak-typed means "we rely on our developers to make thing correct". 
But human beings are just not that reliable when faced with tremendous complexity. Thus in the past tens of years, they build software engineering, design patterns and variety of tools. All aims at reducing complexity in software development.

Interface is one of them. __The Zen of Interface is, to rely on contract rather than implementation.__

The adoption of `apply` acts just opposite to Interface: 
> You know you can `apply` `introduce` to `tim`, 
> because you know the inner logic of `introduce` and the inner structure of `tim`.

JavaScript is weak-typed, and do type-checking in JavaScript is somewhat annoying. Thus it may be impossible to force a contract.

But we do have approaches to decrease the risk. Here are some I adopt in my daily coding:

* _Explicitly Declare it if I want to reuse something._

  For example, if I want `tim` to do self-introduction the same way `max` does.
  I would choose to declare `introduceWithWow(person)` under a shared name-space 
  such as `HumanActions`.
  
* _Avoid using complicated structures as parameters._ 

  A `Person` can have dozens of properties, 
  but we won't use most of them when doing self-introduction. 
  Moreover, we want `introduce` work for all person-like objects.
  So I would declare `introduceWithWow(name)` to accept a `string` rather than a `Person`.
  
* _Use ECMAScript-2016 and Babel._

  In ES-5, it's annoying when you want to pass a list of arguments to functions like `Math.max`.
  
  But with ES-6, you can do it this way:
  ```javascript
  const nums = [1,2,3,4]
  const max_num = Math.max(...nums)
  ```
  Though Babel will translate it to:
  ```javascript
  Math.max.apply(Math, nums);
  ```
  It's still better to do it manually: at least Babel is fully tested.
  
  Another case is for closure:
  ```javascript
  //it will break in es-5
  var nick = {
   name: 'nick',
   say: function () {
    return function (phrase) {
     return this.name + ' says: ' + phrase
    }
   }
  }
  
  nick.say()("hi")
  ```
  You may write it this way:
  ```javascript
  var nick = {
   name: 'nick',
   say: function () {
    return (function (phrase) {
     return this.name + ' says: ' + phrase
    }).bind(this)
   }
  }
  
  nick.say()("hi")
  ```
  But in es-6, with arrow functions (or lambda in other languages):
  ```javascript
  const nick = {
    name: 'nick',
    say: function () {
      return (p) => this.name + ' says: ' + p
    }
  }
  ```
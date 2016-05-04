# Write a MVVM framework from scratch (2)



Writing a framework out of thin air is difficult, so we'll try to write some simple application code in this post.

We start with basic JavaScript and refactor it to a MVVM pattern based program.

I wrote all the code in this post with [JSbin](http://jsbin.com) in babel/ES-6 mode. If you feeling confused with any line, don't hesitate to try them out there.

## Write your code in a MVVM way

### The student info app in basic Js

We continue with the student info application mentioned in last post.

Say if we want to write such an app, we may start with the following code:

```Javascript
const student = {
  'first-name': 'Tracy',
  'last-name': 'Kent',
  'height': 170,
  'weight': 50,
}

const root = document.createElement('ul')

const nameLi = document.createElement('li')
const nameLabel = document.createElement('span')
nameLabel.textContent = 'Name: '
const name_ = document.createElement('span')
name_.textContent = student['first-name'] + ' ' + student['last-name']
nameLi.appendChild(nameLabel)
nameLi.appendChild(name_)

const heightLi = document.createElement('li')
const heightLabel = document.createElement('span')
heightLabel.textContent = 'Height: '
const height = document.createElement('span')
height.textContent = '' + student['height'] / 100 + 'm'
heightLi.appendChild(heightLabel)
heightLi.appendChild(height)

const weightLi = document.createElement('li')
const weightLabel = document.createElement('span')
weightLabel.textContent = 'Weight: '
const weight = document.createElement('span')
weight.textContent = '' + student['weight'] + 'kg'
weightLi.appendChild(weightLabel)
weightLi.appendChild(weight)

root.appendChild(nameLi)
root.appendChild(heightLi)
root.appendChild(weightLi)

document.body.appendChild(root)
```

The output is a list like below:

>* Name: Tracy Kent
>* Height: 1.7m
>* Weight: 50kg

A three-row list costs mess of code, amazing horror.

### Refactory for reuse

Why are programmers fascinated with all kinds of best-practice?

That's a result of their sloth.

_Sloth is a virtue for programmers._ : )

One of the greatest ideas in this industry is "reuse". Our current App is full of repeated lines. And one of the most widely accepted rule in program design is "DRY" : 

_Do not Repeat Yourself_

Now let's make the App a little Drier.

We can see that we wrote `document.createElement` dozens of times to create HTML Nodes for the list. Actually, we don't need to do that, since all the list items share the same structure.

Yeah, that should be a shared function.

We firstly copy the lines for name row in a function:

```javascript
const createListItem = function (label, content) {
  const nameLi = document.createElement('li')
  const nameLabel = document.createElement('span')
  nameLabel.textContent = 'Name: '
  const name_ = document.createElement('span')
  name_.textContent = student['first-name'] + ' ' + student['last-name']
  nameLi.appendChild(nameLabel)
  nameLi.appendChild(name_)
}
```
This should not work, so correct it.
```javascript
const createListItem = function (label, content) {
  const li = document.createElement('li')
  const labelSpan = document.createElement('span')
  labelSpan.textContent = label
  const contentSpan = document.createElement('span')
  contentSpan.textContent = content
  li.appendChild(labelSpan)
  li.appendChild(contentSpan)
  return li
}
```
And the whole App turns to:

```javascript
const student = {
  'first-name': 'Tracy',
  'last-name': 'Kent',
  'height': 170,
  'weight': 50,
}

const createListItem = function (label, content) {
  const li = document.createElement('li')
  const labelSpan = document.createElement('span')
  labelSpan.textContent = label
  const contentSpan = document.createElement('span')
  contentSpan.textContent = content
  li.appendChild(labelSpan)
  li.appendChild(contentSpan)
  return li
}

const root = document.createElement('ul')

const nameLi = createListItem('Name: ', student['first-name'] + ' ' + student['last-name'])

const heightLi = createListItem('Height: ', student['height'] / 100 + 'm')

const weightLi = createListItem('Weight: ', student['weight'] + 'kg')

root.appendChild(nameLi)
root.appendChild(heightLi)
root.appendChild(weightLi)

document.body.appendChild(root)
```

Much shorter and far more readable.

You can't tell what I was doing in a mess of Node-creation lines at first sight. But for the new version, it's obvious that I'm creating a list and its items.

For those who read your code, maybe they don't care how you create a list item, they know you're creating a list item, that's enough. For those who are interested in the list item, they can refer to the function `createListItem`.

And they may not care how you make your list. Then it turns out this way:

```javascript
const student = {
  'first-name': 'Tracy',
  'last-name': 'Kent',
  'height': 170,
  'weight': 50,
}

// The list creation util
const createList = function(kvPairs){
  const createListItem = function (label, content) {
    const li = document.createElement('li')
    const labelSpan = document.createElement('span')
    labelSpan.textContent = label
    const contentSpan = document.createElement('span')
    contentSpan.textContent = content
    li.appendChild(labelSpan)
    li.appendChild(contentSpan)
    return li
  }
  
  const root = document.createElement('ul')
  kvPairs.forEach(function (x) {
    root.appendChild(createListItem(x.key, x.value))
  })
  return root
}

//The business logic
const ul = createList([
  {
    key: 'Name: ',
    value: student['first-name'] + ' ' + student['last-name']
  },
  {
    key: 'Height: ',
    value: student['height'] / 100 + 'm'
  },
  {
    key: 'Weight: ',
    value: student['weight'] + 'kg'
  }])
 
document.body.appendChild(ul)
```

### One more step towards mvvm

Now our App looks more or less a mvvm style, not a joke.

The object `student` is our original data, it never changed over our refactory. We can call it a __"Model"__.

The function `createList` returns a DOM tree we display. I think it is reasonable to call it a __"View"__.

How about the __"View-Model"__? 
Unfortunately, we don't have an __isolated__ "View-Model" at present.

Well, I mean, the "View-Model" we have is not isolated. But it do exist.
The parameter we passed to `createList` is a reformation of "Model".
In other words, We __adapted__ the "Model" to the "View" with the manually created Array.

Let's isolate it:
```javascript
//Model
const tk = {
  'first-name': 'Tracy',
  'last-name': 'Kent',
  'height': 170,
  'weight': 50,
}

//View
const createList = function(kvPairs){
  const createListItem = function (label, content) {
    const li = document.createElement('li')
    const labelSpan = document.createElement('span')
    labelSpan.textContent = label
    const contentSpan = document.createElement('span')
    contentSpan.textContent = content
    li.appendChild(labelSpan)
    li.appendChild(contentSpan)
    return li
  }
  
  const root = document.createElement('ul')
  kvPairs.forEach(function (x) {
    root.appendChild(createListItem(x.key, x.value))
  })
  return root
}

//View-Model
const formatStudent = function (student) {
  return [
    {
      key: 'Name: ',
      value: student['first-name'] + ' ' + student['last-name']
    },
    {
      key: 'Height: ',
      value: student['height'] / 100 + 'm'
    },
    {
      key: 'Weight: ',
      value: student['weight'] + 'kg'
    }]
}

const ul = createList(formatStudent(tk))
 
document.body.appendChild(ul)
```

It looks much better, except the last two lines...

Well, let's encapsulate them:
```javascript
const run = function (root, {model, view, vm}) {
  const rendered = view(vm(model))
  root.appendChild(rendered)
}

run(document.body, {
      model: tk, 
      view: createList, 
      vm: formatStudent
})
```

### Requirement change: BMI

Say that, our PM asked us to add a new row for BMI (Body Mass Index).

It is annoying to do that with the original code base.
I won't do it here. 
I hate copying and pasting `document.createElement` dozens of times.

As a comparison, it's easy make it with the MVVM version: 
We just need to modify the "ViewModel" since BMI can be calculated from height and weight.

```javascript
const formatStudent = function (student) {
  return [
    {
      key: 'Name: ',
      value: student['first-name'] + ' ' + student['last-name']
    },
    {
      key: 'Height: ',
      value: student['height'] / 100 + 'm'
    },
    {
      key: 'Weight: ',
      value: student['weight'] + 'kg'
    },
    {
      key: 'BMI: ',
      value:  student['weight'] / (student['height'] * student['height'] / 10000)
    }]
}
```

We can simply do it this way or do some farther optimization inside the function.
That's not our issue here.

What I want to say here is:

_Why we choose to modify the View-Model?_

In mvvm pattern, we always make modifying the View-Model the first-choice if it's time to change.

I think it's not difficult to understand: 

_The View could be used to display other data-sets; 
it cares only about how things are displayed._

_The Model could be displayed in other forms; 
it cares only about what is done in with the business._

They have the potential of __reuse__. 
Thus we'd better make them generic.

_The View-Model is something you would hardly reuse; 
it is a specialized adapter between a certain View and a certain Model._

Since it's specialized, modifying it won't throw you at a risk of breaking down other parts of the program.
But if you want to do something with a View or Model, you need to check the whole solution for every placed they're used.

### Toggle the measurement of height

In China, there is a joke that a programmer can be friend with anyone except a product manager. Because PMs are always changing their requirements : ). 

Imagine that the PM tells you to add a toggle to change the measurement of height...

Actually, I don't want to explain much about how to manage user inputs here. 
It is a little complicated, so I planned to make it in later posts.
But user inputs is so important in UI development.
I think it is necessary to leave a few words on this issue.

To add a button, we need to modify our View.
Our View may be reused by others, so we should not modify the present view rashly.

Here we'll reuse the old View by composing it with some new code.

Firstly, we need something to stand for the current measurement, so we have to invoke a new Model.

```javascript
const tk = {
  'first-name': 'Tracy',
  'last-name': 'Kent',
  'height': 170,
  'weight': 50
}

const measurement = 'cm'
```

We add a measurement data source, rather than modify tk:
So tk can still be used by other modules.

For the View part, we can reuse our list-view as part of the new View:
```javascript
const createList = function(kvPairs){
  const createListItem = function (label, content) {
    const li = document.createElement('li')
    const labelSpan = document.createElement('span')
    labelSpan.textContent = label
    const contentSpan = document.createElement('span')
    contentSpan.textContent = content
    li.appendChild(labelSpan)
    li.appendChild(contentSpan)
    return li
  }
  
  const root = document.createElement('ul')
  kvPairs.forEach(function (x) {
    root.appendChild(createListItem(x.key, x.value))
  })
  return root
}

const createToggle = function (options) {
  const createRadio = function (name, opt){
    const radio = document.createElement('input')
    radio.name = name
    radio.value = opt.value
    radio.type = 'radio'
    radio.textContent = opt.value
    radio.addEventListener('click', opt.onclick)
    radio.checked = opt.checked
    
    return radio
  }
  
  const root = document.createElement('form')
  options.opts.forEach(function (x) {
    root.appendChild(createRadio(options.name, x))
    root.appendChild(document.createTextNode(x.value))
  })
  
  return root
}

const createToggleableList = function(vm){
  const listView = createList(vm.kvPairs)
  const toggle = createToggle(vm.options)
  
  const root = document.createElement('div')
  root.appendChild(toggle)
  root.appendChild(listView)
  
  return root
}
```
Our `createToggle` function returns a form with a series of radio buttons.
But from the current code, we know nothing about what the role it will play in our App.
In other words, __it is business decoupled.__

Now the last, View-Model part:

As we can see, the `createToggleableList` function requires a different parameter from our previous `createList`.  
Thus a refactory on View-Model structure is necessary.

```javascript
const createVm = function (model) {
  const calcHeight = function (measurement, cms) {
    if (measurement === 'm'){
      return cms / 100 + 'm'
    }else{
      return cms + 'cm'
    }
  }
  
  const options = {
    name: 'measurement',
    opts: [
      {
        value: 'cm',
        checked: model.measurement === 'cm',
        onclick: () => model.measurement = 'cm'
      },
      {
        value: 'm',
        checked: model.measurement === 'm',
        onclick: () => model.measurement = 'm'
      }
    ]
  }
  
  const kvPairs = [
    {
      key: 'Name: ',
      value: model.student['first-name'] + ' ' + model.student['last-name']
    },
    {
      key: 'Height: ',
      value: calcHeight(model.measurement, model.student['height'])
    },
    {
      key: 'Weight: ',
      value: model.student['weight'] + 'kg'
    },
    {
      key: 'BMI: ',
      value:  model.student['weight'] / (model.student['height'] * model.student['height'] / 10000)
    }]
  return {kvPairs, options}
}
```
We add `opts` for `createToggle` and warp them into one single object.

According to measurement, we use different formula to calculate `height`; 
 When any radio is clicked, the model's measurement would change.

Seems perfect.
But it will not work when you click the radio button.
Because we have no update-mechanism for data change.

This part, about how a MVVM framework handles model update is a little twisted (Thought not hard). 
I'd like to leave it for next post.

Here we'll use a most simple way to make it. 
```javascript
const run = function (root, {model, view, vm}) {
  let m = {...model}
  let m_old = {}
  
  setInterval( function (){
    if(!_.isEqual(m, m_old)){
      const rendered = view(vm(m))
      root.innerHTML = ''
      root.appendChild(rendered)
      
      m_old = {...m}
    }
  },1000)
}

run(document.body, {
      model: {student:tk, measurement}, 
      view: createToggleableList, 
      vm: createVm 
})
```
This mechanism is called "polling" in computer science. 
It is not a good idea to use it in your own App in browser.
Though it is widely used by browsers : ).

Here we invoked a foreign lib. 
I'm too lazy to write a `areEqual` function by myself. 
So I use [lodash](https://lodash.com/) for checking model update.

Every second, the `run` function would check if a model update happens:
If so, we'll rerender the whole View (this would cause performance issues when you have large amounts of DOM Nodes);
If not, we'll do nothing, and wait for next second.

This is the simple example of a MVVM styled App. 
next post we'll try to build a toy MVVM framework from this App.
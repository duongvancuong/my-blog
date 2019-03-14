---
title: Call, Apply and Bind in Javascript
date: "2018-08-07T23:46:37.121Z"
template: "post"
draft: false
slug: "/posts/call-apply-bind-in-javascript/"
category: "Javascript"
tags:
  - "Javascript"
description: "Làm quen với javascript nhiều thì chúng ta sẽ luôn nhìn thấy các keyword như call, apply, bind và chúng là những khái niệm ai làm việc với javascript cũng nên biết và nắm được"
---

chào mọi người, :hand:

Hôm này mình sẽ giới thiệu về call(), apply() và bind().

**call** và **apply** có hình thức hoạt động giống nhau, chỉ khác nhau là **call** nhận vào 1 serise arguments còn **apply** thì nhận vào 1 array.

```javascript
function add (a, b) {
    return a + b;
}

// Outputs: 3
console.log(add.call(this, 1, 2));

// Outputs: 3
console.log(add.apply(this, [1, 2]));
```

# CALL

- **call** sẽ gọi ngay lập tức function cái mà được đính kèm vào nó. Nếu bạn muốn thay đối value của **this** , bạn có thể pass value mong muốn như parameter đâu tiên đến **call**. :

```javascript
var animal = {
    introduce: function() {
        return this.name + " is a " + this.type + " and says " + this.sound + "!";
    }
};

var whiskey = {
    name: "Whiskey",
    type: "dog",
    sound: "woof"
};

var moxie = {
    name: "Moxie",
    type: "cat",
    sound: "meow"
};

// set the thisArg to be the object whiskey:
animal.introduce.call(whiskey); // "Whiskey is a dog and says woof!"

// set the thisArg to be the object moxie:
animal.introduce.call(moxie); // "Moxie is a cat and says meow!"
```

Từ parameters thứ 2 - n sẽ là params của function được khai báo trong **call**:

```javascript
var person1 = {
    name: "Matt",
    greet: function(otherName) {
        return "Hi, " + otherName + ", I'm " + this.name + ". Nice to meet you!";
    }
}

var person2 = {
    name: "Tim"
}

// person1 greets person2:

person1.greet(person2.name); // "Hi Tim, I'm Matt. Nice to meet you!

// person2 tries to greet person1, but there's a problem...

person2.greet(person1.name); // TypeError: person2.greet is not a function

// person2 doesn't have a greet method! So let's borrow the one from person1...

person1.greet.call(person2); // "Hi, undefined, I'm Tim. Nice to meet you!"

// We still need to pass person1's name to the function being called! Let's pass the argument to the function inside of call:

person1.greet.call(person2, person1.name); // "Hi, Matt, I'm Tim. Nice to meet you!"
```

Một trong những trường hợp sử dụng **call** là để chuyển đổi một object giống array thành một array thật sự:

```javascript
function sumArgumentsIncorrectly(){
    return arguments.reduce(function(acc,next){
        return acc + next;
    },0);
}

sumArgumentsIncorrectly(1,2,3,4,5) // this throws a type error because the arguments "array-like object" does not contain the method reduce!

function sumArgumentsCorrectly(){
    // we are going to use the slice method which makes copies of arrays, but instead of making a copy of [], we will use the arguments array as the context that we want slice to be called in. We can immediately attach reduce and we are good to go!
    return [].slice.call(arguments).reduce(function(acc,next){
        return acc + next;
    },0)
}

sumArgumentsCorrectly(1,2,3,4,5) // 15
```

# APPLY
- Nó khá giống với **call**, chỉ ngoại trừ nó chỉ nhận vào tối đa 2 arguments.
Như **call** argument đầu tiên sẽ để áp dụng cho ***thisArg***, cái sẽ thay đổi value cho **this***. Không giống **call** argument thứ 2 luôn là một array các parameters.

- Vậy thì khi nào nên sử dụng **call** khi nào nên sử dụng **apply**?
Nếu bạn không phải truyền bất cứ đối số nào vào function thì bạn có thể sử dụng 1 trong 2 cách. Nếu ngược lại bạn có 1 array các params để pass và không biết chính xác là có bao nhiêu params thì **apply** sẽ là lựa chọn.

```javascript
var matt = {
    firstName: "Matt",
    lastName: "Lane",
    instructor: true,
    favColor: "blue",
    dogOwner: true,
    deleteInfo: function() {
        console.log(arguments);
        for (var i = 0; i < arguments.length; i++) {
            delete this[arguments[i]];
        }
    }
}

var tim = {
    firstName: "Tim",
    lastName: "Garcia",
    instructor: true,
    favColor: "blue",
    dogOwner: false
};

var elie = {
    firstName: "Elie",
    lastName: "Schoppik",
    instructor: true,
    favColor: "purple",
    dogOwner: false
};

matt.deleteInfo('instructor', 'favColor');
matt; // {firstName: "Matt", lastName: "Lane", dogOwner: true}

matt.deleteInfo.apply(tim,['firstName', 'dogOwner', 'favColor']);
tim; // {lastName: "Garcia", instructor: true}

matt.deleteInfo.apply(elie,['instructor','favColor','dogOwner','lastName']);
elie; // {firstName: "Elie"}
```

Trong những trường hợp cụ thể chúng ta sẽ có thể áp dụng như thế này:

```javascript
var numbersArray = [1,2,3,4,5];

Math.max.apply(this,numbersArray) // 5

```

```javascript
var arrayToBeFlattened = [1,2,[3,4],[5,6]]

[].concat.apply([],arrayToBeFlattened) // [1,2,3,4,5,6]
```

# BIND
Giống như **call** thì **bind** nó lấy param đầu tiên như để cho **this**, và từ param thứ 2-n thì áp dụng cho function.
Tuy nhiên function không được thực hiên ngay, thay vào đó nó trả về một function có thể được gọi ngày sau đó tại một  thời điểm nào đó. Nói cách khác ```fn.bind(thisArg,arg1,arg1,arg3)()``` giống với ```fn.call(thisArg,arg1,arg2,arg3)```

```javascript
function add(a,b){
    return a+b;
}

var partialAdd = add.bind(this,2)
partialAdd(4) // 6
```

Chúng ta có thể nhìn thấy Bind() trả về một function mới thì nó giống như cách đơn giản để sử dụng closure.

```javascript
function bind(fn,thisArg){
    var outerArgs = Array.prototype.slice.call(arguments)
    var argsWeWant = outerArgs.slice(2) // we don't want the fn and thisArg values! Let's copy from the 2nd index of the arguments array to the end!
    return function(){
        return fn.apply(thisArg, argsWeWant.concat([].slice.call(arguments)))
        /*/
        remember that the 2nd parameter of apply takes in an array.
        So we are concatenating (joining) the arguments from the outer  function with the arguments from the inner function
        to form 1 big array of arguments to be used when the inner function is finally called.
        /*/
    }
}

function add(a,b){
    return a+b;
}

// check out some cool stuff we can do with our bind function now!

bind(add,this,1,11)() // 12
bind(add,this)(1,11) // 12
bind(add,this,1)(11) // 12
bind(add,this,1)(11,5,6,7,8,9) // 12 - the rest are ignored!
```

Thank you so much!

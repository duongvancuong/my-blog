---
title: Currying trong Javascript
date: "2018-07-25T23:46:37.121Z"
template: "post"
draft: false
slug: "/posts/currying-in-javascript/"
category: "Javascript"
tags:
  - "Javascript"
description: "Currying là một kỹ thuật khá hay trong Javascript, Currying là kỹ thuật đề cập đến quá trình chuyển đổi một function với việc đưa vào n đối số và đưa nó vào trong n function"
---

chào mọi người, :hand:

Hôm này mình sẽ giới thiệu về một kỹ thuật trong javascript, đó là Currying và những điều thú vị của nó mang lại.

# Currying là gì ?

Currying là kỹ thuật đề cập đến quá trình chuyển đổi một function với việc đưa vào n đối số và đưa nó vào trong n function mà mỗi function lấy một đối số. Về cơ bản nó tạo ra một chuổi các function mà cuối cùng được resolve bằng một value.

Nó có nghĩa là bạn có thể pass tất các cả arguments vào trong một function đang mong đợi và lấy kết quả. Hoặc là pass một tâp con các đối số và mong đợi lấy về một function cái sẽ trông đợi các arguments còn lại.

Để có thể hình dung nó làm việc như thế nào chúng ta hãy tạp một curried function đầu tiên. Giả sử chúng ta sẽ có một function trả một một message chào đón 1 ai đó với name của họ.

```javascript
var xinchao = function(greeting, name) {
  console.log(greeting + ',' + name);
}

xinchao('xin chao', 'cuong');
=> 'xin chao, cuong'
```

function trên mong đội 2 arguments là greeting và name để thực hiện đúng. Chúng ta có thể viết lại bằng cách sử dụng một nested currying chỉ yêu cầu greeting và trả về một function cần một argument là name.

```javascript
var greetCurried = function(greeting) {
  return function(name) {
    console.log(greeting + ' , ' + name);
  }
}
```
và với cách viết trên chúng ta sẽ có thể tạo ra một new function với bất cứ greeting nào. và pass vào new function đó một name nào đó.

```javascript
var greetHello = greetCurried('Hello');
greetHello('cuong'); // Hello, cuong
greetHello('world'); // Hello, world
```

Chúng ta cũng có thể call curried function trực tiếp, bằng các pass từng arguments vào trong từng cặp dấu ngoặc tròn.

```javascript
var greetCurried('xin chao')('cuong'); // xin chao, cuong
```

chúng ta có thể là như vậy cho nhiều arguments hơn. ko chỉ là 2 như ví dụ trên.

Với cách viết như vậy sẽ có thể gây khó đọc và cảm thấy lộn xộn khi phải sử dụng nhiều dấu ngoặc đơn. Để giải quyết vấn đề này chúng ta có thể thực hiện một currying function cái sẽ lấy là tên của một function đã được viết mà không có các nested.

```javascript
var curryIt = function (uncurried) {
  var parameters = Array.prototype.slice.call(arguments, 1);
  return function() {
    return uncurried.apply(this, parameters.concat(
      Array.prototype.slice.call(arguments, 0);
    ));
  };
}
```

Để sử dụng nó chúng ta pass tên của function đã được tạo trước đó, vào các arguments cần sử dụng:

```javascript
var greeter = function(greeting, separator, emphasis, name) {
  console.log(greeting + separator + name + emphasis);
};
var greetHello = curryIt(greeter, "Hello", ", ", ".");
greetHello("Cuong"); //"Hello, Cuong."
greetHello("World"); //"Hello, World."
```

Chúng ta không giới hạn số lượng các arguments. chúng ta có thể viết như sau:

```javascript
var greetGoodbye = curryIt(greeter, "Goodbye", ", ");
greetGoodbye(".", "Cuong"); //"Goodbye, Cuong."
```
Một điều quan trọng là chúng ta cần chú ý đến thứ tự của các arguments. Suy nghĩ về thứ tự các arguments trước sẽ làm cho việc sử dụng currying function của chúng ta hiệu quả hơn.



```javascript
function convert(str){
    return str.toUpperCase().split('').filter(function(val){
        return val.match(/[AEIOU]/);
    }).join(":")
}

convert("hello") // "O:E"


function compose(...functions) {
	return (start) => {
		return functions.reduceRight((acc, next) => {
			return next(acc);
		}, start)
	}
}

function complexCurry(fn) {
  return function f1(...f1innerArgs) {
    if (f1innerArgs.length >= fn.length) {
      return fn.apply(this, f1innerArgs);
    } else {
      return function f2(...f2innerArgs) {
        return f1.apply(this, f1innerArgs.concat(f2innerArgs));
      }
    }
  };
}

var join = complexCurry(function(str, arr){
    return arr.join(str);
})

var filter = complexCurry(function(fn,arr){
    return arr.filter(fn);
})

var isVowel = complexCurry(function(char){
    return char.match(/[AEIOU]/);
})

var split = complexCurry(function(delimiter, str){
    return str.split(delimiter);
})

var toUpperCase = complexCurry(function(str){
    return str.toUpperCase();
})

var convertLetters = compose(join(':'), filter(isVowel), split(''),toUpperCase());

console.log(convertLetters('This is some pretty crazy stuff')); // I:I:O:E:E:A:U
```
If we wanted to start with the outer most function f -> g -> x we can use what is called a sequence function.

```javascript
function sequence(...functions) {
    return function(start){
        return functions.reduce(function(acc, next) {
            return next(acc);
        } , start);
    }
}

var convertLetters = sequence(toUpperCase(), split(''), filter(isVowel), join(':'));

console.log(convertLetters('This is some pretty crazy stuff')); // I:I:O:E:E:A:U
```


Thank you so much!

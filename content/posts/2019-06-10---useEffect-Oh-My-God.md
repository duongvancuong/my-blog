---
title: How about useEffect in Hooks.
date: "2019-06-10T23:46:37.121Z"
template: "post"
draft: false
slug: "/posts/how-about-useEffect/"
category: "ReactJS"
tags:
  - "Javascript"
  - "ReactJS"
  - "Hooks"
description: "Bắt đầu từ version 16.8.xx React Team của Facebook đã đưa ra Hooks một số function mới và theo bản thân mình thì nó làm thay đổi khá nhiều về cách code 1 Component, cũng chạy theo trend nên mình cũng đã áp dụng vào một vài dự án và thấy một vài trải nghiệm thú vị nên viết bài này để chia sẽ một vài trải nghiệm của mình với 1 effect trong Hooks là useEffect"
---

Trong Hooks có nhiều methods nhưng ở đây mình sẽ chỉ nói về thằng **useEffect**, vì thằng này nó thú vị và khôn lường lắm, cũng bởi vì thế mà nhiều vấn đề đau đầu vì nó nếu chúng ta không hiểu rõ về nó.

### Tôi đã thay ComponentDidMount bởi useEffect thế nào:
```useEffect(fn, [])``` đó là cách tôi thay thế **ComponentDidMount()**, Với cách viết như vầy parameter đầu tiên sẽ là một function, trong function đó chúng ta có thể thực hiện những logic như fecth data, hay setState làm những thứ các bạn đã có thể làm được với **ComponentDidMount*, parameter thứ 2 sẽ là một Array rỗng tếch. Chính vì thế mà nó sẽ chỉ chạy vào Effect này 1 lần duy nhất sau lần `render`  đầu tiền.

Nhưng đưng code nhiều thứ quá ở nó nhé nhìn sẽ nổ mắt đấy =))

### Vậy cái [] (cái hộp rỗng tếch) mình dùng ở trên thì nó có tác dụng gì:
Với cái cách đặt ```[]``` đúng vị trí và không bỏ cái gì vào đó, sẽ làm cho effect này không sử dụng bất kỳ value vào tham gia vào quá trình xử lý data của React. Và đó là cách để nó chỉ thực hiện 1 lần duy nhất.

Vậy nếu chúng ta ném vào trong cái cái hộp này ```[]``` vài giá trị thì sẽ thế nào nhỉ ? mình sẽ nói ở dưới sau :v

### useEffect lặp vô hạn (???) cái này chắc ai khi bắt đầu sử dụng cũng sẽ bị hoy:
Lấy 1 ví dụ nhé, khi chúng ta thực hiện fetching data trong useEffect và không đưa và parameter thứ 2, ```useEffect(fn)```. và trong effect này cũng ta thực hiện việc set một value sẽ được dùng trong data flow của component. thì nó sẽ xuất hiện ngay error.
Hoặc một trường hợp khác bạn ném vào trong cái ```[]``` này, những value luôn thay đổi, thì cái effect nãy sẽ loop mãi cho mà xem.

OK! vậy ở đây chúng ta sẽ có 2 cách để làm việc với useEffect nữa:
- ```useEffect(fn)``` không đưa vào [], thì effect này sẽ thực hiện liên tục mỗi khi component được render.
- ```useEffect(fn, [...value]);``` đưa value vào mình dùng ... để nói là nó có thể là nhiều value nhé, kể cả là function luôn :v. với cách này thì effect sẽ thực hiện mỗi khi có ít nhất một value được thay đổi. thấy có thể thay thế ```ComponetDidUpdate``` không ? không hẳn đâu nhé. :v

### Bàn về rendering trong react để rõ hơn về bản ngã của useEffect
Trước khi tiếp tục với **useEffect** chúng ta sẽ tìm hiểu chút về việc React rendering component nhưng thế nào với ```props`` và ```state``` . Cái này khá mà phức tạo nên Khá bãnh cũng khá đuối :v.

Vậy nên những chia sẽ tiếp theo sẽ là trải nghiệm và góp nhặt của tôi mang tới cho các bạn để hiểu rõ hơn. (Đừng phán xét tôi nhé) =))

Now, hãy xem chúng ta có gì ở những dòng code này

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

chúng ta có **count** đơn giản chỉ là 'con số 0' không có gì magic ở đây, và khi chúng ta click thì **count** đó sẽ change, và component sẽ re-render.
Mỗi lần click thì **count** được trả về từ **useState()** sẽ là 0, 1, 2, rồi tiếp theo đó ...
Và chúng ta có thể thấy rằng **count** là một ```const``` tại mỗi lần render của component, vì thế chúng ta biết rằng **count** bên trong mỗi lần render sẽ không thể thay đổi.

### Vậy thì mỗi lần render với handle event sẽ thế nào ?

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert('You clicked on: ' + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
      <button onClick={handleAlertClick}>
        Show alert
      </button>
    </div>
  );
}
```

Nhìn vào những dòng code trên chúng ta có gì? chúng ta sẽ có 1 alert message được phóng ra sau 3s khi click button.
Hãy xem cụ thể hơn nào, bất code https://codesandbox.io lên nào, right now!

làm theo cái tôi mô tả :
- click vào button **click me** 3 lần.
- click vào button **show alert** ngay.
- lại click ngay **click me** thêm 2 lần nữa.

rồi đoán xem cái alert message sẽ là gì ? À mà cái vì dụ này tôi cũng có ở bài trước để phân tích Class và Function [xem ở đây nào](https://tek-fun.netlify.com/posts/how-are-function-component-different-from-classes/)

Ok! kết quả là 3 đúng ko nào, Nhưng tại sao lại là 3 mà không phải là số khác. Mổ xẻ nó ra nào !

Như đã nói ở trên **count** là một constant vậy nên giá trị của nó tại mỗi lần call function cụ thể sẽ không hề thay đổi.
Thêm 1 ví dụ nữa nhé

```javascript
  function sayHi(person) {
    const name = person.name;
    setTimeout(() => {
      alert('Hello, ' + name);
    }, 3000);
  }

  let someone = {name: 'Dan'};
  sayHi(someone);

  someone = {name: 'Yuzhi'};
  sayHi(someone);

  someone = {name: 'Dominic'};
  sayHi(someone);
```

với value **someone** luôn thay đổi sau mỗi lần gọi **sayHi** function, thì **name** bên trong của mỗi function **sayHi** cũng sẽ không hề thay.

Như vậy đã giải thích cách event handle xử lý **count** tại mỗi lần click. vào đó là lý cho tại sao chúng ta có click thêm thì việc xử lý giá trị tại mỗi event handle là 1 giá trị cụ thể không thay đổi tại mỗi lần.

### Vậy mỗi lần Render thì các effect của nó như thê nào ?

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

Chúng ta biết rằng userEffect sẽ thực hiện sau mỗi lần component render. vì thế giá trị **count** cũng sẽ thuộc về mỗi effect sau mỗi lần render.
Như vậy chúng ta có thể thấy răng **useEffect** nhưng một phần cụ thể tại thuộc về mỗi lần render khác nhau của Component.

### Vậy thì props và state tại mỗi lần render sẽ thế nào ?
```javascript
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    setTimeout(() => {
      console.log(`You clicked ${count} times`);
    }, 3000);
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

thực hiện đoạn code trên và show logs chúng ta thấy nó vẫn thể hiện đúng như những phân tích ở trên.

Nhưng nếu chúng ta thử nó trong Class component thì sẽ thế nào ```this.state```

```javascript
componentDidUpdate() {
    setTimeout(() => {
      console.log(`You clicked ${this.state.count} times`);
    }, 3000);
  }
```

với cách này chúng ta sẽ thấy nó show console **count** cuối cùng, không còn là giá trị cụ thể tại mỗi lần render nữa.
Như thế khi sử dụng ```this.state.count`` chúng ta luôn nhận được giá trị cuối cùng.

Nhưng một vài trường hợp nào đó, chúng ta lại muốn lây giá trị cuối cùng của props hoặc state thì sao? Cách đơn giản là sử dụng **refs**.

```javascript
function Example() {
  const [count, setCount] = useState(0);
  const latestCount = useRef(count);

  useEffect(() => {
    // Set the mutable latest value
    latestCount.current = count;
    setTimeout(() => {
      // Read the mutable latest value
      console.log(`You clicked ${latestCount.current} times`);
    }, 3000);
  });
```
Và đây là cách chúng ta lấy state mới nhật tại mỗi render và useEffect tại mỗi lần render.

### Vậy đâu là cái cách mà React render lại Component và làm tăng perfomance?

Như chúng ta đã biết, React thay vì re-render mọi thứ, thì nó chỉ re-render những phần thay đổi của DOM.

chúng ta thay đổi từ

```javascrtip
<Counter className="blue">this is counter</Counter>
```

thành

```javascrtip
<Counter className="blue">this is example</Counter>
```

React sẽ nhìn phần được 2 Object

```javascript
const oldProps = {className: 'blue', children: 'this is counter'};
const newProps = {className: 'blue', children: 'this is example'}
```
thì cái thay đổi cho là children chứ ko phải className, Vậy nên nó chỉ thay đổi
```
domNode.innerText = 'this is example';
```

Vậy nên trong useEffect để tránh việc thực hiện việc chạy lại những function không cần thiết, chúng ta cần xác định dựa vào việc đưa vào cái [], những dependencies cụ thể.

```javascript
function Greeting({ name }) {
  const [counter, setCounter] = useState(0);

  useEffect(() => {
    document.title = 'Hello, ' + name;
  });

  return (
    <h1 className="Greeting">
      Hello, {name}
      <button onClick={() => setCounter(count + 1)}>
        Increment
      </button>
    </h1>
  );
}
```

với code trên thì useEffect sẽ luôn chạy lại sau mỗi lần re-render khi change count, trong khi đó useEffect không hề xử lý logic gì đến nó.
Để giải quyêt điều đó chúng ta sẽ thực hiện lại

```javascript
useEffect(() => {
    document.title = 'Hello, ' + name;
  }, [name]);
```

Như Vậy useEffect này chỉ chạy lại khi props name thay đổi mà thôi.

Hazz! Nội dung chia sẽ về thằng này còn rất là nhiều, nên mình sẽ viết thêm ở bài sau. giờ thì đau lưng rồi, xin phép dừng ở đây nhé.
Bài sau sẽ ra lò ngày mọi người sẽ không phải đợi lâu :vs

Thank you for your reading!

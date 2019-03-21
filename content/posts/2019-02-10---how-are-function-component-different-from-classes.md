---
title: How are function component different from classes
date: "2019-02-10T23:46:37.121Z"
template: "post"
draft: false
slug: "/posts/how-are-function-component-different-from-classes/"
category: "ReactJS"
tags:
  - "Javascript"
  - "ReactJS"
description: "Chắc hẳn ai đã từng nghiên cứu hoặc làm việc với ReactJS cũng đã từng đặt ra suy nghĩ, thắc mắc về Class và Function Component, khi nào thì sử dụng, và nó khác nhau như thế nào. Hay một số người lại cho rằng cái này tốt hơn cái kia..."
---

Chắc hẳn ai đã từng nghiên cứu hoặc làm việc với ReactJS cũng đã từng đặt ra suy nghĩ, thắc mắc về Class và Function Component, khi nào thì sử dụng, và nó khác nhau như thế nào. Hay một số người lại cho rằng cái này tốt hơn cái kia, hay Function component là stateless nên performance tốt hơn, vân vân.

Cũng tò mò và cảm thấy khó hiều, mò mẫm mãi tham khảo mãi cũng hiểu 1 chút ít nên tôi muốn chia sẽ 1 chút về sự khác nhau giữa Function Component và Class Component.

Tôi không nói đến việc cái nào tốt hơn cái nào, hay nên dùng cái nào, chỉ là sự khác nhau của nó thôi nhé.

> Function components capture the rendered values.

Chúng ta hãy phân tích câu trên 1 chút nhé.

Giả sử chúng ta có Function Component:

```javascript
function ProfilePage(props) {
  const showMessage = () => {
    alert('Followed ' + props.user);
  };

  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };

  return (
    <button onClick={handleClick}>Follow</button>
  );
}
```

Chúng ta có 1 Button để Follow, khi click vào Button sau 3s thì nó sẽ show alert `Followed Tek Fun` chẳng hạn.
và chú ý là việc sử dụng function hay arrow function cho `handleClick` như trên thì cũng sẽ như nhau.

và tương tự chúng ta sẽ có 1 class Component :

```javascript
class ProfilePage extends React.Component {
  showMessage = () => {
    alert('Followed ' + this.props.user);
  };

  handleClick = () => {
    setTimeout(this.showMessage, 3000);
  };

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
}
```

Bạn đã nhìn thấy nó khác nhau chưa nhỉ? Thật sự tôi cũng khó để nhận biết là nó khác nhau ở chổ nào, ngoài cách viết chúng.

Nếu bạn dự định sử dụng Function Component thường xuyên thì có lẽ nên hiểu về nó. Tôi cũng chưa hiểu vậy nên phân tích sâu hơn 1 chút.

Minh hoạ rõ hơn về nó tôi thêm 1 App ( tôi sưu tầm trên Internet thôi nhé. ) Compnent để sử dụng 2 Compoent trên:

```javascript
class App extends React.Component {
  state = {
    user: 'TekFun',
  };
  render() {
    return (
      <>
        <label>
          <b>Choose profile to view: </b>
          <select
            value={this.state.user}
            onChange={e => this.setState({ user: e.target.value })}
          >
            <option value="TekFun">TekFun</option>
            <option value="Facebook">Facebook</option>
            <option value="Instagram">Instagram</option>
          </select>
        </label>
        <h1>Welcome to {this.state.user}’s profile!</h1>
        <p>
          <ProfilePageFunction user={this.state.user} />
          <b> (function)</b>
        </p>
        <p>
          <ProfilePageClass user={this.state.user} />
          <b> (class)</b>
        </p>
        <p>
          Can you spot the difference in the behavior?
        </p>
      </>
    )
  }
}
```

các bạn nên code lại để hình dung hơn nhé,

---

Mô tả 1 chút về Component trên nhé:

1. Click vào một trong 2 button follow.
2. thay đổi select input trước 3s.
3. Show alert text.

Chúng ta sẽ chú ý alert text được show ra khi click vào 2 button.

* Với `ProfilePageFunction` thì khi click vào và sau đó đổi select input thành `Facebook` thì alert vẫn sẽ show ra text là `Followed TekFun`.
* Với `ProfilePageClass` tương tự thao tác thì sẽ show ra là `Followed Facebook`.

Như vậy chúng ta đã hình dung nó khác nhau rồi đấy. vậy tại sao? và cái nào là đúng ?

Cái thứ nhất là đúng nhé, về mặt người dùng thì khi click rồi sau đó thay đổi nhưng đối tượng follow vẫn là người đầu tiên lúc click chứ nhỉ?  vì nếu như cái thứ 2 sẽ làm người dùng cảm thấy khó hiểu vì sao chọn `TekFun` click rồi , sau đó đổi lại `Facebook` lại show ra `Followed Facebook`. phải là `Followed TekFun` chứ. thế bug thôi. ăn ngay 1 con gián.

---

Tại sao lại như vậy ?

nhìn vào `showMessage` method của class:

```javascript
class ProfilePage extends React.Component {
  showMessage = () => {
>   alert('Followed ' + this.props.user);
  };
```

Class method trên đọc từ `this.props.user` . Props là immutable trong ReactJS, vì thế nó sẽ không bao giờ thay đổi. Nhưng `this` sẽ luôn thay đổi vì nó là mutable.

Đó là mục đích của `this` trong class. React luôn tự biến đổi nó, vì thế chúng ta có thể đọc được các version mới trong `render` và `lifecycle` methods.

Vì thế nếu component re-render trong khi request đã trong trong tiến trình, `this.props` sẽ thay đổi. `ShowMessage` method sẽ đọc `user` từ `props` mới nhất.

Điều đó cho thấy rằng, nếu chúng ta nói rằng UI là một thể hiện của state hiện tại. các `event handler` sẽ nên chỉ xử lý 1 props hoặc state cụ thể. Tuy nhiên việc sử dụng `this.props` đã phá vỡ điều đó.

---

Vậy tại sao Function Component lại giải quyết được vấn đề đó ?

Chúng ta sẽ thử điều chỉnh 1 chút về class Component đó như thế này :

```javascript
class ProfilePage extends React.Component {
  showMessage = (user) => {
>  alert('Followed ' + user);
  };

  handleClick = () => {
>  const {user} = this.props;
    setTimeout(() => this.showMessage(user), 3000);
  };

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
}
```

Với điều chỉnh trên chúng ta đã lấy props chính xác tại thời điểm đó. và ném nó vào `showMessage`.
nó hoạt động tốt. Nhưng nó làm cho code trở nên dài dòng hơn, và dễ phát sinh bug về sau này khi phức tạp hơn.

Tuy nhiên vấn đề này có thể giải quyết bằng cách dựa trên `javascript Closures`.

Điều đó có nghĩa là nếu chúng ta đóng state hay props cụ thể nào tại một render cụ thể nào đó. thì chúng ta luôn có chính xác chúng là như nhau.

```javascript
class ProfilePage extends React.Component {
  render() {
    // chúng ta bắt lấy props.
    const props = this.props;

    // Note: chúng ta làm điều này trong render.
    // không có class methods.
    const showMessage = () => {
      alert('Followed ' + props.user);
    };

    const handleClick = () => {
      setTimeout(showMessage, 3000);
    };

    return <button onClick={handleClick}>Follow</button>;
  }
}
```

với cách này chúng ta bắt chính xác props tại mỗi lần `render`.

Cách trên có thể đúng, nhưng hơi phức tạp và điều gì sẽ xảy ra khi chúng ta khai báo các methods bên trong `render` thay vì là `class methods`.

```javascript
function ProfilePage(props) {
  const showMessage = () => {
    alert('Followed ' + props.user);
  };

  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };

  return (
    <button onClick={handleClick}>Follow</button>
  );
}
```

Như cách trên thì `props` của chúng ta vẫn đang bị bắt, React ném chúng như một biến. không như `this` thì `props` là object không bao giờ bị thay đổi bởi React.

Khi Component cha render `ProfilePage` với props khác thì call `ProfilePage` một lần nữa. nhưng event handler của chúng ta khi clicked đã thuộc về render trước và `showMessage` đã đọc nó. chúng sẽ không bị mất.

---

> Function components capture the rendered values.

bấy giờ chúng ta đã hiểu sự khác nhau lớn giữa `Function Compoent` và `Class Component`.

With `Hooks` ở version 16.8 React, cũng dựa trên nguyên tắc như vậy. nói như vậy không có nghĩa tôi khuyên bạn refactor toàn bộ project nhé. `Hooks` vẫn hoạt động tốt khi chúng ta keep `Class Component`.

---

Đó là sự khác nhau mà mình đã tìm hiều, và tham khảo. Bài viết được dựa trên sự tìm hiểu, không phải sự khẳng định.
Các bạn nên tham khảo và tìm hiều nhiều hơn nhé.

Thank you for your reading!

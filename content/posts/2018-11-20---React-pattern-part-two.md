---
title: React Pattern Part 2
date: "2018-11-20T23:46:37.121Z"
template: "post"
draft: false
slug: "/posts/react-pattern-part-two/"
category: "ReactJS"
tags:
  - "Javascript"
  - "ReactJS"
  - "Pattern"
description: "Khi làm việc với ReactJS chúng ta nên chú ý đến những Pattern này. Chúng đều được nói đến trong Document của ReactJS nhưng thật sự ít ai để ý đến, Trong bài viết này mình sẽ nói qua về những pattern này"
---

# Event handlers:

Hầu hết trong việc xử lý xự kiện DOM trong component đều có chứa các element xử lý events. như ví dụ bên dưới chúng ta có một clik handler và chúng ta muốn nó thực hiện một function hoặc method trên cùng một component:

```javascript
class Switcher extends React.Component {
  render() {
    return (
      <button onClick={ this._handleButtonClick }>
        click me
      </button>
    );
  }
  _handleButtonClick() {
    console.log('Button is clicked');
  }
};
```

Mọi thứ đều ổn vì `_handleButtonClick()` là một function và chúng ta pass nó vào trong một attribute `onClick`. Nhưng vấn đề là cái function này không keep scope. Có nghĩa là giả sử chúng ta muốn sử dụng `this` bên trong function này thì chắc chắn sẽ nhận ngay một cái cục Error đỏ ỏng vào mặt.

```javascript
class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { name: 'React in patterns' };
  }
  render() {
    return (
      <button onClick={ this._handleButtonClick }>
        click me
      </button>
    );
  }
  _handleButtonClick() {
    console.log(`Button is clicked inside ${ this.state.name }`);
    // leads to
    // Uncaught TypeError: Cannot read property 'state' of null
  }
};
```

Vì thế cách thông thường nhất chúng ta có thể làm là :

```javascript
<button onClick={ this._handleButtonClick.bind(this) }>
  click me
</button>
```

Tuy nhiên việc sử dụng `bind()` ngày trong attribute `onClick` này làm cho function này lăp đi lặp lại vì Component có thể sẽ render nhiều lần. Vậy nên cách tiếp cận tốt nhất là bind nó ở `contructor`

```javascript
class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { name: 'React in patterns' };
    this._buttonClick = this._handleButtonClick.bind(this);
  }
  render() {
    return (
      <button onClick={ this._buttonClick }>
        click me
      </button>
    );
  }
  _handleButtonClick() {
    console.log(`Button is clicked inside ${ this.state.name }`);
  }
};
```

# Composition

Một trong những lợi ích lớn nhất của ReactJs là khả năng kết hợp, ví dụ như trong ứng dụng có một Header và chúng ta muốn đặt một Navigation vào bên trong nó, chúng ta sẽ có 3 Component như thế này :

```javascript
<App>
  <Header>
    <Navigation> ... </Navigation>
  </Header>
</App>
```

Cách bình thưởng nhất để thực hiện việc này là tham chiều chúng đến cùng một nơi chúng ta cần chúng.

```javascript
// app.jsx
import Header from './Header.jsx';

export default class App extends React.Component {
  render() {
    return <Header />;
  }
}

// Header.jsx
import Navigation from './Navigation.jsx';

export default class Header extends React.Component {
  render() {
    return <header><Navigation /></header>;
  }
}

// Navigation.jsx
export default class Navigation extends React.Component {
  render() {
    return (<nav> ... </nav>);
  }
}
```

Tuy nhiên nó có một vài vấn đề như:
  - App component là nơi chúng ta tập hợp mọi thứ đó là nơi tốt để kết hợp. `Header` component chứa một vài thứ khác như Logo, slogan... Header sẽ trông nice! hơn khi các thành phần đó được pass từ bên ngoài vào hơn là hard-code bên trong. `Navigation` là thành phần động thật khó khi chúng ta hard-code bên trong Header.

Để giải quyết vấn đề này chúng ta sử dụng đến `Children API` cách mà Component cha đọc và truy câp vào component con:

```javascript
// App.jsx
export default class App extends React.Component {
  render() {
    return (
      <Header>
        <Navigation />
      </Header>
    );
  }
}

// Header.jsx
export default class Header extends React.Component {
  render() {
    return <header>{ this.props.children }</header>;
  }
};
```

Mọi thứ trở nên linh động hơn, lúc này chúng ta chỉ cần tập trung vào một Component.
Và mọi Component đều có thể nhận `Props`. như một property.

```javascript
// App.jsx
class App extends React.Component {
  render() {
    var title = <h1>Hello there!</h1>;

    return (
      <Header title={ title }>
        <Navigation />
      </Header>
    );
  }
};

// Header.jsxs
export default class Header extends React.Component {
  render() {
    return (
      <header>
        { this.props.title }
        <hr />
        { this.props.children }
      </header>
    );
  }
};
```

Kỹ thuật này giúp chúng ta có thể mix giữa Component đã tồn tại bên trong và nhưng Component đụợc cung cấp từ bên ngoài.

# Tóm lại:

Trên đây là những chia sẽ của tôi về 2 Pattern khá thú vị và có khả năng áp dụng khá cao nhưng ít người để ý. Cảm ơn vì đã theo dõi.
Hãy theo dõi blog và đừng ngại tương tác nhé.

Thank you so much!

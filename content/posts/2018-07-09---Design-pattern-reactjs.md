---
title: Design Pattern ReactJS
date: "2018-07-03T23:46:37.121Z"
template: "post"
draft: false
slug: "/posts/desgin-pattern-reactjs/"
category: "ReactJS"
tags:
  - "Javascript"
  - "ReactJS"
  - "Pattern"
description: "Design Pattern trong lập trình giúp chúng ta khá nhiều, nó giúp cho code của chúng ta dễ dàng sử dụng hơn, clean hơn, và rất nhiều điều tốt hơn. :D"
---

chào mọi người,

Design Pattern trong lập trình giúp chúng ta khá nhiều, nó giúp cho code của chúng ta dễ dàng sử dụng hơn, clean hơn, và rất nhiều điều tốt hơn. :D

Bài viết này là bài viết đầu tiên của mình trên blog này.(vọc mãi mới tạo được cái này để có hứng thú viết...)

Mình sẽ giới thiệu đến mọi người các kỹ thuật trong reactJS giúp bạn khá nhiều, có thể bạn đã và đang sử dụng nó mà không để ý.

okie bắt đầu nào!

# Compound Components

Compound Components là một cách viết các elements bên trong những cái khác, Nhưng điều chính là các điều kiện bên trong các components lại không hoạt động mà không có các điều kiện của parrent.

Chúng ta có thể định nghĩa nó như một sự chia sẽ các state giống parrent.

Nếu bạn muốn chia sẽ cái gì đó từ parrent container đến các children của nó, bạn có thể sử dụng _React.Children_ cung cấp các tiện ích với _this.props.children_.

``` javascript
{React.Children.map(this.props.children, child =>
  React.cloneElement(child, {
    user,
    logout,
  })
)}
```
Nhưng bạn có thể thấy rằng nó không đủ linh hoạt cho compound components, bởi vì khi bạn thay đổi thứ tự của các phần hoặc bọc chúng ở bên trong các thành phần khác, chúng sẽ không thể truy cập đến props.

Để giải quyết vấn đề trên, chúng ta cần những cách khác để chia sẽ state giữa các components mà không phá vỡ nó. Chúng ta có thể sử dụng Context API để làm điều đó.

Element đầu tiên sử dụng Context API được gọi là Provider, và các provider element là các wrapper parrent để chia sẽ các state cho các elements được chia sẽ state như nhau. Sau đó chúng ta có Consumer là cái component bên trong provider có thể sử dụng để get các values từ provider.

``` javascript
  const ToggleContext = React.createContext({
    on: false,
    toggle: () => {},
    getTogglerProps: props => props
  });
```

Đó là cách mà chúng ta khởi tạo một Context API.

``` javascript
  <ToggleContext.Provider value={this.state} {...rest}>
    {this.props.children}
  </ToggleContext.Provider>
```

và trên là các parrent element mà các consumer cần sử dụng state từ parrent đó.

``` javascript
  <Toggle.Consumer>
    {({ on }) => (on ? children : null)}
  </Toggle.Consumer>
```
và đây là cách mà các consumer sử dụng các state từ thằng parrent của nó. :D

Compound Components là hữu ích, bạn nên sử dụng Context API hơn là _React.children_.

# Render Props

Đây là kỹ thuật sử dụng props, bạn pass một function như là là một render method và nó return lại một element ReactJS.
Chúng ta có thể sử dụng children props như function, và bạn có thể pass các state để render ở một nơi khác.

cho ví dụ nào:

``` javascript
<Toggle {...props}>
  {this.props.children(this.state.on, this.props.getTogglerProps)}
</Toggle>
```

Chúng ta cũng có thể sử dụng render props theo cách khác mà không cần sử dụng _this.props.children_.

``` javascript
<Toggle {...props}>
  {({ on, getTogglerProps }) => (
    <div>
      {on ? "The button is on" : "The button is off"}
      <Switch {...getTogglerProps()} />
    </div>
  )}
</Toggle>
```

# Controlled Components

Controlled components tức là kỹ thuật lấy value hiện tại thông qua props hoặc state, và nó sử dụng các callback như _onchange()_ để thực hiện thay đổi props đó. sau đó pass các props hay state đó xuống các controlled component.

Nói đến cái này thì nên nói thêm cái khác là Uncontrolled Component, nó lấy và quản lý value trực tiếp từ DOM thông qua ***ref*** để tìm đến giá trị hiện tại.

# State reducer

Một reducer là một function đơn giản lấy input và trả lại output mà không thay đổi bất kỳ state của ứng dụng.

State Reducer được sử dụng để đưa vào component một function để generate một state mới dựa vào value được trả về từ reducer.

Pattern này khá giống với việc controlling một component props.

``` javascript
  toggleStateReducer = (state, changes) => {
    if (changes.type === 'forced') {
      return changes;
    }

    if (this.state.timeClicked >= 4) {
      return {...changes, on: false};
    }

    return changes;
  }
```

``` javascript
  <Toggle stateReducer={this.toggleStateReducer} >
    {this.props.children}
  </Toggle>
```

# Higher-Order Components

Đây là một kỹ thuật hầu hết những người làm ReactJS chắc chắn đã 1 lần sử dụng đến, nó đưa vào một Component và một vài logic sau đó return về nột component mới.

Nó là pattern được sử dụng trong hầu hết các thư việc của ReactJS.

``` javascript
const withToggle = (Component) => {
  const Wrapper = (props, ref) => (
    <Toggle.Consumer>
      {toggleState => <Component {...props} toggle={toggleState} ref={ref}/>}
    </Toggle.Consumer>
  );

  return Wrapper;
}
```

``` javascript
const MyToggleComponent = ({ toggle: { on, getTogglerProps } }) => {
  return (
    <div>
      {on ? "The button is on" : "The button is off"}
      <Switch {...getTogglerProps()} />
    </div>
  );
}
const MyWrappedToggleComponent = withToggle(MyToggleComponent);
```

# Kết luận

Trên là những pattern mình đã tìm hiểu được và cũng có một vài ví dụ, nó khá hữu ích khi chúng ta thực hiện trong dự án.

Tùy từng pattern sẽ có những ích lợi riêng, nhưng hầu hết là cho code chúng ta dễ sử dụng hơn và clean hơn nhiều.

Ở các bài sau mình sẽ chia sẽ các kỹ thuật khác thú vị hơn ạ.

Thank you so much!

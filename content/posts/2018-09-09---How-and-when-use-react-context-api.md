---
title: Làm thế nào và khi nào thì nên sử dụng React Context Api
date: "2018-09-09T23:46:37.121Z"
template: "post"
draft: false
slug: "/posts/how-and-when-use-react-context-api/"
category: "ReactJS"
tags:
  - "Javascript"
  - "ReactJS"
description: "Từ React 16.3 chắc hẳn mọi người đã biết đến 1 tính năng mới đó New Context Api, nhưng sử dụng nó như thế nào và trong trường hợp nào, nó có ích như thế nào thì trong bài này mình xin được chia sẽ 1 chút tìm hiểu của mình về nó"
---

chào mọi người, :hand:

Từ React 16.3 chắc hẳn mọi người đã biết đến 1 tính năng mới đó New Context Api, nhưng sử dụng nó như thế nào và trong trường hợp nào, nó có ích như thế nào thì trong bài này mình xin được chia sẽ 1 chút tìm hiểu của mình về nó.

# Những Problem mà Context Api có thể giải quyết:

React Context là cách mà các child Component có thể truy cập được đến value của một Parent Component.

Khi làm việc với React trong việc truyền props từ Parent Component đến các Child Component có lẽ chúng ta đã gặp phải trường hợp khá phổ biến như "Props Drilling ". Đúng với tên gọi của nó, Problem này xuất hiện khi chúng ta muốn lấy một state từ TOP đên BOTTOM của một Component, chúng ta phải truyền nó xuống từng Component trong Component tree một cách không cần thiết và phá phiền toái ( nhìn thôi đã phát điên rồi TOO CRAZY! ).

Thông thường để giải quyết cái này thì chúng ta tìm tới các giải pháp khá phổ biến như REDUX hay MOBX. Nhưng liệu nó có thích hợp cho những Project nhỏ và không quá phức tạp đòi hỏi phải nhờ tới Redux.

Thì Context Api là giải pháp để giải quyết problem (khó ưa) này. và tôi đã thích Context Api từ đó :D .

# Vậy React New Context Api trông như thế nào ?

Nó gồm 3 phần được mô tả khá rõ trong DOCs offical React.

## React.createContext

React.createContext được sử dụng để khởi tạo một Context và ném vào đó những giá trị khởi tạo. Và nó return về một object với Provider và Consumer. Provider và Consumer là một cặp. Mỗi Provider thì sẽ có 1 Consumer tường ứng.

```javascript
const Context = React.createContext();
```

## Provider

Provider component được sử dụng cao hơn trong tree Component và nhận một props là **value**. Bất cứ child Component nào trong tree đó cũng có thể access value đó được cung cấp bởi context provider.

```javascript
render() {
  return (
    <Provider value={this.state.contextValue}>
      {this.props.children}
    </Provider>
  );
}
```

Bất khì một Consumer vào được đặt trong tree của Provider đó cũng có thể access được value bất để có lồng sâu thế nào đi nữa. :D (thấy sâu sâu là thích rồi !).

## Consumer

Consumer sử dụng data được truyền từ Provider và hiển thì bằng cách sử dụng kỹ thuật render prop API.
Render prop Api là cách khá thú vị và hữu ích. Tôi thật sự thích làm việc với nó.

```javascript
render() {
  return (
    <Consumer>
      {contextValue => <Child arbitraryProp={contextValue} />}
    </Consumer>
  )
}
```

# Quan trọng là khi nào thì sử dụng nó?

Bạn có thể sử dụng nó trong trường hợp bạn muốn quản lý state một cách đơn giản. trong những project không quá phức tạp.
hay trong trường hợp bạn muốn ném state từ TOP xuống BOTTOM cho một thằng Component nào đó đang đợi dưới đó và không muốn các nhờ đến các thằng ở giữa ném xuống giúp. (mình cứ tự trừu tượng hoá như vậy).

Và mình cũng có làm một ví dụ nhỏ trong việc sử dụng nó trong việc switch Themes trong Project [Demo](https://vancuong.netlify.com/).

Tôi có một file SiteThemeContext.jsx như thế này:

```javascript
import React, { PureComponent } from 'react';
import PropTypes from 'prop-types';

import themes from '../styles/abstracts/themes';

export const SiteThemeContext = React.createContext();

class SiteThemeProvider extends PureComponent {
  constructor(props) {
    super(props);
    this.state = {
      theme: themes.theme1,
    };
    this.handleThemeChange = this.handleThemeChange.bind(this);
  }

  handleThemeChange = (e) => {
    const key = e.target.value;
    const theme = themes[key];
    this.setState({ theme });
  }

  render() {
    const { children } = this.props;
    return (
      <SiteThemeContext.Provider
        value={{
          ...this.state,
          handleThemeChange: this.handleThemeChange,
        }}
      >
        {children}
      </SiteThemeContext.Provider>
    );
  }
}

SiteThemeProvider.propTypes = {
  children: PropTypes.oneOfType([
    PropTypes.arrayOf(PropTypes.node),
    PropTypes.node,
  ]).isRequired,
};

export default SiteThemeProvider;
```

và một file Consumer đó là SiteThemeSelect.jsx

```javascript
import React from 'react';
import styled from 'styled-components';

import { SiteThemeContext } from '../../../context/SiteThemeContext';
import themes from '../../../styles/abstracts/themes';

const SelectWrapper = styled.div``;

const Select = styled.select``;

const SelectOpt = styled.option``;

const SiteThemeSelect = () => (
  <SiteThemeContext.Consumer>
    {({ handleThemeChange }) => (
      <SelectWrapper>
        <Select onChange={e => handleThemeChange(e)}>
          {Object.keys(themes).map((theme, index) => (
            <SelectOpt key={index} value={theme}>
              Theme {theme}
            </SelectOpt>
          ))}
        </Select>
      </SelectWrapper>
    )}
  </SiteThemeContext.Consumer>
);

export default SiteThemeSelect;
```

trong ví dụ trên tôi có sử dụng styled-component nó cho phép viết CSS in JS (tuyệt với lắm).
bằng cách áp dụng styled-component chúng ta dễ dàng sử dụng themes trong React khá thú vị.

# Tóm lại:

Trên đây là những chia sẽ của tôi về New Context Api và ví dụ về nó khá thú vị.
Hãy theo dõi blog và đừng ngại tương tác nhé.

Thank you so much!

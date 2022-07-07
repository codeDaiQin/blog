---
title: ReactDOM.createPortal
date: '2022-07-07'
tags: ['react']
draft: false
summary: 将子节点渲染到存在于父组件以外的 DOM 节点
---

# [ReactDOM.createPortal](https://zh-hans.reactjs.org/docs/react-dom.html#createportal)

**将子节点渲染到已 DOM 节点中的方式，该节点存在于 DOM 组件的层次结构之外。**

> ReactDOM.createPortal(child, container)

```tsx
import React from 'react'
import ReactDOM from 'react-dom'

type PropsType = {
  getContainer?: HTMLElement // 指定组件挂载得节点
}

const App: React.FC<PropsType> = (props) => {
  const { getContainer = document.body } = props
  return ReactDOM.createPortal(<div>hello world</div>, getContainer)
}
```

应用场景

```tsx
//
const Modal: React.FC = () => {
  // 子元素会超出父元素 导致显示不全
  // return <div style={{ height: 1000, width: 1000 }}>1</div>

  // 可以使用ReactDOM.createPortal使其渲染到body下 不受父元素overflow影响
  return ReactDOM.createPortal(<div style={{ height: 1000, width: 1000 }}>1</div>, document.body)
}

const App: React.FC = () => {
  const [visible, setVisible] = useState(false)

  return (
    <div style={{ height: 100, width: 100, overflow: 'hidden' }}>
      <Modal visible={visible} />
    </div>
  )
}
```

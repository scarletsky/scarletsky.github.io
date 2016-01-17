---
title: React-DnD 的使用
date: 2015-11-17 10:59:31
tags: react
---

## 介绍

React DnD 是一组 React 高阶组件，可以用来帮你构建复杂的拖拽接口，同时解耦你的组件。React DnD 非常适合像 Trello 和 Storify 这样的应用，在不同地方通过拖拽转移数据，而组件会改变它们的外观和应用的状态来响应拖拽事件。

## 基本用法

1. 把应用的根组件包装在 `DragDropContext` 中
2. 把可以拖拽的组件包装在 `DragSource` 中
  1. 设置 type
  2. 设置 spec，让组件可以响应拖拽事件
  3. 设置 collect，把拖拽过程中需要信息注入组件的 props
3. 把可以接受拖拽的组件包装在 `DropTarget` 中
  1. 设置 type
  2. 设置 spec，让组件可以响应拖拽事件
  3. 设置 collect，把拖拽过程中需要信息注入组件的 props
4. 完

翻译成代码就是：

```js
// 1
import HTML5Backend from 'react-dnd-html5-backend';
import { DragDropContext } from 'react-dnd';

class App { ... }
export default DragDropContext(HTML5Backend)(App);

/*---------------------------*/

// 2
import { DragSource } from 'react-dnd';

class MyComponent { ... }
export default DragSource(type, spec, collect)(MyComponent);

/*---------------------------*/

// 3
import { DropTarget } from 'react-dnd';

class MyComponent2 { ... }
export default DropTarget(types, spec, collect)(MyComponent2);
```

这样，MyComponent 就变得可以拖拽，而 MyComponent2 就变得可以接受拖拽了，但这并不代表 MyComponent 可以放到 MyComponent2 中！

## 一些概念

React DnD 中有一些特殊的概念，理解这些概念之后才能活用这个库！

- `Backend` 实现 DnD 的方式，默认是用 HTML5 DnD API，它不能在触屏环境下工作，而且在 IE 下可定制性比其他浏览器弱。你也可以用自己实现，具体请看官方文档。
- `Items` 拖拽数据的表现形式，用 Object 来表示。譬如，要拖拽一张卡片，那这张卡片的**数据**的表现形式可能是 `{ id: xxx, content: yyy }`。
- `Types` 表示拖/放组件的兼容性，`DragSource` 和 `DropTarget` 必须指定 `type`。只有在 `type` 相同的情况下，`DragSource` 才能放到 `DropTarget` 中。
- `Monitors` 用来响应拖拽事件，可以用来更新组件的的状态。
- `Connectors` 底层接触 DOM 的东西，用来封装你的组件，让你的组件有拖拽的特性。一般在 collect 方法中指定，然后注入到组件的 props 中，最后 render 方法中包装你自己的组件。
- `DragSource && DropTarget` 把上面的概念都绑在一起的东西，也是真正跟你的组件打交道的东西。


## 主要 API 介绍

这些主要 API 都是通过包装你的组件，然后返回一个新的组件。

### DragDropContext(backend)

- `backend` 实现 DnD 的方式，一般是 HTML5Backend

```js
export default DragDropContext(HTML5Backend)(App);
```

-----


### DragSource(type, spec, collect)
### DropTarget(type, spec, collect)


- `type` 必须。type 是自定义的，可以是 string，symbol，也可以是用一个函数来返回该组件的其他 props。该组件只能放到相同 type 的 DropTarget 中。
- `spec` 必须。一个带有特定方法的纯 Object，里面是一些响应拖拽事件的方法。
- `collect` 必须。一个函数返回一个 Object，这个 Object 会注入到组件的 props 中。
- `options` 可选。除非有性能问题，否则不需要关心这个参数。

```js
const type = 'xxx';
const spec = { ... };
function collect(connect, monitor) { ... }

export default DragSource(type, spec, collect)(MyComponent);
export default DropTarget(type, spec, collect)(MyComponent2);
```

-----

### DragSource#spec

让你的组件响应 dnd 相关事件，支持以下方法：

- `beginDrag(props, monitor, component)` **必须**
- `endDrag(props, monitor, component)` 可选
- `canDrag(props, monitor)` 可选
- `isDragging(props, monitor)` 可选

参数含义如下：

- `props` 组件当前的 props
- `monitor` 是一个 `DragSourceMonitor` 实例，用来查询当前 drag state 的信息。
- `component` 表示当前组件，可以省略。


```js
const spec = {
    beginDrag(props) {
        return { 
        	id: props.id, 
        	content: props.content
        }
    }
    //...
}
```

-----

### DropTarget#spec

让你的组件响应 dnd 相关事件，支持以下方法：

- `drop(props, monitor, component)` 可选，响应 drop 事件
- `hover(props, monitor, component)` 可选
- `canDrop(props, monitor)` 可选

参数含义如下：

- `props` 组件当前的 props
- `monitor` 是一个 `DropTargetMonitor` 实例，用来查询当前 drag state 的信息。
- `component` 表示当前组件，可以省略。

```js
const spec = {
	drop(props, monitor, component) {
		// 获取正在拖放的数据
		const item = monitor.getItem();
		// 更新组件状态
		component.setState({
			item
		})
		
	}
}
```

-----

### DragSource#collect(connect, monitor)
### DropTarget#collect(connect, monitor)

返回一个 object，这个 object 可以会注入到组件的 props 中。

- `connect` 一个 `DragSourceConnector`/`DropTargetConnector` 实例，可以用 `connect.dragSource()`/`connect.dropTarget()` 来封装我们的组件。
- `monitor` 一个 `DragSourceMonitor`/`DropTargetMonitor` 实例，用来查询当前拖拽的信息。

```js
function collect(connect, monitor) {
    return {
        isDragging: monitor.isDragging(),
        connectDragSource: connect.dragSource()
    }
}

class MyComponent extends Component {
	render() {
		// 可以访问 collect 中返回的  object
		const { isDragging, connectDragSource } = this.props;
		// 需要用 connect.dragSource()/connect.dropTarget() 封装自己的组件
		return connectDragSource(
			<div>123</div>
		)
	}
}

```

## 具体例子
- [演示](http://gaearon.github.io/react-dnd/examples-chessboard-tutorial-app.html)
- [代码](https://github.com/gaearon/react-dnd/tree/master/examples)


## 参考资料
[官方文档](http://gaearon.github.io/react-dnd/)
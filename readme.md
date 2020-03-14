# 原生JavaScript完成todolist

*作者：Dary*

## ES5数组API改造

### 一、首先，从所有的todos中筛选出已完成和未完成我们可以使用**filter**

改造前：

```javascript
var hasFinishedTodos = []
for (var i = 0; i < todos.length; i++) {
    if (todos[i].hasFinished) hasFinishedTodos.push(todos[i])
}
var unFinishedTodos = []
for (var i = 0; i < todos.length; i++) {
    if (!todos[i].hasFinished) unFinishedTodos.push(todos[i])
}

```

改造后：

```javascript
var hasFinishedTodos = todos.filter(function (todo) {
    return todo.hasFinished
})
var unFinishedTodos = todos.filter(function (todo) {
    return !todo.hasFinished
})
```



### 二、计算未完成事项的总时间，可以使用**reduce**

改造前：

```javascript
var hours = 0
for (var i = 0; i < unFinishedTodos.length; i++) {
    hours += unFinishedTodos[i].time
}
```

改造前：

```javascript
var hours = unFinishedTodos.reduce(function (time, todo) {
    return time + todo.time
}, 0)
```



### 三、循环数组拼接html字符串可以使用**reduce**

改造前：

```javascript
var hasFinishHTML = ''
for (var i = 0; i < hasFinishedTodos.length; i++) {
    hasFinishHTML += '<div class="column is-one-quarter" data-id="${todo.id}">'+
        '...'+
    '</div>'
}
document.querySelector('#has-finish-wrap').innerHTML = hasFinishHTML
```

改造后：

```javascript
document.querySelector('#has-finish-wrap').innerHTML = 
    hasFinishedTodos.reduce(function (html, todo) {
    	return html + '<div class="column is-one-quarter" data-id="${todo.id}">'+
        	'...'+
    	'</div>'
}, '')
```



### 四、事件委托通过class判断事件源可以使用**Array.from**和**includes**配合

改造前：

```javascript
var arrClass = e.target.className.split(/\s+/)
if (arrClass.indexOf('btn-toggle') !== -1) {
    // ...
}
```

改造后：

```javascript
if (Array.from(e.target.classList).includes('btn-toggle')) {
    // ...
}
```



### 五、切换完成状态修改数组可以使用**map**

改造前：

```javascript
for (var i = 0; i < todos.length; i++) {
    if (todos[i].id === id) todos[i].hasFinished = !todos[i].hasFinished
}
```

改造后：

```javascript
todos = todos.map(function (todo) {
    if (todo.id === id) todo.hasFinished = !todo.hasFinished
    return todo
})
```



### 六、删除待办事项可以使用**filter**

改造前：

```javascript
for (var i = 0; i < todos.length; i++) {
    if (todos[i].id === id) {
        todos.splice(i, 1)
        i--
    }
}
```

改造后：

```javascript
todos = todos.filter(function (todo) {
    return todo.id !== id
})
```


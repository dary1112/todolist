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





## 使用Object.defineProperty改造

代码如下：

```javascript
var obj = {}
Object.defineProperty(obj, 'todos', {
    get () {
        return todos
    },
    set (value) {
        todos = value
        // 每一次todos被修改都调用render方法
        render()
    }
})
```

至此，我们把代码中的`todos`都改成`obj.todos`，并且只需要在页面初始调用一次`render()`，其他地方的渲染都会在`set`方法里直接运行，不需要每次调用。

**这样我们就可以集中精力在我们的业务逻辑上而不用担心页面渲染了**

但是，现在我们会发现一个问题，就是添加事项添加不上了，因为我们原本用的是

```javascript
todos.push(todo)
```

而`push`修改的是源数组，这样并不会触发`obj`的`set`方法，解决方案如下：

```javascript
obj.todos = obj.todos.concat([todo])
```


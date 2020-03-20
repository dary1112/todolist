# 原生JavaScript完成todolist

*作者：Dary*

## ES6改造

### 一、很多用到函数的地方只要不涉及this的问题都可以使用箭头函数



### 二、定义变量和常量使用let和const



### 三、从所有的todos中筛选出已完成和未完成我们可以使用**filter**

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
var hasFinishedTodos = todos.filter(todo => todo.hasFinished)
var unFinishedTodos = todos.filter(todo => !todo.hasFinished)
```



### 四、计算未完成事项的总时间，可以使用**reduce**

改造前：

```javascript
var hours = 0
for (var i = 0; i < unFinishedTodos.length; i++) {
    hours += unFinishedTodos[i].time
}
```

改造后：

```javascript
var hours = unFinishedTodos.reduce((time, todo) => time + todo.time, 0)
```



### 五、循环数组拼接html字符串可以使用**reduce**

### 六、使用模板字符串

改造前：

```javascript
var hasFinishHTML = ''
for (var i = 0; i < hasFinishedTodos.length; i++) {
    hasFinishHTML += '<div class="column is-one-quarter" data-id="' + todo.id + '">'+
        '...'+
    '</div>'
}
document.querySelector('#has-finish-wrap').innerHTML = hasFinishHTML
```

改造后：

```javascript
document.querySelector('#has-finish-wrap').innerHTML = 
    hasFinishedTodos.reduce((html, todo) => {
    	return html + 
        `
            <div class="column is-one-quarter" data-id="${todo.id}">
            ...
            </div>
        `
}, '')
```



### 七、事件委托通过class判断事件源可以使用**Array.from**和**includes**配合

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



### 八、切换完成状态修改数组可以使用**map**

改造前：

```javascript
for (var i = 0; i < todos.length; i++) {
    if (todos[i].id === id) todos[i].hasFinished = !todos[i].hasFinished
}
```

改造后：

```javascript
todos = todos.map(todo => {
    if (todo.id === id) todo.hasFinished = !todo.hasFinished
    return todo
})
```



### 九、删除待办事项可以使用**filter**

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
todos = todos.filter(todo => todo.id !== id)
```



### 十、判断是否全部已完成可以使用every

```javascript
const isAllFinish = this.todos.every(todo => todo.hasFinished)
```



### 构造新增todo可以使用解构赋值

```javascript
var title = this.inputTitle.value
var time = Number(this.inputTime.value)
var todo = {
	title,
	time
}
```



### 十一、新增todo时使用扩展运算符

```javascript
this.todos.push({
	...todo,
	id: Date.now(),
	hasFinished: false
})
```



### 十二、class改写面向对象

```javascript
class TodoList {
    constructor () {
        this.todos = todos
        // ...
    }
    render () {
        // ...
    }
    // ...
}
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



## 使用数据劫持

```javascript
function observer(obj) {
    if(typeof obj === 'object') {
        for (let key in obj) {
            // defineReactive 方法设置get和set
            defineReactive(obj, key, obj[key])
        }
    }
}

function defineReactive(obj, key, value) {
	// value 可能也是对象，递归劫持
    observer(value)
    
    Object.defineProperty(obj, key, {
        get () {
            return value
        },
        set (val) {
            console.log('数据更新了')
            value = val
            todolist.render()
        }
    })
}
```

但是，如果对象是一个数组，Object.defineProperty就无法起作用了

我们可以重写一下数组的方法

```javascript
let arr = ['push', 'slice', 'shift', 'unshift', 'splice']
arr.forEach(method => {
    let oldMethod = Array.prototype[method]
    Array.prototype[method] = function (value) {
        console.log('数据更新了')
        oldMethod.call(this, value)
        todolist.render()
    }
})
```

在最后，调用一下`observer`方法即可

```javascript
observer(todolist)
```


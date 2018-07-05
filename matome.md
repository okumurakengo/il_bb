# backbone.jsのまとめ

title |  |  説明 |
|---|---|---|
| Model | obj | Backbone.Model.extend |
| Model | ins | new Task |
| View | obj| Backbone.View.extend |
| View | ins | insnew TaskView |
| Collection | obj| Backbone.Collection.extend |
| Collection | ins| new Tasks |

## Model obj

```js
var Task = Backbone.Model.extend({
    defaults:{
        title:"do sumething!",
        completed:false
    },
    validate:function(attrs){
        if(_.isEmpty(attrs.title)){
            return "title must not be empty!"
        }
    },
    initialize:function(){
        this.on('invalid',function(model,error){
            $('#error').html(error)
        })
    },
    toggle:function(){
        this.set('completed',!this.get('completed'))
    }
})
```

```js
var Task = Backbone.Model.extend({
    defaults:{},
    validate:function(attrs){},
    initialize:function(){},
    toggle:function(){}
})
```

## Model ins

```js
var task1 = new Task({
    title:'do it!',
    completed:true
})
```

- 値を見れる
```js
insModel.toJSON()
```

- 値を更新する
```js
insModel.set(title:'new value')
insModel.set({title:''},{validate:true})
```


---

## View obj

```js
var TaskView = Backbone.View.extend({
    tagName:'li',
    // className:'liClass',
    // id:'liId'
    events:{
        "click .command":"sayHello"
    },
    sayHello:function(){
        alert('hello')
    },
    template:_.template($('#task-template').html()),
    render:function(){
        var template  = this.template(this.model.toJSON())
        this.$el.html(template)
        return this
    }
})
```

```js
var TaskView = Backbone.View.extend({
    tagName:'li',
    // className:'liClass',
    // id:'liId'
    events:{},
    sayHello:function(){},
    template:_.template('<p><%- title %></p>'),
    render:function(){}
})
```

## View ins

```js
var taskView = new TaskView({model:task})
console.log(taskView.render().el)
$('body').append(taskView.render().el)
```

---

## Collection obj

```js
var Tasks = Backbone.Collection.extend({
    model:Task
})
```


## Collection ins

```js
var tasks = new Tasks([
    {
        title:'task1',
        completed:true
    },
    {
        title:'task2',
    },
    {
        title:'task3',
    },
]);
```

---

# todoアプリまとめ

## オブジェクトとインスタンスまとめ

- Model : Task

```js
var Task = Backbone.Model.extend({...})
```

- Collection : Tasks

```js
var Tasks = Backbone.Collection.extend({...})
```

- View : TaskView

```js
var TaskView = Backbone.View.extend({...})
```

- View : TasksView

```js
var TasksView = Backbone.View.extend({...})
```

- View : AddTaskView

```js
var AddTaskView = Backbone.View.extend({...})
```

```js
var tasks = new Tasks([...])
```

```js
var tasksView = new TasksView({...})
```

```js
var addTaskView = new AddTaskView({...})
```

```js
$('#tasks').html(tasksView.render().el)
```

## データの流れ

1. 配列のデータ

```js
[
    {title:'task1',completed:true},
    {title:'task2'},
    {title:'task3'}  
]
```

2. インスタンス化

```js
var tasks = new Tasks([
    {title:'task1',completed:true},
    {title:'task2'},
    {title:'task3'}  
])
```

3. コレクションが作成される

```js
var Tasks = Backbone.Collection.extend({model:Task})
```

5. 配列に対してプロパティがない場合はデフォルト値設定、initializeも実行

```js
var Task = Backbone.Model.extend({
    defaults:{
        title:'do something',
        completed:false
    },
    // ...
    initialize:function(){
        this.on('invalid',function(model,error){
            $('#error').html(error)
        })
    }
})
```
6. TasksViewが作成される

```js
var tasksView = new TasksView({collection:tasks})
```

7. TasksViewのinitializeが実行

```js
var TasksView = Backbone.View.extend({
    tagName:'ul',
    initialize:function(){
        this.collection.on('add',this.addNew,this)
        this.collection.on('change',this.updateCount,this)
        this.collection.on('destroy',this.updateCount,this)
    },
    // ...
})
```

8. addTaskViewが実行される

```js
var addTaskView = new AddTaskView({collection:tasks})
```

9. タグを取得

```js
var AddTaskView = Backbone.View.extend({
    el: '#addTask',
    // ...
})
```

10. tasksViewのrenderを実行

```js
$('#tasks').html(tasksView.render().el)
```

11. tasksViewのrenderを実行が実行される

```js
var TasksView = Backbone.View.extend({
    render:function(){
        this.collection.each(function(task){
            console.log(task.toJSON())
            var taskView = new TaskView({model:task})
            this.$el.append(taskView.render().el)
        },this)
        this.updateCount()
        return this
    }
})
```

12. new TaskViewされ、render()される

```js
var TaskView = Backbone.View.extend({
    tagName:'li',
    initialize:function(){
        this.model.on('destroy',this.remove,this)
        this.model.on('change',this.render,this)
    },
    template: _.template($('#task-template').html()),
    render:function(){
        var template = this.template(this.model.toJSON())
        this.$el.html(template)
        return this
    }
})
```

13. this.updateCount()が実行

```js
var TasksView = Backbone.View.extend({
    updateCount:function(){
        var upcompletedTasks = this.collection.filter(function(task){
            return !task.get('completed')
        })
        $('#count').html(upcompletedTasks.length)
    },
})
```

---

## ブラウザイベント

A.1. deleteボタン

```js
var TaskView = Backbone.View.extend({
    initialize:function(){
        this.model.on('destroy',this.remove,this)
    },
    events:{
        'click .delete':'destroy',
    },
    destroy:function(){
        if(confirm('are you sure?')){
            this.model.destroy()
        }
    },
    remove:function(){
        this.$el.remove()
    },
})
```

B.1. toggleボタン

```js
var TaskView = Backbone.View.extend({
    initialize:function(){
        this.model.on('change',this.render,this)
    },
    events:{
        'click .toggle':'toggle'
    },
    toggle:function(){
        this.model.set('completed',!this.model.get('completed'))  
    },
    template: _.template($('#task-template').html()),
    render:function(){
        var template = this.template(this.model.toJSON())
        this.$el.html(template)
        return this
    }
})
```

C.1. 新規taskをサブミット

```js
var AddTaskView = Backbone.View.extend({
    // el: '#addTask',
    events:{
        'submit':'submit'
    },
    submit:function(e){
        e.preventDefault();
        // var task = new Task({title:$('#title').val()})
        var task = new Task()
        if(task.set({title:$('#title').val()},{validate:true})){
            this.collection.add(task)
            $('#error').empty()
        }
    }
})
```

C.2. バリデーション

```js
var Task = Backbone.Model.extend({
    validate:function(attrs){
        if(_.isEmpty(attrs.title)){
            return 'title must not be empty!'
        }
    },
})
```

C.3. 

```js
var TasksView = Backbone.View.extend({
    initialize:function(){
        this.collection.on('add',this.addNew,this)
    },
    addNew:function(task){
        var taskView = new TaskView({model:task})
        console.log(taskView)
        this.$el.append(taskView.render().el)
        $('#title').val('').focus()
        this.updateCount()
    },
})
```

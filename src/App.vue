<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/axios/0.19.2/axios.min.js"></script>

<template>
    <div class="container my-3" id="app">
        <div>
            {{demo}}
        </div>
        <div class="input-group mb-3">
            <div class="input-group-prepend"><span class="input-group-text" id="basic-addon1">待辦事項</span></div>
            <input class="form-control" type="text" placeholder="準備要做的任務" v-model="newTodo" @keyup.enter="addTodo"/>
            <div class="input-group-append">
            <button class="btn btn-primary" type="button" @click="addTodo">新增</button>
            </div>
        </div>
        <div class="card text-center">
            <div class="card-header">
            <ul class="nav nav-tabs card-header-tabs">
                <li class="nav-item"><a class="nav-link" href="#" :class=" {'active' : visibility == 'all' } " @click="visibility='all'">全部</a></li>
                <li class="nav-item"><a class="nav-link" href="#" :class="{'active' : visibility == 'active'}" @click="visibility='active'">進行中</a></li>
                <li class="nav-item"><a class="nav-link" href="#" :class="{'active' : visibility == 'completed'}" @click="visibility='completed'">已完成</a></li>
            </ul>
            </div>
            <ul class="list-group list-group-flush text-left">
            <li class="list-group-item" v-for="item in filteredTodos" @dblclick="editTodo(item)">
                <div class="d-flex" v-if="item.id !== cacheTodo.id">
                <div class="form-check">
                    <input class="form-check-input" :id="item.id" type="checkbox" v-model="item.completed"/>
                    <label class="form-check-label" :for="item.id" :class="{'completed':item.completed}">{{ item.title }}</label>
                </div>
                <button class="close ml-auto" type="button" aria-label="Close" @click="removeTodo(item)"><span aria-hidden="true">×</span></button>
                </div>
                <input class="form-control" type="text" v-model="cacheTitle" v-if="item.id == cacheTodo.id" @keyup.esc="cancelEdit" @keyup.enter="doneEdit(item)"/>
            </li>
            </ul>
            <div class="card-footer d-flex justify-content-between">
            <section>
                還有  <span>{{activeTodosLength}}</span> 筆任務未完成</section><a href="#" @click="cleanTodo">清除所有任務</a>
            </div>
        </div>
        </div>
</template>


<script>

export default {
    data() {
        return {
            newTodo:'',
            todos: [
                {
                    id: '0',
                    title: 'hello',
                    completed: false
                }
            ],
            visibility: 'all',
            cacheTodo: {},
            cacheTitle: '',
            demo:''
        }
    },
    created: function () {
        var cros="https://cors-anywhere.herokuapp.com/"
        var api="https://kefusystem.ccggww.me/api/get/list"
        var ajaxAPI= cros + api
        var ajaxAPI= cros + api
        this.getApi(ajaxAPI);
    }, 
    methods: {
        postApi: function(url){
            axios({
                method: 'post',
                url: url + '8',
                data: {
                    id: '9',
                    title: 'test',
                    completed: false
                }
            })
        },
        getApi: function(url){
            axios.get(url)
            .then((res) =>{
                this.todos = res.data.rows
                this.demo = res.data.rows  //test
            })
            .catch((err)=>{
                console.log(err)
            })
        },
        addTodo: function(){
            var value = this.newTodo.trim()
            var timestamp = Math.floor(Date.now())
            this.postApi("https://kefusystem.ccggww.me/api/save/");
            if ( !value ) { return }
            this.todos.push(
                {
                    id: timestamp,
                    title: value,
                    computed: false
                }
            ),
            this.newTodo = ''
        },
        removeTodo: function(todo){
            var newIndex = this.todos.findIndex(function(item){
                return todo.id === item.id
            })
            this.todos.splice(newIndex,1)
          },
        editTodo: function(item){
            this.cacheTodo = item
            this.cacheTitle = item.title
        },
        cancelEdit: function(){
            this.cacheTodo = {}
        },
        doneEdit: function(item){
            item.title = this.cacheTitle
            this.cacheTitle = ''
            this.cacheTodo = {}
        },
        cleanTodo: function(todo){
            allTodo = this.todos
            allTodo.splice(0,allTodo.length)
          },
    },
    computed: {
        filteredTodos: function(){
            var nowTab = this.visibility
            var showList = []
            switch (nowTab) {
                case 'all':
                    showList = this.todos
                    break
                case 'active':
                    this.todos.forEach(function(item){
                        if ( !item.completed ) { showList.push(item) }
                    })
                    break
                case 'completed':
                    this.todos.forEach(function(item){
                        if ( item.completed ) { showList.push(item) }
                    })
            }
            return showList
        },
        activeTodosLength: function(){
            return this.todos.filter(item => !item.completed ).length
            // var sum = 0
            // this.todos.forEach(function(item){
            //     if ( !item.completed ) { sum ++ }
            // })
            // return sum
        }
    }
}
</script>
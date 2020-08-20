<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/axios/0.19.2/axios.min.js"></script>

<template>
    <div class="container my-3" id="app">
        <!-- <div>
            {{demo}}
        </div> -->
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
            <li class="list-group-item" v-for="(item,index) in filteredTodos" :key="index" @dblclick="editTodo(item)">
                <div class="d-flex" v-if="item.id !== cacheTodo.id">
                <div class="form-check">
                    <input class="form-check-input" :id="item.id" type="checkbox" v-model="item.completed"/>
                    <label class="form-check-label" :for="item.id" :class="{'completed':item.completed}">{{ item.title }}</label>
                </div>
                <button class="close ml-auto" type="button" aria-label="Close" @click="removeTodo(item.id)"><span aria-hidden="true">×</span></button>
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
        //tip:網址可以短是因爲在config/index.js做掉了proxy table那邊
        //參考網站https://segmentfault.com/a/1190000014265711
        this.getList();
        // this.getApi("/get/list");
    }, 
    methods: {
        //tip:這裏複寫dopost方法
        postApi: function(url){
            this.doPost(url+"/8",
                {
                    id: "9",
                    title: "test",
                    completed: false
                },(res)=>{
                    console.log(res)
                })
        },
        //  //tip:這裏複寫dopost方法
        getList(){
            this.doGet("/get/list",{},(res)=>{
                this.todos = res.data.rows;
                this.demo = res.data.rows; //test
            })
        },
        addTodo: function(){
            var newTodoStr = this.newTodo.trim()
            if ( !newTodoStr ) { return }
            let submitData= {
                id: timestamp,
                title: newTodoStr,
                computed: false
            };
            var timestamp = Math.floor(Date.now())
            this.doPost(`/save/${submitData.id}`,submitData,(res)=>{
                this.getList();
                this.newTodo = '';
            });
        },
        removeTodo: function(id){
            this.doPost(`/del/${id}`,{},()=>{this.getList();})
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
        //tip:這裏傳入 successFunction 然後使用IIFE
        doGet(url, param,successFunction){
            this.$http.get(`${this.HOST}${url}`,{
                params:{
                    ...param
                }
            }).then((req)=>{
                successFunction(req)
            })
        },
        //tip:這裏傳入 successFunction 然後使用IIFE
        //header 變更爲josn不然原本是form
        //參考網站：https://ithelp.ithome.com.tw/articles/10209714
        doPost(url, data,successFunction) {
            const headers={
                'Content-Type': 'application/json',
            };
            const _this = this;
            this.$http.post(this.HOST+url,`"${JSON.stringify(data).replace(/\"/g,"\\\"")}"`,{headers}).then((req)=>{
                successFunction(req)
            })
        }
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
        getNewKey(){
          return Math.max(...this.todos.map(el => +el.key)) //4
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
<template>
    <div class="container-fluid full-height">
        <div class="row full-height">
            <div class="col-md-4 todo-list">
                <!-- Todo Header -->
                <div class="list-header">
                    <h2>{{title}}</h2>
                </div>
                <!-- Todo Header End -->
                <div class="clr20"></div>
                <!-- Todo Filters and Add button -->
                <div class="col-md-12 text-center">
                    <button type="button" class="btn btn-primary pull-left" v-on:click="showTodoForm" v-bind:disabled="is_edit" title="Add Todo">
                        <span class="glyphicon glyphicon-plus"></span>
                    </button>
                    <div class="btn-group pull-right" role="group" aria-label="...">
                        <button title="Show Completed" type="button" class="btn btn-default" v-bind:class="{'active' : active_filter=='complete'}" v-on:click="updateTodoFilter('complete')">Completed({{todoCounter('complete')}})</button>
                        <button title="Show Pending" type="button" class="btn btn-default" v-bind:class="{'active' : active_filter=='pending'}" v-on:click="updateTodoFilter('pending')">Pending({{todoCounter('pending')}})</button>
                        <button title="Show All" type="button" class="btn btn-default" v-bind:class="{'active' : active_filter=='all'}" v-on:click="updateTodoFilter('all')">All({{todoCounter('all')}})</button>
                    </div>
                </div>
                <!-- Todo Filters End -->
                <div class="clr20"></div>
                <!-- Todo Filters List -->
                <div class="list-group">
                    <template v-if="filterList.length > 0">
                    <a href="#" 
                        class="list-group-item" 
                        v-for="todo in filterList" 
                        v-bind:key="todo.id" 
                        v-bind:class="{'active' : is_edit===true && currentTodo == todo.id, 'todos_completed' : todo.is_done===true}"
                        >
                        {{ todo.text }}
                        <span class="pull-right" v-show="is_edit===false">
                            <button class="btn btn-xs btn-default" v-on:click="doneTodo(todo.id)" v-if="todo.is_done===false" title="Complete Todo">
                                <span class="glyphicon glyphicon-ok"></span>
                            </button>
                            <button class="btn btn-xs btn-default" v-on:click="editTodo(todo.id)" title="Update Todo">
                                <span class="glyphicon glyphicon-pencil"></span>
                            </button>
                            <button class="btn btn-xs btn-danger" v-on:click="deleteTodo(todo.id)" title="Delete Todo">
                                <span class="glyphicon glyphicon-trash"></span>
                            </button>
                        </span>
                    </a>
                    </template>
                    <template v-else>
                        <div class="alert alert-info">No Todo</div>
                    </template>    
                </div>
                 <!-- Todo Filters List End -->
            </div>
            <!-- Todo Save/Cancvel buttons -->
            <div class="col-md-8 todo-editor">
                <div v-show="is_edit" class="full-height">
                    <textarea v-model="todo_text" autofocus="autofocus" autocomplete="false" ref="todo_text" v-bind:focus="is_edit" placeholder="Please enter your todo..."></textarea>
                    <button type="button" class="btn btn-primary" v-on:click="addUpdateTodo">{{ buttonText }}</button>
                    <button type="button" class="btn btn-default" v-on:click="resetEditing">Cancel</button>
                </div>
            </div>
            <!-- Todo Save/Cancvel buttons End -->
        </div>
    </div>    
</template>
<script>
export default {
  // Set the name of Vue Component  
  name:'Todo',
  data () {
    return{
        title:'Todo List', // Set heading
        active_filter:'all', // Set default filter on list
        todos : [
            { id:1, text:'Todo text 1', is_done:false },
            { id:2, text:'Todo text 2', is_done:false },
            { id:3, text:'Todo text 3', is_done:false },
        ],
        currentTodo : null, // Set current todo while updating
        is_edit : false, // Set status of updating/adding a todo
        todo_text:'' // Store the text of todo adding/updating
    }
  },
  computed:{
    /** 
    * Generate todo button text 
    * @param : none
    * @return : String
    */  
    buttonText () {
        return (this.is_edit===true && this.currentTodo === null) ? 'Add Todo' : 'Update Todo'
    },
    /** 
    * Filter the todo list based on active_filter(All, Completed, Pending)
    * @param : none
    * @return : Array
    */ 
    filterList () {
        if(this.active_filter == 'all') {
           this.setPageTitle("Todo List("+this.todoCounter('all')+")")
           return this.todos;
        }else if(this.active_filter == 'pending') {
           this.setPageTitle("Pending List("+this.todoCounter('pending')+")")
           return this.todos.filter(todo => todo.is_done===false)
        }else if(this.active_filter == 'complete') {
           this.setPageTitle("Complete List("+this.todoCounter('complete')+")")
           return this.todos.filter(todo => todo.is_done===true)
        }
    }
  },
  methods : {
    /* Setter Functions  */
    /** 
    * Set the page title
    * @param : {String} title
    * @return : None
    */ 
    setPageTitle (title) {
         document.title = typeof(title != "undefined") ? title : 'Todo List'
    },  
    /** 
    * Generate an Id for todo
    * @param : None
    * @return : {Number}
    */ 
    generateId () {
       return this.todos.length+1;
    }, 
    /** 
    * Generate a counter for filter type(All, Completed, Pending)
    * @param : {String} type {all,pedning,complete}
    * @return : {Number}
    */  
    todoCounter (type) {
        if(type == 'all') {
            return this.todos.length
        }else if(type == 'pending') {
            return this.todos.filter(todo => todo.is_done===false).length
        }else if(type == 'complete') {
            return this.todos.filter(todo => todo.is_done===true).length
        }
    },  
    /** 
    * Generate a counter for filter type(All, Completed, Pending)
    * @param : {Number} id
    * @return : {String}
    */ 
    activateCurrentEdit (id) {
        return {'active' : this.is_edit===true && this.currentTodo == id  }
    }, 
    /** 
    * Update todo filter
    * @param : {String} filter (All, Completed, Pending)
    * @return : None
    */ 
    updateTodoFilter (filter) {
        this.active_filter = filter
    },
    /* End Setter Functions */

    /* Todo Crud Functions */
    
    /** 
    * Show Todo form and update the editing state
    * @param : None
    * @return : None
    */
    showTodoForm() {
        this.is_edit = true; 
        this.setPageTitle("Add Todo")
    },
    /** 
    * Insert a new todo and reset editing state
    * @param : None
    * @return : None
    */
    addTodo () {
        const newTodo = {
            id : this.generateId(),
            text : this.todo_text,
            is_done : false
        }
        this.todos.push(newTodo);
        this.resetEditing();
    },
    /** 
    * Add/Update a todo based on editing state
    * @param : None
    * @return : None
    */
    addUpdateTodo () {
        if(this.is_edit===true && this.currentTodo === null){
            if(this.validTodo() === true) {
                this.addTodo();
            }
        }else{
            this.updateTodo();
        }
    },
    /** 
    * On the editing state
    * @param : {Number} id(Todo id)
    * @return : None
    */
    editTodo (id) {
        if(this.isValid(id) === true) {
            this.is_edit = true;
            this.currentTodo = id;
            this.todo_text = this.getTodoText(id);
            this.setPageTitle("Update Todo")
        } else{
            alert('Todo is not found.');
        } 
    },
    /** 
    * Update the todo
    * @param : None
    * @return : None
    */
    updateTodo () {
       if(this.isValid(this.currentTodo) === true) {
            if(this.validTodo() === true) {
                this.setTodoText(this.currentTodo);
                this.resetEditing();
            }
        } else{
            alert('Todo is not found.');
        } 
    },
    /** 
    * Update the status of Todo to done
    * @param : {Number} id
    * @return : None
    */
    doneTodo (id) {
        var index = this.getTodoIndexbyId(id);
        if(index > -1) {
            this.todos[index].is_done = true;
        } else {
            alert('Todo is not found.');
        } 
    },
    /** 
    * Delete the todo
    * @param : {Number} id
    * @return : None
    */
    deleteTodo (id) {
        if(this.isValid(id) === true) {
            if(confirm('Are you sure you want to delete?') === true){
                var index = this.getTodoIndexbyId(id);
                if(index > -1) {
                    this.todos.splice(index,1);
                }
            }
        }else{
            alert('Todo is not found.');
        } 
    },
    /* End Todo Crud Functions */
    
    /* Helper Functions  */
    
    /** 
    * Get the todo text by id
    * @param : {Number} id
    * @return : {String}
    */
    getTodoText (id) {
        var index = this.getTodoIndexbyId(id);
        if(index > -1) {
            return this.todos[index].text
        }
    },
    /** 
    * Get the todo text by id
    * @param : {Number} id
    * @return : {String}
    */
    setTodoText (id) {
        var index = this.getTodoIndexbyId(id);
        if(index > -1) {
            this.todos[index].text = this.todo_text
        }
    },
    /** 
    * Get the todo index by todo id
    * @param : {Number} id
    * @return : {Number} index
    */
    getTodoIndexbyId (id) {
        return this.todos.map(todo => {
            return todo.id
        }).indexOf(id)
    },
    /** 
    * Reset editing state
    * @param : None
    * @return : None
    */
    resetEditing () {
        this.todo_text = '';
        this.is_edit = false;
        this.currentTodo = null;
        document.title = "Todo List("+this.todoCounter('all')+")";
    },
    /** 
    * Validate todo text 
    * @param : None
    * @return : None
    */
    validTodo () {
        if(this.todo_text.length <= 0) {
            alert('Please add a todo text.');
            return false;
        }else{
            return true;
        }
    },
    /** 
    * Check a todo is exits by todo id
    * @param : None
    * @return : None
    */
    isValid (id) {
        if(this.getTodoIndexbyId(id) > -1)
            return true;
        else
            return false;    
    }
    /* End of Helper Functions  */
  }
}
</script>


<template>
  <div class="container">
  <div class="header">
  <div>
    <input type="text" v-model="addTask" class="scanf"
     @keyup.enter="addTodo" placeholder="请输入你的任务名称，按回车键确认">
  </div>
  <div>
<taskRow 
  v-for="(todo, index) in todos" 
  :key="index" 
  :todo="todo" 
  :index="index"
  @delete="deleteTodo(index)"
  @edit="editTodo(index, $event)"
  @update:completed="todo.completed = $event" 
></taskRow>

  </div>
  <div class="footer">
    <input type="checkbox" v-model="allChecked" @change="toggleAll">
    <div>已完成 {{ completedCount }} / 全部 {{ todos.length }}</div>
    <button @click="clearCompleted" style="background-color: red;border-radius: 4px;margin-right: 5px;">清除已完成任务</button>
  </div>
  </div>
</div>

</template>

<script>
import TaskRow from './components/TodoList.vue';
import pubsub from 'pubsub-js';

export default {
  name: 'App',
  components: {
    TaskRow,
  },
  data() {
    return {
      addTask: '',
      todos: JSON.parse(localStorage.getItem('todos')) || [],
      allChecked: false,
    };
  },
  computed: {
    completedCount() {
      return this.todos.filter(todo => todo.completed).length;
    },
  },
  methods: {
    addTodo() {
      if (this.addTask.trim() === '') {
        return this.todos.push({id: Date.now(), text: this})
      }
      const newTodo = { text: this.addTask, completed: false };
      this.todos.push(newTodo);
      localStorage.setItem('todos', JSON.stringify(this.todos));
      this.addTask = '';
    },
    deleteTodo(index) {
      this.todos.splice(index, 1);
      localStorage.setItem('todos', JSON.stringify(this.todos));
    },
    editTodo(index, value) {
      this.todos[index].text = value;
      localStorage.setItem('todos', JSON.stringify(this.todos));
    },
   clearCompleted() {
  this.todos = this.todos.filter(todo => !todo.completed);
  // 添加深拷贝确保响应式更新
  localStorage.setItem('todos', JSON.stringify([...this.todos])); 
},

    toggleAll() {
      this.todos.forEach(todo => {
        todo.completed = this.allChecked;
      });
      localStorage.setItem('todos', JSON.stringify(this.todos));
    },
  },
  watch: {
    todos: {
      handler(newVal) {
        this.allChecked = newVal.every(todo => todo.completed);
      },
      deep: true,
    },
  },
  created() {
    pubsub.subscribe('addTodo', (msg, data) => {
      this.addTodo(data);
    });
    pubsub.subscribe('deleteTodo', (msg, data) => {
      this.deleteTodo(data);
    });
    pubsub.subscribe('editTodo', (msg, data) => {
      this.editTodo(data.index, data.value);
    });
  },
  beforeDestroy() {
    pubsub.unsubscribe('addTodo');
    pubsub.unsubscribe('deleteTodo');
    pubsub.unsubscribe('editTodo');
  },
};
</script>

<style scoped>

.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh; /* 让容器高度占满整个视口高度 */
  border: 1px solid #000000;

}
.header {
  border: 1px solid #000000;
  width: 420px;
  padding-left: 10px;
  padding-right: 10px;
}

.scanf {
  width: 400px;
  height: 40px;
  margin-bottom: 5px;
  margin-top: 3px;
}
.taskline {
  width: 400px;
  height: 40px;
  border: 1px solid #000000;
  margin-bottom: 2px;
}

.footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 400px; /* 使底部与输入框宽度一致 */
  height: 40px;
  border: 1px solid #000000;
  margin-bottom: 3px;

}

/* 其他样式保持不变 */
</style>


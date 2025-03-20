<template>
  <div class="container">
    <div class="header">
      <div>
        <input type="text" v-model="addTask" @keyup.enter="addTodo" placeholder="请输入任务...">
      </div>
      <div>
        <task-row v-for="(todo, index) in todos" :key="todo.id" :todo="todo" :index="index" @delete-task="deleteTask" @edit-task="editTask" @update-completed="updateCompleted" />
      </div>
      <div class="footer">
        <input type="checkbox" :checked="allChecked" @change="toggleAll">
        <div>已完成 {{ completedCount }} / 全部 {{ todos.length }}</div>
        <button @click="clearCompleted">清除已完成</button>
      </div>
    </div>
  </div>
</template>

<script>
import pubsub from 'pubsub-js'
import TaskRow from './components/TodoList.vue'

export default {
  components: {
    TaskRow
  },
  data() {
    return {
      addTask: '',
      todos: JSON.parse(localStorage.getItem('todos')) || []
    }
  },
  computed: {
    allChecked() {
      return this.todos.every(todo => todo.completed);
    },
    completedCount() {
      return this.todos.filter(todo => todo.completed).length;
    }
  },
  methods: {
    addTodo() {
      if (this.addTask.trim() === '') return;
      const newTodo = {
        id: Date.now(),
        text: this.addTask,
        completed: false
      };
      this.todos.push(newTodo);
      localStorage.setItem('todos', JSON.stringify(this.todos));
      this.addTask = '';
    },
    editTask(index, task) {
      this.todos[index].text = task;
      localStorage.setItem('todos', JSON.stringify(this.todos));
    },
    deleteTask(index) {
      this.todos.splice(index, 1);
      localStorage.setItem('todos', JSON.stringify(this.todos));
    },
    toggleAll(event) {
      const checked = event.target.checked;
      this.todos.forEach(todo => {
        todo.completed = checked;
      });
      localStorage.setItem('todos', JSON.stringify(this.todos));
    },
    clearCompleted() {
      this.todos = this.todos.filter(todo => !todo.completed);
      localStorage.setItem('todos', JSON.stringify(this.todos));
    }
  }
}
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh; /* 让容器高度占满整个视口高度 */
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

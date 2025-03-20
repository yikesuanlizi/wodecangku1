<template>
  <div class="task-row">
    <div>
      <input v-if="isEditing" type="text" v-model="editText" @blur="finishEdit" @keydown.enter="finishEdit" ref="editInput">
      <template v-else>
        <input type="checkbox" v-model="todo.completed" @change="updateCompleted">
        <span :class="{ completed: todo.completed }" @click="startEdit">
          {{ todo.text }}
        </span>
      </template>
    </div>
    <div>
      <button v-if="!isEditing" @click="editTodo" class="btn1">编辑</button>
      <button @click="confirmDelete" class="btn2">删除</button>
    </div>
  </div>
</template>

<script>
import pubsub from 'pubsub-js'

export default {
  name: 'TaskRow',
  props: {
    todo: Object,
    index: Number
  },
  data() {
    return {
      isEditing: false,
      editText: this.todo.text
    }
  },
  mounted() {
    // 订阅 addTask 事件
    pubsub.subscribe('addTask', this.handleAddTask);
    pubsub.subscribe('editTask', this.handleEditTask);
    pubsub.subscribe('deleteTask', this.handleDeleteTask);
  },
  methods: {
    handleAddTask(msg, addT) {
      // 这里通常不需要在这个子组件里处理添加任务的逻辑
      // 除非你有特别的需求希望子组件能够响应这个事件
    },
    handleEditTask(msg, payload) {
      if (payload.index === this.index) {
        this.editText = payload.task;
        this.finishEdit();
      }
    },
    handleDeleteTask(msg, index) {
      if (index === this.index) {
        this.deleteTask();
      }
    },
    startEdit() {
      this.isEditing = true;
      this.$nextTick(() => {
        this.$refs.editInput.focus();
      });
    },
    finishEdit() {
      this.isEditing = false;
      this.todo.text = this.editText;
      // 直接在这里更新本地存储
      localStorage.setItem('todos', JSON.stringify(this.$parent.todos));
    },
    editTodo() {
      this.startEdit();
    },
    updateCompleted() {
      // 直接在这里更新本地存储
      localStorage.setItem('todos', JSON.stringify(this.$parent.todos));
    },
    confirmDelete() {
      if (confirm('确定要删除此任务吗？')) {
        this.deleteTask();
      }
    },
    deleteTask() {
      // 直接在这里删除任务并更新本地存储
      this.$parent.todos.splice(this.index, 1);
      localStorage.setItem('todos', JSON.stringify(this.$parent.todos));
    }
  },
  beforeDestroy() {
    // 组件销毁前取消订阅
    pubsub.unsubscribe('addTask', this.handleAddTask);
    pubsub.unsubscribe('editTask', this.handleEditTask);
    pubsub.unsubscribe('deleteTask', this.handleDeleteTask);
  }
}
</script>

<style scoped>
.task-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 400px;
  border-bottom: 1px solid #ccc;
}
input[type="checkbox"] {
  accent-color: #00ffff;
}
.btn1 {
  background-color: #00ffff;
  border-radius: 4px;
}
.btn2 {
  background-color: red;
  border-radius: 4px;
}
</style>

<template>
  <div class="task-row">
    <div>
      <input 
        v-if="isEditing" 
        type="text" 
        v-model="todo.text" 
        @blur="finishEdit"
        @keydown.enter="finishEdit"
        ref="editInput"
      >
      <template v-else>
        <input 
          type="checkbox" 
          v-model="todo.completed" 
          @change="updateCompleted"
        >
        <span 
          :class="{ completed: todo.completed }" 
          @click="startEdit"
        >
          {{ todo.text }}
        </span>
      </template>
    </div>
    <div>
      <!-- 移除完成按钮 -->
      <button v-if="!isEditing" @click="editTodo" class="btn1">编辑</button>
      <button @click="deleteTodo" class="btn2">删除</button>
    </div>
  </div>
</template>

<script>
import pubsub from 'pubsub-js';

export default {
  name: 'TaskRow',
  props: {
    todo: Object,
    index: Number,
  },
  data() {
    return {
      isEditing: false,
    };
  },
  methods: {
    updateCompleted() {
      this.$emit('update:completed', this.todo.completed);// 触发父组件的更新事件
      pubsub.publish('editTodo', { index: this.index, value: this.todo });// 触发全局的更新事件
},
    deleteTodo() {
      const confirmDelete = confirm('确认删除吗？');
      if (confirmDelete) {
        pubsub.publish('deleteTodo', this.index);
      }
    },
    editTodo() {
      this.isEditing = true;
      this.$nextTick(() => {
        this.$refs.editInput.focus();
      });
    },
    startEdit() {
      this.isEditing = true;
      this.$nextTick(() => {
        this.$refs.editInput.focus();
      });
    },
    finishEdit() {
      if (this.todo.text.trim() === '') {
        alert('任务名称不能为空');
        return;
      }
      this.isEditing = false;
      pubsub.publish('editTodo', { index: this.index, value: this.todo });
    },
  },
};
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

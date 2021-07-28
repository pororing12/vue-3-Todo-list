<template>
  <div class="container">
    <h2>To-Do List</h2>
    <input
      class="form-control"
      type="text"
      v-model="searchText"
      placeholder="Search"
    >
    <hr>
    <TodoSimpleForm @add-todo="addTodo"/>
    <div style="color: red">{{ error }}</div>

    <div v-if="!filteredTodos.length">
      There is noting to display
    </div>

    <TodoList 
      :todos="filteredTodos" 
      @toggle-todo="ToggleTodo"
      @delete-todo="deleteTodo" />
  </div>
</template>

<script>
import { ref, computed } from "vue";
import TodoSimpleForm from '@/components/TodoSimpleForm.vue';
import TodoList from "@/components/TodoList.vue";
import axios from "axios"
export default {
  name: "App",
  components: {
    TodoSimpleForm,
    TodoList
  },
  setup() {
    const todos = ref([]);
    const error = ref("");

    const getTodos = async () => {
      try {
        const res = await axios.get("http://localhost:3000/todos");
        todos.value = res.data
      } catch(err) {
        console.log(err);
      }
    }

    getTodos();

    const addTodo = async (todo) => {
      //데이터베이스 Todo 저장
      try {
        error.value = "";
        const res = await axios.post("http://localhost:3000/todos", {
          subject: todo.subject,
          completed: todo.completed
        })
        todos.value.push(res.data);
      } catch(err) {
        console.log(err)
        error.value = "Something went wrong.";
      }
      // .then(res => {
      //   todos.value.push(res.data);
      // }).catch(err => {
      //   console.log(err);
      //   error.value = "something went wrong"
      // })
    };

    const ToggleTodo = async (index) => {
      const id = todos.value[index].id
      try {
        await axios.patch(`http://localhost:3000/todos/${id}`, {
          completed: !todos.value[index].completed
        })
        todos.value[index].completed = !todos.value[index].completed
      } catch(err) {
        console.log(err);
      }
    }

    const deleteTodo = async (index) => {
      const id = todos.value[index].id
      try {
        await axios.delete(`http://localhost:3000/todos/${id}`);
        todos.value.splice(index, 1);
      } catch(err) {
        console.log(err);
      }
    }

    const searchText = ref("");

    const filteredTodos = computed(() => {
      if (searchText.value) {
        return todos.value.filter( todo => {
          return todo.subject.includes(searchText.value)
        })
      } 
      return todos.value
    })

    return {
      todos,
      error,
      getTodos,
      addTodo,
      deleteTodo,
      ToggleTodo,
      searchText,
      filteredTodos
    };
  },
};
</script>

<style>
  .todo {
    color: gray;
    text-decoration: line-through;
  }
</style>

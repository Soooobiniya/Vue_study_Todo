<template>
  <div>
    <h2>To-Do List</h2>
    <input
      class="form-control"
      type="text"
      v-model="searchText"
      placeholder="Search"
      @keyup.enter="searchTodo"
    />
    <hr />
    <TodoSimpleForm @add-todo="addTodo"/>
    <div style="color: red">{{ errorMsg }}</div>

    <div v-if="!todos.length">
      There is nothing to display.
    </div>

    <TodoList 
      :todos="todos"
      @toggle-todo="toggleTodo"
      @delete-todo="deleteTodo"
    />
    <hr />
    <nav aria-label="Page navigation example">
      <ul class="pagination">
        <li
          v-if="currentPage !== 1" 
          class="page-item"
        >
          <a style="cursor: pointer" class="page-link" @click="getTodos(currentPage - 1)">Previous</a>
        </li>
        <li 
          v-for="num in numOfPages" 
          :key="num"
          class="page-item"
          :class="currentPage === num ? 'active' : ''"
        >
          <a style="cursor: pointer" class="page-link" @click="getTodos(num)">{{ num }}</a>
        </li>
        <li 
          v-if="currentPage !== numOfPages" 
          class="page-item"
        >
          <a style="cursor: pointer" class="page-link" @click="getTodos(currentPage + 1)">Next</a>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
import { ref, computed, watch } from "vue";
import TodoSimpleForm from '@/components/TodoSimpleForm.vue';
import TodoList from '@/components/TodoList.vue';
import axios from 'axios';

export default {
  components: {
    TodoSimpleForm,
    TodoList
  },
  setup() {
    const todos = ref([]);
    const errorMsg = ref('');
    const numOfTodos = ref(0);
    const pageLimit = 5;
    const currentPage = ref(1);
    const searchText = ref('');

    const numOfPages = computed(() => {
      return Math.ceil(numOfTodos.value/pageLimit);
    })
    
    const getTodos = async (page = currentPage.value) => {
      currentPage.value = page;
      try {
        const res = await axios.get(
          `http://localhost:3000/todos?_sort=id&_order=desc&subject_like=${searchText.value}&_page=${page}&_limit=${pageLimit}`
        );
        numOfTodos.value = res.headers['x-total-count'];
        todos.value = res.data;
      } catch (err) {
        console.log(err);
        errorMsg.value = 'Something went wrong!';
      }
    };

    getTodos();
  
    const addTodo = async (todo) => {
      errorMsg.value = '';
      try {
        // db에 Todo를 저장
        await axios.post('http://localhost:3000/todos', {
          subject: todo.subject,
          completed: todo.completed
        });

        getTodos(1); // 한 페이지에 5개만 나오게 하는 것 유지(새로고침 느낌)
      } catch (err) {
        console.log(err);
        errorMsg.value = 'Something went wrong!';
      }
    };

    const toggleTodo = async (index, checked) => {
      errorMsg.value = '';
      const id = todos.value[index].id;
      try {
        await axios.patch('http://localhost:3000/todos/' + id, {
          completed: checked
        });
        todos.value[index].completed = checked;
      } catch(err) {
        console.log(err);
        errorMsg.value = 'Something went wrong!';
      }
    };

    const deleteTodo = async (index) => {
      errorMsg.value = '';
      const id = todos.value[index].id;
      try {
        await axios.delete('http://localhost:3000/todos/' + id)
        getTodos(1);
      } catch (err) {
          console.log(err);
          errorMsg.value = 'Something went wrong!';
      }
    };

    // search 기능에 watch, timeout 적용
    let timeout = null;

    const searchTodo = () => {
      clearTimeout(timeout);
      getTodos(1);
    };

    watch(searchText, () => {
      clearTimeout(timeout);
      timeout = setTimeout(() => {
        getTodos(1); // 항상 첫 페이지를 보여주게 인자 설정
      }, 2000)
    });

    // const filteredTodos = computed(() => {
    //   if (searchText.value) {
    //     return todos.value.filter(obj => {
    //       return obj.subject.includes(searchText.value);
    //     });
    //   }

    //   return todos.value;
    // });

    return {
      todos,
      addTodo,
      toggleTodo,
      deleteTodo,
      searchText,
      // filteredTodos,
      errorMsg,
      numOfPages,
      currentPage,
      getTodos,
      searchTodo
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

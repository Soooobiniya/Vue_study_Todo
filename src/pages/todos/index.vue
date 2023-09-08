<template>
  <div>
    <div>
      <div class="d-flex justify-content-between mb-3">
        <h2>To-Do List</h2>
        <button 
          class="btn btn-primary"
          @click="moveToCreatePage"
        >
          Create Todo
        </button>
      </div>

      <input
        class="form-control"
        type="text"
        v-model="searchText"
        placeholder="Search"
        @keyup.enter="searchTodo"
      />
      <hr />

      <div v-if="!todos.length">
        There is nothing to display.
      </div>

      <TodoList 
        :todos="todos"
        @toggle-todo="toggleTodo"
        @delete-todo="deleteTodo"
      />
      <hr />
      <Pagination
        v-if="todos.length"
        :numOfPages="numOfPages"
        :currentPage="currentPage"
        @click="getTodos"
      />
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from "vue";
import TodoList from '@/components/TodoList.vue';
import axios from '@/axios';
import { useToast } from '@/composables/toast';
import { useRouter } from 'vue-router';
import Pagination from '@/components/Pagination.vue';

export default {
  components: {
    TodoList,
    Pagination
  },
  setup() {
    const router = useRouter();
    const todos = ref([]);
    const errorMsg = ref('');
    const numOfTodos = ref(0);
    const pageLimit = 5;
    const currentPage = ref(1);
    const searchText = ref('');
    const numOfPages = computed(() => {
      return Math.ceil(numOfTodos.value/pageLimit);
    });

    const {
      toastMessage,
      toastAlertType,
      showToast,
      triggerToast
    } = useToast()

    const getTodos = async (page = currentPage.value) => {
      currentPage.value = page;
      try {
        const res = await axios.get(
          `todos?_sort=id&_order=desc&subject_like=${searchText.value}&_page=${page}&_limit=${pageLimit}`
        );
        numOfTodos.value = res.headers['x-total-count'];
        todos.value = res.data;
      } catch (err) {
        console.log(err);
        errorMsg.value = 'Something went wrong!';
        triggerToast('Something went wrong!', 'danger');
      }
    };

    getTodos();
  
    const addTodo = async (todo) => {
      errorMsg.value = '';
      try {
        // db에 Todo를 저장
        await axios.post('todos', {
          subject: todo.subject,
          completed: todo.completed
        });

        getTodos(1); // 한 페이지에 5개만 나오게 하는 것 유지(새로고침 느낌)
      } catch (err) {
        console.log(err);
        errorMsg.value = 'Something went wrong!';
        triggerToast('Something went wrong!', 'danger');
      }
    };

    const toggleTodo = async (index, checked) => {
      errorMsg.value = '';
      const id = todos.value[index].id;
      try {
        await axios.patch('todos/' + id, {
          completed: checked
        });
        todos.value[index].completed = checked;
      } catch(err) {
        console.log(err);
        errorMsg.value = 'Something went wrong!';
        triggerToast('Something went wrong!', 'danger');
      }
    };

    const deleteTodo = async (id) => {
      errorMsg.value = '';
      try {
        await axios.delete('todos/' + id)
        getTodos(1);
      } catch (err) {
          console.log(err);
          errorMsg.value = 'Something went wrong!';
          triggerToast('Something went wrong!', 'danger');
      }
    };

    const moveToCreatePage = () => {
      router.push({
        name: 'TodoCreate',
      })
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
      searchTodo,
      toastMessage,
      toastAlertType,
      showToast,
      moveToCreatePage
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

<template>
  <!-- <div
    v-for="(obj, index) in todos" 
    :key="obj.id" 
    class="card mt-2"
  > -->
  <List :items="todos">
    <template #default="{ item, index }">
    <div
      class="card-body p-2 d-flex aligh-items-center"
      style="cursor: pointer"
      @click="moveToPage(item.id)"
    >
      <div class="flex-grow-1">
        <input
          class="ml-2 mr-2"
          type="checkbox"
          :checked="item.completed"
          @change="toggleTodo(index, $event)"
          @click.stop
        />
        <span :class="{ todo: item.completed }">
          {{ item.subject }}
        </span>
      </div>
      <div>
        <button class="btn btn-danger btn-sm" @click.stop="openModal(item.id)">
          Delete
        </button>
      </div>
    </div>
    </template>
  </List>
  <!-- </div> -->
  <teleport to="#modal">
    <Modal v-if="showModal" @close="closeModal" @delete="deleteTodo">
      <template #title> Delete Todo </template>
      <template #body> Are you sure you want to delete the todo? </template>
      <template #footer>
        <button>Ok</button>
      </template>
    </Modal>
  </teleport>
</template>

<script>
import { useRouter } from "vue-router";
import Modal from "./DeleteModal.vue";
import { ref } from "vue";
import List from "./List.vue";

export default {
  components: {
    Modal,
    List,
  },

  props: {
    todos: {
      type: Array,
      required: true,
    },
  },

  emits: ["toggle-todo", "delete-todo"],

  setup(props, { emit }) {
    const router = useRouter();
    const showModal = ref(false);
    const todoDeleteId = ref(null);

    const toggleTodo = (index, event) => {
      emit("toggle-todo", index, event.target.checked);
    };

    const openModal = (id) => {
      todoDeleteId.value = id;
      showModal.value = true;
    };

    const closeModal = () => {
      todoDeleteId.value = null;
      showModal.value = false;
    };

    const deleteTodo = () => {
      emit("delete-todo", todoDeleteId.value);
      closeModal();
    };

    const moveToPage = (todoId) => {
      // router.push('./todos/' + todoId);
      router.push({
        name: "Todo",
        params: {
          id: todoId,
        },
      });
    };

    return {
      toggleTodo,
      deleteTodo,
      moveToPage,
      showModal,
      openModal,
      closeModal,
    };
  },
};
</script>

<style>
</style>
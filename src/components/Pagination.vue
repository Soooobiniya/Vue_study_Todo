<template>
  <nav aria-label="Page navigation example">
    <ul class="pagination">
      <li v-if="currentPage !== 1" class="page-item">
        <a
          style="cursor: pointer"
          class="page-link"
          @click="onClick(currentPage - 1)"
          >Previous</a
        >
      </li>
      <li
        v-for="num in numOfPages"
        :key="num"
        class="page-item"
        :class="currentPage === num ? 'active' : ''"
      >
        <a style="cursor: pointer" class="page-link" @click="onClick(num)">{{
          num
        }}</a>
      </li>
      <li v-if="currentPage !== numOfPages" class="page-item">
        <a
          style="cursor: pointer"
          class="page-link"
          @click="onClick(currentPage + 1)"
          >Next</a
        >
      </li>
    </ul>
  </nav>
</template>

<script>
import { getCurrentInstance } from 'vue';
export default {
    props: {
        numOfPages: {
            type: Number,
            required: true
        },
        currentPage: {
            type: Number,
            required: true
        }
    },
    emits: ['click'],
    setup() {
        const { emit } = getCurrentInstance();

        const onClick = (page) => {
            emit('click', page);
        };
        return {
            onClick
        };
    }
};
</script>

<style>
</style>
<script>
import { onMounted } from 'vue';

export default {
  props: {
    pages: {
      type: Object,
      default: () => ({}),
    },
  },
  setup(props, { emit }) {
    onMounted(async () => {});

    const updatePages = (item) => {
      emit('update-pages', item);
    };

    return {
      updatePages,
    };
  },
};
</script>

<template>
  <nav aria-label="Page navigation example">
    <ul class="pagination justify-content-center">
      <li class="page-item" :class="{ disabled: pages.current_page === 1 }">
        <a
          class="page-link"
          href="#"
          aria-label="Previous"
          @click.prevent="updatePages(pages.current_page - 1)"
        >
          <span aria-hidden="true">&laquo;</span>
        </a>
      </li>
      <li
        v-for="(item, index) in pages.total_pages"
        :key="`page${index}`"
        class="page-item"
        :class="{ active: item === pages.current_page }"
      >
        <span class="page-link" v-if="item === pages.current_page">{{
          item
        }}</span>
        <a
          class="page-link"
          href="#"
          v-else
          @click.prevent="updatePages(item)"
          >{{ item }}</a
        >
      </li>
      <li
        class="page-item"
        :class="{ disabled: pages.current_page === pages.total_pages }"
      >
        <a
          class="page-link"
          href="#"
          aria-label="Next"
          @click.prevent="updatePages(pages.current_page + 1)"
        >
          <span aria-hidden="true">&raquo;</span>
        </a>
      </li>
    </ul>
  </nav>
</template>

<style lang="scss" scoped></style>

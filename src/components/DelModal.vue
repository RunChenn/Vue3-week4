<script>
import { ref, onMounted } from 'vue';
import api from '../api/index.js';

import { Modal } from 'bootstrap';

export default {
  props: {
    id: {
      type: String,
      default: '',
    },
    productTitle: {
      type: String,
      default: '',
    },
  },
  setup(props, { emit }) {
    console.log(props);
    console.log(props.productTitle);

    let delProductModal = ref(null);

    onMounted(async () => {
      delProductModal.value = new Modal(
        document.getElementById('delProductModal'),
        {
          keyboard: false,
        }
      );
    });

    // 刪除商品
    const delProduct = async () => {
      try {
        const res = await api.products.delProducts(props.id);

        alert(res.message);

        delProductModal.value.hide();

        emit('update');
      } catch (err) {
        alert(err.message);
      }
    };

    return {
      delProduct,
    };
  },
};
</script>

<template>
  <div
    id="delProductModal"
    ref="delProductModal"
    class="modal fade"
    tabindex="-1"
    aria-labelledby="delProductModalLabel"
    aria-hidden="true"
    data-bs-keyboard="false"
  >
    <div class="modal-dialog modal-dialog-centered">
      <div class="modal-content border-0">
        <div class="modal-header bg-primary text-white">
          <h5 id="delProductModalLabel" class="modal-title">
            <span>刪除產品</span>
          </h5>
          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="modal"
            aria-label="Close"
          ></button>
        </div>
        <div class="modal-body">
          是否刪除
          <strong class="text-danger">{{ productTitle }}</strong>
          商品
          <br />刪除後將無法恢復
        </div>
        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-outline-danger"
            data-bs-dismiss="modal"
          >
            取消
          </button>
          <button type="button" class="btn btn-success" @click="delProduct">
            確認刪除
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped></style>

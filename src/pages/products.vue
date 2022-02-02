<script>
import { ref, onMounted } from 'vue';
import api from '../api/index.js';

import ProdModal from '../components/ProdModal.vue';
import DelModal from '../components/DelModal.vue';
import Pagination from '../components/Pagination.vue';

export default {
  components: { ProdModal, DelModal, Pagination },
  setup() {
    let products = ref([]);

    let prodInfo = ref({});

    let isEnabled = ref(false);

    let isNew = ref(false);

    let tempProduct = ref({ imagesUrl: [], id: '' });

    let pagination = ref({});

    const prodsDetail = (item) => {
      prodInfo.value = item;
    };

    // 載入所有商品
    const getData = async (page = 1) => {
      try {
        const prodsData = await api.products.getProducts(page);

        products.value = prodsData.products;
        pagination.value = prodsData.pagination;
      } catch (err) {
        alert(err.message);
      }
    };

    onMounted(async () => {
      try {
        // 檢查權限
        await api.auth.checkAuth();
        getData();
      } catch (err) {
        alert(err.message);
      }
    });

    const openModal = (status, item) => {
      isNew.value = status === 'new' ? true : false;

      tempProduct.value =
        status === 'new'
          ? {
              imagesUrl: [],
            }
          : { ...item };
    };

    // 新增圖片
    const createImages = () => {
      tempProduct.value.imagesUrl = [];
      tempProduct.value.imagesUrl.push('');
    };

    return {
      products,
      isEnabled,
      prodsDetail,
      prodInfo,
      tempProduct,
      isNew,
      openModal,
      createImages,
      getData,
      pagination,
    };
  },
};
</script>

<template>
  <div>
    <div class="container">
      <div class="row py-1">
        <div class="text-end mb-3">
          <button
            type="button"
            class="btn btn-success"
            @click="openModal('new')"
            data-bs-toggle="modal"
            data-bs-target="#productModal"
          >
            建立新的產品
          </button>
        </div>
        <div class="col-12 col-sm-12">
          <h3>產品列表</h3>
          <hr />
          <table class="table table-hover">
            <thead>
              <tr>
                <th scope="col">分類</th>
                <th scope="col" class="text-start">產品名稱</th>
                <th scope="col">原價</th>
                <th scope="col">售價</th>
                <th scope="col">是否啟用</th>
                <th scope="col">操作</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in products" :key="item.id">
                <td>{{ item.category }}</td>
                <td class="text-start">{{ item.title }}</td>
                <td>{{ item.origin_price }}</td>
                <td>{{ item.price }}</td>
                <td>
                  <span v-if="item.is_enabled" class="text-success">啟用</span>
                  <span v-else>未啟用</span>
                </td>
                <td>
                  <button
                    type="button"
                    class="btn btn-outline-primary btn-sm me-2 mb-md-1"
                    data-bs-target="#productModal"
                    data-bs-toggle="modal"
                    @click="openModal('edit', item)"
                  >
                    編輯
                  </button>
                  <button
                    type="button"
                    class="btn btn-outline-danger btn-sm me-2 mb-md-1"
                    data-bs-target="#delProductModal"
                    data-bs-toggle="modal"
                    @click="openModal('delete', item)"
                  >
                    刪除
                  </button>
                </td>
              </tr>
            </tbody>
          </table>

          <Pagination v-model:pages="pagination" @update-pages="getData" />
        </div>
      </div>
    </div>
    <!-- Modal -->
    <ProdModal
      v-model:temp-product="tempProduct"
      v-model:is-new="isNew"
      @update="getData"
    />
    <!-- delModal -->
    <DelModal
      v-model:id="tempProduct.id"
      v-model:product-title="tempProduct.title"
      @update="getData"
    />
  </div>
</template>

<style lang="scss" scoped>
table {
  vertical-align: middle;
}
img {
  max-width: 100%;

  object-fit: contain;
}

.primary-image {
  height: 300px;
}

.form-check-label span.checked {
  color: #198754;
}

.images {
  height: 150px;
}
.form-check-input:checked {
  border-color: #198754;
  background-color: #198754;
}
</style>

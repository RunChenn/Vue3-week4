<script>
import { ref, onMounted } from 'vue';
import api from '../api/index.js';

import ProdModal from '../components/ProdModal.vue';

import { Modal } from 'bootstrap';

export default {
  components: { ProdModal },
  setup() {
    let productModal = ref(null);
    let delProductModal = ref(null);
    let detailProdsModal = ref(null);

    let products = ref([]);

    let prodInfo = ref({});

    let isEnabled = ref(false);

    let isNew = ref(false);

    let tempProduct = ref({ imagesUrl: [], id: '' });

    const prodsDetail = (item) => {
      prodInfo.value = item;
      detailProdsModal.value.show();
    };

    // 載入所有商品
    const getData = async () => {
      try {
        const prodsData = await api.products.getProducts();
        products.value = prodsData.products;
      } catch (err) {
        alert(err.message);
      }
    };

    onMounted(async () => {
      // productModal.value = new Modal(document.getElementById('productModal'), {
      //   keyboard: false,
      // });

      delProductModal.value = new Modal(
        document.getElementById('delProductModal'),
        {
          keyboard: false,
        }
      );

      detailProdsModal.value = new Modal(
        document.getElementById('detailProdsModal'),
        {
          keyboard: false,
        }
      );

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

      if (status === 'delete') {
        delProductModal.value.show();
      }
    };

    // 刪除商品
    const delProduct = async () => {
      try {
        const res = await api.products.delProducts(tempProduct.value.id);

        alert(res.message);

        getData();

        delProductModal.value.hide();
      } catch (err) {
        alert(err.message);
      }
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
      delProduct,
      createImages,
      getData,
    };
  },
};
</script>

<template>
  <div>
    <div class="container">
      <div class="row py-1">
        <div class="text-end mb-4">
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
                <th scope="col">產品名稱</th>
                <th scope="col">原價</th>
                <th scope="col">售價</th>
                <th scope="col">是否啟用</th>
                <th scope="col">操作</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in products" :key="item.id">
                <th scope="row">{{ item.title }}</th>
                <td>{{ item.origin_price }}</td>
                <td>{{ item.price }}</td>
                <td>
                  <span v-if="item.is_enabled" class="text-success">啟用</span>
                  <span v-else>未啟用</span>
                  <!-- <label
                    class="form-check-label"
                    :for="`switchCheckBox${index}`"
                  >
                    <span v-if="item.is_enabled" class="checked">啟用</span>
                    <span v-else>未啟用</span>
                  </label> -->
                  <!-- <div class="form-check form-switch">
                    <input
                      class="form-check-input form-check-input-checked-bg-color-success"
                      type="checkbox"
                      role="switch"
                      :id="`switchCheckBox${index}`"
                      v-model="products[index].is_enabled"
                      :true-value="1"
                      :false-value="0"
                    />
                  </div> -->
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
                    @click="openModal('delete', item)"
                  >
                    刪除
                  </button>
                  <button
                    type="button"
                    class="btn btn-outline-success btn-sm me-2 mb-md-1"
                    @click="prodsDetail(item)"
                  >
                    查看細節
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
          <p>
            目前有
            <span
              ><strong>{{ Object.keys(products).length }}</strong></span
            >
            項產品
          </p>
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
    <div
      id="delProductModal"
      ref="delProductModal"
      class="modal fade"
      tabindex="-1"
      aria-labelledby="delProductModalLabel"
      aria-hidden="true"
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
            <strong class="text-danger">{{ tempProduct.title }}</strong>
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

    <!-- detailProdsModal -->
    <div
      id="detailProdsModal"
      ref="detailProdsModal"
      class="modal fade text-start"
      tabindex="-1"
      aria-labelledby="detailProdsModalLabel"
      aria-hidden="true"
      data-bs-keyboard="false"
    >
      <div class="modal-dialog modal-dialog-scrollable modal-xl">
        <div class="modal-content border-0">
          <div class="modal-header bg-primary text-white">
            <h5 id="detailProdsModalLabel" class="modal-title">
              {{ prodInfo.title }}
            </h5>
            <button
              type="button"
              class="btn-close text-white"
              data-bs-dismiss="modal"
              aria-label="Close"
            ></button>
          </div>
          <div class="modal-body">
            <!-- <h4>{{ prodInfo.title }}</h4> -->
            <div v-if="prodInfo.title">
              <div class="card mb-3">
                <img
                  :src="prodInfo.imageUrl"
                  class="card-img-top primary-image"
                  alt="主圖"
                />
                <div class="card-body">
                  <h5 class="card-title">
                    {{ prodInfo.title }}
                    <span class="badge bg-primary ms-2">{{
                      prodInfo.category
                    }}</span>
                  </h5>
                  <p class="card-text">商品描述：{{ prodInfo.description }}</p>
                  <p class="card-text">商品內容：{{ prodInfo.content }}</p>
                  <div class="d-flex">
                    <p class="card-text me-2 text-danger">
                      ${{ prodInfo.price }}
                    </p>
                    <p class="card-text text-black-50">
                      <del>{{ prodInfo.origin_price }}</del>
                    </p>
                    {{ prodInfo.unit }} / 元
                  </div>
                </div>
              </div>
              <span v-for="image in prodInfo.imagesUrl" :key="image">
                <img
                  v-if="image"
                  :src="image"
                  :alt="prodInfo.title"
                  class="images m-2"
                />
              </span>
            </div>
          </div>
          <div class="modal-footer">
            <button
              type="button"
              class="btn btn-success mx-auto"
              data-bs-dismiss="modal"
            >
              關閉
            </button>
          </div>
        </div>
      </div>
    </div>
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

<template>
  <view class="index">
    <nut-toast
      :msg="toast.msg"
      v-model:visible="toast.show"
      :type="toast.type"
      :cover="toast.cover"
    />
    <nut-cell>
      <view-block class="infiniteUl">
        <nut-infiniteloading
          pull-icon="JD"
          load-txt="loading"
          load-more-txt="没有啦～"
          container-id="scrollDemo"
          :is-open-refresh="infiniteloading.isOpenRefresh"
          :has-more="infiniteloading.hasMore"
          @load-more="loadMore"
        >
          <view-block
            class="infiniteLi"
            v-for="(item, index) in pictures"
            :key="item.id"
          >
            <img
              :src="`https://picsum.photos/id/${item.id}/200`"
              @click="toDetail(index)"
            />
          </view-block>
        </nut-infiniteloading>
      </view-block>
    </nut-cell>
  </view>
</template>

<script>
import { toRefs, reactive, onMounted, computed } from "vue";
import { useStore } from "vuex";
import Taro from "@tarojs/taro";``
export default {
  name: "Index",
  components: {},
  setup() {
    const store = useStore();
    const state = reactive({
      pictures: computed(() => store.state.pictures),
      page: {
        num: 1,
        limit: 5,
      },
      toast: {
        msg: "toast",
        type: "text",
        show: false,
        cover: false,
      },
      infiniteloading: {
        isOpenRefresh: false,
        hasMore: true,
      },
    });

    const methods = {
      openToast: (type, msg, cover = false) => {
        state.toast.show = true;
        state.toast.msg = msg;
        state.toast.type = type;
        state.toast.cover = cover;
      },

      loadPage: async (page, limit) => {
        methods.openToast("loading", "加载中", true);
        try {
          const resp = await store.dispatch("LOAD_PICTURE_MUTATIONS", {
            page: page,
            limit: limit,
          });
          store.commit("LOAD_PICTURE_MUTATIONS", resp);
        } catch (error) {
          methods.openToast("fail", "加载失败");
          console.log(error.message())
        } finally {
          state.toast.show = false;
        }
      },

      loadMore: (done) => {
        methods.loadPage(++state.page.num, state.page.limit);
        done();
      },

      toDetail: (index) => {
        Taro.navigateTo({
          url: `/pages/detail/detail?idx=${index}`,
        });
      },

      init: () =>{
        methods.loadPage(state.page.num, state.page.limit);
      }
    };

    onMounted(() => {
      methods.init();
    });
    return {
      ...toRefs(state),
      ...methods,
    };
  },
};
</script>

<style lang="scss">
.index {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
}

.infiniteUl {
  height: 500px;
  width: 100%;
  overflow-y: auto;
  overflow-x: hidden;
}

.infiniteLi {
  margin-top: 10px;
  font-size: 14px;
  color: rgba(100, 100, 100, 1);
  text-align: center;
}
</style>

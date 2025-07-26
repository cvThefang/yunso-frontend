<template>
  <div class="index-page">
    <a-input-search
      v-model:value="searchParams.text"
      placeholder="input search text"
      enter-button="Search"
      size="large"
      @search="onSearch"
    />
    <MyDivider />
    <a-tabs v-model:activeKey="activeKey" @change="onTabChange">
      <a-tab-pane key="post" tab="文章">
        <PostList :post-list="postList" />
      </a-tab-pane>
      <a-tab-pane key="picture" tab="图片">
        <PictureList :picture-list="pictureList" />
      </a-tab-pane>
      <a-tab-pane key="user" tab="用户">
        <UserList :user-list="userList" />
      </a-tab-pane>
    </a-tabs>
  </div>
</template>

<script setup lang="ts">
import { ref, watchEffect } from "vue";
import PostList from "@/components/PostList.vue";
import PictureList from "@/components/PictureList.vue";
import UserList from "@/components/UserList.vue";
import MyDivider from "@/components/MyDivider.vue";
import { useRoute, useRouter } from "vue-router";
import myAxios from "@/plugins/myAxios";

const router = useRouter();
const route = useRoute();

const userList = ref([]);

const postList = ref([]);

const pictureList = ref([]);

const initSearchParams = {
  text: "",
  pageSizes: 10,
  pageNum: 1,
};

/**
 * 加载数据
 * 这个方法是旧版，效率不高 线面优化了一个聚合搜索接口
 * @param params
 */
const loadDataOld = (params: any) => {
  /**
   * 参数调整
   * 获取文章列表
   */
  const postQuery = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("post/list/page/vo", postQuery).then((res: any) => {
    postList.value = res.records;
  });

  /**
   * 参数调整
   * 获取图片列表
   */
  const pictureQuery = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("picture/list/page/vo", pictureQuery).then((res: any) => {
    pictureList.value = res.records;
  });

  /**
   * 参数调整
   * 获取用户列表
   */
  const userQuery = {
    ...params,
    userName: params.text,
  };
  myAxios.post("user/list/page/vo", userQuery).then((res: any) => {
    userList.value = res.records;
  });
};

const loadData = (params: any) => {
  /**
   * 参数转换
   */
  const query = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("search/all", query).then((res: any) => {
    userList.value = res.userList;
    postList.value = res.postList;
    pictureList.value = res.pictureList;
  });
};

const searchParams = ref(initSearchParams);

//首次加载页面是执行一次
loadData(initSearchParams);

// 监听路由变化，更新 searchParams
watchEffect(() => {
  searchParams.value = {
    ...initSearchParams,
    text: route.query.text,
  } as any;
});

const onSearch = (value: string) => {
  console.log(value);
  router.push({
    query: searchParams.value,
  });
  loadData(searchParams.value);
};

const activeKey = route.params.category;

const onTabChange = (key: string) => {
  router.push({
    path: `/${key}`,
    query: searchParams.value,
  });
};
</script>

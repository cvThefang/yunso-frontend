<template>
  <div class="index-page">
    <a-input-search
      v-model:value="searchText"
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
import { message } from "ant-design-vue";

const router = useRouter();
const route = useRoute();
const activeKey = route.params.category;

const userList = ref([]);

const postList = ref([]);

const pictureList = ref([]);

const initSearchParams = {
  text: "",
  type: activeKey,
  pageSizes: 10,
  pageNum: 1,
};

const searchText = ref(route.query.text);

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

/**
 * 加载聚合数据
 * @param params
 */
const loadAllData = (params: any) => {
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

/**
 * 加载单类数据
 * @param params
 */
const loadData = (params: any) => {
  const { type } = params;
  if (!type) {
    message.error("类别为空");
    return;
  }
  const query = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("search/all", query).then((res: any) => {
    if (type === "post") {
      postList.value = res.dataList;
    } else if (type === "picture") {
      pictureList.value = res.dataList;
    } else if (type === "user") {
      userList.value = res.dataList;
    }
  });
};

const searchParams = ref(initSearchParams);

//首次加载页面是执行一次 首次进入调用两次请求 优化
// loadData(initSearchParams);

// 监听路由变化，更新 searchParams
watchEffect(() => {
  searchParams.value = {
    ...initSearchParams,
    text: route.query.text,
    type: route.params.category,
  } as any;
  loadData(searchParams.value);
});

//
/**
 * onSearch 是表单提交事件 点击搜索按钮时调用
 * 触发路由跳转 上面的 watchEffect 监听路由变化 触发 loadData 方法 从而达到刷新数据的效果
 * @param value
 */
const onSearch = (value: string) => {
  console.log(value);
  router.push({
    query: {
      ...searchParams.value,
      text: value,
    },
  });
};

const onTabChange = (key: string) => {
  router.push({
    path: `/${key}`,
    query: searchParams.value,
  });
};
</script>

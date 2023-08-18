<!--<template>-->
<!--  <div class="index-page">-->
<!--    <a-input-search-->
<!--      v-model:value="searchParams.text"-->
<!--      placeholder="input search text"-->
<!--      enter-button="Search"-->
<!--      size="large"-->
<!--      @search="onSearch"-->
<!--    />-->
<!--    &lt;!&ndash;      {{ JSON.stringify(searchParams) }}}&ndash;&gt;-->
<!--    &lt;!&ndash;    {{ JSON.stringify(postList) }}}&ndash;&gt;-->
<!--    <MyDivider />-->
<!--    <a-tabs v-model:activeKey="activeKey" @change="onTabchange">-->
<!--      <a-tab-pane key="post" tab="文章">-->
<!--        <PostList :post-list="postList" />-->
<!--      </a-tab-pane>-->
<!--      <a-tab-pane key="picture" tab="图片">-->
<!--        <PictureList :picture-list="pictureList" />-->
<!--      </a-tab-pane>-->
<!--      <a-tab-pane key="user" tab="用户">-->
<!--        <UserList :user-list="userList" />-->
<!--      </a-tab-pane>-->
<!--    </a-tabs>-->
<!--  </div>-->
<!--</template>-->

<!--<script setup lang="ts">-->
<!--// import postList from "@/pages/PostList.vue";-->
<!--const postList = ref([]);-->
<!--const userList = ref([]);-->
<!--const pictureList = ref([]);-->
<!--const route = useRoute();-->
<!--const router = useRouter();-->
<!--import { ref, watchEffect } from "vue";-->
<!--import PostList from "@/pages/PostList.vue";-->
<!--import UserList from "@/pages/UserList.vue";-->

<!--const activeKey = route.params.category; // 激活选择动态路由的参数-->
<!--import MyDivider from "@/components/MyDivider.vue";-->
<!--import { useRoute, useRouter } from "vue-router";-->
<!--import PictureList from "@/pages/PictureList.vue";-->
<!--import myAxios from "@/plugins/myAxios";-->

<!--/**-->
<!-- * 加载数据-->
<!-- * @param params-->
<!-- */-->
<!--const initSearchParams = {-->
<!--  text: "",-->
<!--  pageSize: 10,-->
<!--  pageNum: 1,-->
<!--};-->
<!--// watchEffect(() => {-->
<!--//   searchParams.value = {-->
<!--//     ...initSearchParams,-->
<!--//     text: route.query.text,-->
<!--//   } as any;-->
<!--// });-->
<!--const onTabchange = (key: string) => {-->
<!--  router.push({-->
<!--    path: `/${key}`,-->
<!--    query: searchParams.value,-->
<!--  });-->
<!--};-->
<!--/**-->
<!-- * 加载数据-->
<!-- * @param params-->
<!-- */-->
<!--const loadData = (params: any) => {-->
<!--  const postQuery = {-->
<!--    ...params,-->
<!--    searchText: params.text,-->
<!--  };-->
<!--  myAxios.post("/post/list/page/vo", postQuery).then((res: any) => {-->
<!--    postList.value = res.records;-->
<!--  });-->
<!--  const pictureQuery = {-->
<!--    ...params,-->
<!--    searchText: params.text,-->
<!--  };-->
<!--  myAxios.post("/picture/list/page/vo", pictureQuery).then((res: any) => {-->
<!--    pictureList.value = res.records;-->
<!--  });-->
<!--  const userQuery = {-->
<!--    ...params,-->
<!--    userName: params.text,-->
<!--  };-->
<!--  myAxios.post("/user/list/page/vo", userQuery).then((res: any) => {-->
<!--    userList.value = res.records;-->
<!--  });-->
<!--};-->
<!--const searchParams = ref(initSearchParams);-->
<!--// 首次请求-->
<!--loadData(initSearchParams);-->

<!--const onSearch = (value: string) => {-->
<!--  console.log(value);-->
<!--  router.push({-->
<!--    query: searchParams.value,-->
<!--  });-->
<!--  // 根据条件查询-->
<!--  loadData(searchParams.value);-->
<!--};-->
<!--</script>-->
<template>
  <div class="index-page">
    <a-input-search
      v-model:value="searchText"
      placeholder="快来搜索吧"
      enter-button="搜索"
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

const postList = ref([]);

const userList = ref([]);

const pictureList = ref([]);

const route = useRoute();
const router = useRouter();
const activeKey = route.params.category;

const initSearchParams = {
  type: activeKey,
  text: "",
  pageSize: 10,
  pageNum: 1,
};

const searchText = ref(route.query.text || "");

/**
 * 加载数据
 * @param params
 */
const loadDataOld = (params: any) => {
  const postQuery = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("post/list/page/vo", postQuery).then((res: any) => {
    postList.value = res.records;
  });

  const userQuery = {
    ...params,
    userName: params.text,
  };
  myAxios.post("user/list/page/vo", userQuery).then((res: any) => {
    userList.value = res.records;
  });

  const pictureQuery = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("picture/list/page/vo", pictureQuery).then((res: any) => {
    pictureList.value = res.records;
  });
};

/**
 * 加载聚合数据
 * @param params
 */
const loadAllData = (params: any) => {
  const query = {
    ...params,
    searchText: params.text,
  };
  myAxios.post("search/all", query).then((res: any) => {
    postList.value = res.postList;
    userList.value = res.userList;
    pictureList.value = res.pictureList;
  });
};

/**
 * 加载单类数据
 * @param params
 */
const loadData = (params: any) => {
  const { type = "post" } = params;
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
    } else if (type === "user") {
      userList.value = res.dataList;
    } else if (type === "picture") {
      pictureList.value = res.dataList;
    }
  });
};

const searchParams = ref(initSearchParams);

watchEffect(() => {
  searchParams.value = {
    ...initSearchParams,
    text: route.query.text,
    type: route.params.category,
  } as any;
  loadData(searchParams.value);
});

const onSearch = (value: string) => {
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

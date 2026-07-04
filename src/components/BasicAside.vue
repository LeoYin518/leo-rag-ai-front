<template>
  <div id="basic-aside">
    <el-menu
      :default-active="defaultPath"
      class="aside-menu"
      :collapse="isCollapse"
      background-color="transparent"
      text-color="#D7E1EE"
      active-text-color="#FFFFFF"
    >
      <div class="menu-header">
<!--        <el-icon size="50">-->
<!--          <el-image :src="Logo"></el-image>-->
<!--        </el-icon>-->
        <h1>LEO-RAG-AI</h1>
        <el-text style="color: #bfcbd9" size="small">知识库AI问答系统</el-text>
      </div>
      <el-divider />

      <!-- <div class="aside-header">
      <el-icon :size="25" style="cursor: pointer" @click="openMenu">
        <Expand v-if="isCollapse" />
        <Fold v-else />
      </el-icon>
    </div> -->

      <el-menu-item
        v-for="item in menuRouterList"
        :index="item.path"
        @click="handleSelect(item)"
      >
        <el-icon>
          <component :is="item.meta?.icon"></component>
        </el-icon>
        <template #title>{{ item.meta?.description }}</template>
      </el-menu-item>

    </el-menu>
  </div>
</template>

<script setup lang="ts">
import routes from "@/router/config.ts";
import router from "@/router"; 

const emit = defineEmits(["changeAside"]);
const isCollapse = ref(false);
const path = router.currentRoute.value.fullPath;
const defaultPath = ref(path === "/" ? "/chat" : path);

// 使用计算属性过滤不是菜单项的路由选项
const menuRouterList = computed(() => {
  return routes.filter((item) => {
    return item.meta?.isMenu;
  });
});

router.afterEach((to) => { 
  defaultPath.value = to.path;
});

onMounted(() => {
  console.log(defaultPath.value);
});

const handleSelect = (e: any) => {
  console.log(e);
  router.push({
    path: e.path,
  });
};
</script>

<style scoped lang="less">
#basic-aside {
  height: 100%;
  background:
    radial-gradient(circle at 20% 0%, rgba(34, 211, 238, 0.24), transparent 15rem),
    radial-gradient(circle at 90% 18%, rgba(139, 92, 246, 0.18), transparent 12rem),
    linear-gradient(180deg, rgba(5, 10, 24, 0.98), rgba(8, 16, 32, 0.98));
  position: relative;

  &::before {
    position: absolute;
    inset: 0;
    pointer-events: none;
    content: "";
    background-image:
      linear-gradient(rgba(125, 249, 255, 0.045) 1px, transparent 1px),
      linear-gradient(90deg, rgba(125, 249, 255, 0.045) 1px, transparent 1px);
    background-size: 28px 28px;
    opacity: 0.62;
  }
}
:deep(.el-menu) {
  z-index: 10; // 配合 ChatView
}
.aside-header {
  display: flex;
  justify-content: right;
}
.menu-header {
  height: 112px;
  padding: 24px 18px 18px;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  align-items: flex-start;
  color: #eef6ff;
  position: relative;
  border-bottom: 0;
  box-shadow: inset 0 -1px 0 rgba(103, 232, 249, 0.045);

  &::before {
    width: 38px;
    height: 4px;
    margin-bottom: 14px;
    border-radius: 999px;
    content: "";
    background: linear-gradient(90deg, #22d3ee, #8b5cf6);
    box-shadow: 0 0 24px rgba(34, 211, 238, 0.62);
  }

  h1 {
    margin: 0 0 6px;
    color: #ffffff;
    font-size: 23px;
    font-weight: 800;
    line-height: 1;
    letter-spacing: 0;
    text-shadow: 0 0 26px rgba(34, 211, 238, 0.26);
  }

  :deep(.el-text) {
    color: rgba(215, 225, 238, 0.72) !important;
    line-height: 1.4;
  }
}
.aside-menu {
  height: 100%;
  padding: 0 10px 12px;
  border: 0;
  border-right: none;
  background: transparent;

  :deep(.el-divider) {
    display: none;
  }

  :deep(.el-menu-item) {
    height: 46px;
    margin: 7px 0;
    padding: 0 13px !important;
    border-radius: 8px;
    color: rgba(215, 225, 238, 0.86);
    font-weight: 650;
    letter-spacing: 0;
    border: 0;
    transition: background-color .18s ease, color .18s ease, transform .18s ease, border-color .18s ease, box-shadow .18s ease;

    .el-icon {
      width: 20px;
      margin-right: 10px;
      color: rgba(125, 249, 255, 0.86);
      font-size: 18px;
      filter: drop-shadow(0 0 10px rgba(34, 211, 238, 0.22));
    }
  }

  :deep(.el-menu-item:hover) {
    color: #ffffff;
    background: rgba(34, 211, 238, 0.08);
    box-shadow: inset 0 0 0 1px rgba(103, 232, 249, 0.055);
    transform: translateX(2px);
  }

  :deep(.el-menu-item.is-active) {
    color: #effcff;
    background:
      linear-gradient(135deg, rgba(34, 211, 238, 0.22), rgba(139, 92, 246, 0.20)),
      rgba(15, 23, 42, 0.46);
    box-shadow: inset 3px 0 0 #22d3ee, inset 0 0 0 1px rgba(125, 249, 255, 0.12), 0 0 28px rgba(34, 211, 238, 0.16);

    .el-icon {
      color: #a5f3fc;
    }
  }
}
</style>

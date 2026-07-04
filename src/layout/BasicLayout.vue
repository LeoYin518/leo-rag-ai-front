<template>
  <div id="basic-layout">
    <el-container>
      <el-aside :style="asideStyle">
        <BasicAside @change-aside="changeAside" />
      </el-aside>
      <el-main class="basic-main">
        <RouterView />
      </el-main>
    </el-container>
  </div>
</template>

<script setup lang="ts">
const asideStyle = ref({
  height: "calc(100vh - 24px)",
  width: "224px",
  transition: "width 0.22s ease",
});
const changeAside = (isCollapse: boolean) => {
  if (isCollapse) {
    asideStyle.value.width = "auto";
  } else {
    // 动态变到150px
    asideStyle.value.width = "224px";
  }
};
</script>

<style scoped lang="less">
#basic-layout {
  min-height: 100vh;
  padding: 12px;
  background:
    radial-gradient(circle at 12% 12%, rgba(34, 211, 238, 0.18), transparent 24rem),
    radial-gradient(circle at 88% 0%, rgba(139, 92, 246, 0.22), transparent 28rem),
    linear-gradient(135deg, rgba(3, 7, 18, 0.96), rgba(7, 11, 22, 0.94));

  :deep(.el-container) {
    height: calc(100vh - 24px);
    gap: 12px;
  }

  :deep(.el-aside) {
    flex-shrink: 0;
    overflow: hidden;
    border-radius: 8px;
    border: 0;
    box-shadow: 0 24px 84px rgba(0, 0, 0, 0.52), 0 0 42px rgba(34, 211, 238, 0.08), inset 0 0 0 1px rgba(103, 232, 249, 0.045);
  }
}

.basic-main {
  position: relative;
  min-width: 0;
  padding: 0;
  overflow: hidden;
  border: 0;
  border-radius: 8px;
  background:
    linear-gradient(180deg, rgba(8, 13, 28, 0.76), rgba(3, 7, 18, 0.72)),
    radial-gradient(circle at 100% 0%, rgba(34, 211, 238, 0.08), transparent 22rem);
  box-shadow: 0 24px 78px rgba(0, 0, 0, 0.42), inset 0 0 0 1px rgba(103, 232, 249, 0.045), inset 0 1px 0 rgba(125, 249, 255, 0.025);
  backdrop-filter: blur(18px);

  &::before {
    position: absolute;
    inset: 0;
    z-index: 0;
    pointer-events: none;
    content: "";
    background:
      linear-gradient(90deg, transparent, rgba(125, 249, 255, 0.08), transparent) 0 0 / 100% 1px no-repeat,
      radial-gradient(circle at 76% 18%, rgba(139, 92, 246, 0.10), transparent 20rem);
  }

  :deep(> *) {
    position: relative;
    z-index: 1;
    animation: page-in 0.28s ease both;
  }
}

@keyframes page-in {
  from {
    opacity: 0;
    transform: translateY(8px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>

<template>
  <div id="app">
    <router-view />
  </div>
</template>
<script lang="ts" setup>
import {
  TUIStore,
  StoreName,
} from "@tencentcloud/chat-uikit-engine";
import router from "./router/index";
TUIStore.watch(StoreName.USER, {
  kickedOut: (value: string) => {
    if (value && router.currentRoute.name !== 'login') {
      localStorage.removeItem("TUIKit-userInfo");
      router.replace({ name: "login" });
    }
  },
});
</script>
<!-- 动态引入验证码JS示例 -->
<style lang="scss">
html, body, #app{
    width: 100%;
    height: 100%;
    margin: 0;
}
</style>

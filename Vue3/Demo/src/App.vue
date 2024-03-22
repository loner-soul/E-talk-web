<template>
  <div id="app">
    <router-view :key="locale" :language="locale" @changeLanguage="changeLanguage" />
    <!-- <TUIRoomKit ref="TUIRoomRef" /> -->
  </div>
</template>
<script setup lang="ts">
import { onMounted, ref } from './TUIKit/adapter-vue';
import { RouterView, useRouter } from 'vue-router';
import { TUIStore, StoreName } from '@tencentcloud/chat-uikit-engine';
import { TUILogin } from '@tencentcloud/tui-core';
import { genTestUserSig } from "./TUIKit";
import { vueVersion } from "./TUIKit/adapter-vue";
// import TUIRoomKit from './TUIKit/TUIRoom/index.vue';
const router = useRouter();
const locale = ref<string>('zh');
TUIStore.watch(StoreName.USER, {
  kickedOut: (value: string) => {
    if (value && router.currentRoute.value.name !== 'login') {
      localStorage.removeItem('TUIKit-userInfo');
      router.replace({ name: 'login' });
    }
  },
});
function changeLanguage(language: string) {
  locale.value = language;
}
const userID = ref<string>('');
const loginAgain = () => {
  const options = genTestUserSig({
    SDKAppID: 20005138,
    secretKey: '3032ed63ca186008f0bcdd95a1c9fe1fa58ef7b56404280b8190013099fcb937',
    userID: userID.value
  });
  console.log(options)
  const loginInfo: any = {
    SDKAppID: 20005138,
    userID: userID.value,
    userSig: options.userSig,
    useUploadPlugin: true,
    useProfanityFilterPlugin: true,
    framework: `vue${vueVersion}`
  };
  TUILogin.login(loginInfo).then((res: any) => {
    console.log("登录成功", res);
  }).catch((error: any) => {
    console.log(error);
  })
}
onMounted(() => {
  document.addEventListener('visibilitychange', () => {
    if (document.visibilityState === 'hidden') {
      userID.value = sessionStorage.getItem("userId") || '';
      console.log("退出登录", userID.value);
      TUILogin.logout()
    } else {
      console.log("重新登录成功", userID.value);
      loginAgain()
    }
  })
})
</script>

<style lang="scss">
html,
body,
#app {
  width: 100%;
  height: 100%;
  margin: 0;
}
</style>

<template>
  <div id="app">
    <div class="app-wrapper">
      <LayoutView v-if="$route.meta.layout !== false">
        <transition name="fade" mode="out-in">
          <router-view />
        </transition>
      </LayoutView>

      <router-view v-else />

      <FooterView v-if="$route.meta.footer !== false" />
    </div>
  </div>
</template>

<script>
import { ref, provide, onMounted } from 'vue';
import LayoutView from '@/layouts/LayoutView.vue';
import FooterView from '@/components/FooterView.vue';  // 푸터 import

export default {
  name: 'App',
  components: { LayoutView, FooterView },  // 푸터 등록

  setup() {
    const user = ref(null);

    onMounted(() => {
      const storedUser = localStorage.getItem('user');
      if (storedUser) {
        user.value = JSON.parse(storedUser);
      }

      window.addEventListener('storage', () => {
        const updatedUser = localStorage.getItem('user');
        user.value = updatedUser ? JSON.parse(updatedUser) : null;
      });
    });

    const logout = () => {
      localStorage.removeItem('user');
      user.value = null;
    };

    provide('user', user);
    provide('logout', logout);

    return { user, logout };
  },
};
</script>

<style>
html, body, #app {
  height: 100%;
  margin: 0;
}

.app-wrapper {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

/* 콘텐츠를 확장시키고 푸터는 밑으로 밀기 */
.app-wrapper > *:first-child {
  flex: 1;
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>

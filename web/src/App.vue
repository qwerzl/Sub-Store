<template>
  <v-app>
    <TopToolbar></TopToolbar>
    <v-main>
      <router-view></router-view>
    </v-main>
    <BottomNav ref="bottomNavBar"></BottomNav>
    <v-snackbar :value="successMessage" app bottom color="success" elevation="20">
      {{ successMessage }}
    </v-snackbar>

    <v-snackbar :value="errorMessage" app bottom color="error" elevation="20">
      {{ errorMessage }}
    </v-snackbar>

    <v-overlay :value="isLoading">
      <v-progress-circular indeterminate size="64" color="primary"></v-progress-circular>
    </v-overlay>
  </v-app>
</template>

<script>

import TopToolbar from "@/components/TopToolbar";
import BottomNav from "@/components/BottomNav";
import { showError } from "@/utils";


async function initStore(store) {
  await store.dispatch('FETCH_SUBSCRIPTIONS').catch(() => {
    showError(`无法拉取订阅列表!`);
  });
  await store.dispatch("FETCH_COLLECTIONS").catch(() => {
    showError(`无法拉取组合订阅列表！`);
  });
  await store.dispatch("FETCH_ARTIFACTS").catch(() => {
    showError(`无法拉取配置列表！`);
  });
  await store.dispatch("FETCH_SETTINGS").catch(() => {
    showError(`无法拉取配置列表！`);
  });
  await store.dispatch("FETCH_ENV").catch(() => {
    showError(`无法获取当前运行环境！`);
  });
}

export default {
  components: {
    TopToolbar,
    BottomNav,
  },

  created() {
    initStore(this.$store);

    const vuetify = this.$vuetify;

    if (window.matchMedia) {
      if (window.matchMedia('(prefers-color-scheme: dark)').matches) {
        vuetify.theme.dark = true;
      }
      window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', (e) => {
        console.log(`changed to ${e.matches ? "dark" : "light"} mode`)
        vuetify.theme.dark = e.matches ? true : false;
      });
    }
  },

  mounted (){
    const bottomNavBar = this.$refs.bottomNavBar.$el;
    const height = bottomNavBar.offsetHeight || bottomNavBar.clientHeight;
    this.$store.commit("SET_BOTTOM_NAVBAR_HEIGHT", height);
  },

  computed: {
    successMessage() {
      return this.$store.state.successMessage;
    },
    errorMessage() {
      return this.$store.state.errorMessage;
    },
    isLoading() {
      return this.$store.state.isLoading;
    }
  },

  watch: {
    successMessage() {
      if (this.$store.state.snackbarTimer) {
        clearTimeout(this.$store.state.snackbarTimer);
      }
      const timer = setTimeout(() => {
        this.$store.commit("SET_SUCCESS_MESSAGE", "");
      }, 3000);
      this.$store.commit("SET_SNACK_BAR_TIMER", timer);
    },
    errorMessage() {
      if (this.$store.state.snackbarTimer) {
        clearTimeout(this.$store.state.snackbarTimer);
      }
      const timer = setTimeout(() => {
        this.$store.commit("SET_ERROR_MESSAGE", "");
      }, 3000);
      this.$store.commit("SET_SNACK_BAR_TIMER", timer);
    }
  }
}
</script>

<style lang="scss">
@import "./assets/css/app";
@import "./assets/css/general.css";
</style>

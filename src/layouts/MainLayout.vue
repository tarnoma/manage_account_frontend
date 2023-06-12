<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />

        <q-toolbar-title> Manage Acc App </q-toolbar-title>
        <div v-if="storeLogUser.getUserId">
          Hello {{ storeLogUser.getFullname }} <q-space />
          <q-btn round>
            <q-avatar size="32px"><img :src="storeLogUser.getAvatar" alt="" /></q-avatar>
          </q-btn>
          <q-btn flat color="white" icon="logout" @click="logout" />
        </div>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" show-if-above bordered>
      <q-list>
        <q-item-label header> Essential Links </q-item-label>

        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent, ref } from "vue";
import EssentialLink from "components/EssentialLink.vue";
import { useLoginUserStore } from "../stores/loginUserStore";
import { Notify } from "quasar";
const linksList = [
  {
    title: "Home",
    caption: "Landing page",
    icon: "home",
    link: "/",
  },
  {
    title: "Register",
    caption: "Add new user",
    icon: "person_add",
    link: "/register",
  },
  {
    title: "List",
    caption: "Retrive all users",
    icon: "people_outline",
    link: "/user",
  },
];

export default defineComponent({
  name: "MainLayout",

  components: {
    EssentialLink,
  },

  setup() {
    const leftDrawerOpen = ref(false);
    const storeLogUser = useLoginUserStore();

    return {
      essentialLinks: linksList,
      leftDrawerOpen,
      toggleLeftDrawer() {
        leftDrawerOpen.value = !leftDrawerOpen.value;
      },
      storeLogUser,
    };
  },
  methods: {
    logout() {
      this.storeLogUser.clearStorage();
      Notify.create({
        
        type: "info",
        message: "Logout successfully.",
      });
      this.$router.push("/");
    },
    // getAvatar() {
    //   return this.storeLogUser.getAvatar;
    // },
  },
});
</script>

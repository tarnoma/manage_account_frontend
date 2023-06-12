<template>
  <q-page class="flex flex-center">
    <q-card class="my-card bg-grey-1 q-px-md q-py-md">
      <div class="flex flex-center">
        <q-icon name="account_circle" color="grey-6" size="4rem" />
      </div>
      <q-card-section>
        <q-form
          @submit.prevent="onSubmit"
          @reset="onReset"
          class="q-gutter-md"
          ref="myLoginForm"
        >
          <div>
            <q-input
              outlined
              v-model="username"
              label="Your Username"
              lazy-rules
              :rules="[
                (val) =>
                  (val && val.length > 0) || 'Must be at least 1 character!',
              ]"
            />
          </div>
          <div>
            <q-input
              v-model="password"
              outlined
              :type="isPwd ? 'password' : 'text'"
              label="Your Password"
              lazy-rules
              :rules="[
                (val) =>
                  (val && val.length >= 6) || 'Must be at least 6 characters!',
              ]"
            >
              <template #append>
                <q-icon
                  :name="isPwd ? 'visibility_off' : 'visibility'"
                  class="cursor-pointer"
                  @click="isPwd = !isPwd"
                />
              </template>
            </q-input>
          </div>
          <div>
            <q-btn
              label="Submit"
              type="submit"
              color="primary"
              style="width: 100%"
            />
          </div>
          <div>
            <text-caption class="text-cyan-8">Not registered?
              <a href="/register">Create an Account</a></text-caption>
          </div>
        </q-form>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script>
import { useLoginUserStore } from "../stores/loginUserStore";
import { Notify } from "quasar";
export default {
  name: "LoginPage",
  data() {
    return {
      password: null,
      username: null,
      isPwd: true,
      storeLogUser: useLoginUserStore(),
    };
  },
  methods: {
    onSubmit() {
      const data = {
        username: this.username,
        password: this.password,
      };
      this.$api
        .post("/auth/login", data)
        .then((response) => {
          if (response.status == 200) {
            Notify.create({
              type: "positive",
              message: "Login successfully.",
            });
            this.storeLogUser.userid = response.data.id;
            this.storeLogUser.fullname = response.data.fullname;
            this.storeLogUser.accessToken = response.data.accessToken;
            if (response.data.img != null) {
              this.storeLogUser.avatar =
                this.$RESTURL + "/file/" + response.data.img;
            } else {
              this.storeLogUser.avatar = "default-avatar.png";
            }
            this.$router.push("/user");
          }
        })
        .catch((err) => {
          console.log("invalid login. " + err);
          Notify.create({
            type: "negative",
            message: "Invalid Username or Password.",
          });
        });
      this.$refs.myLoginForm.reset();
    },
    onReset() {
      this.username = null;
      this.password = null;
      this.isPwd = true;
    },
  },
};
</script>

<style></style>

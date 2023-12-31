<template>
  <q-page class="flex flex-center">
  <q-card class="my-card bg-grey-1 q-px-md q-py-md">
    <div class="flex flex-center" v-if="isShowIcon">
      <q-icon name="account_circle" color="grey-6" size="4rem" />
    </div>
    <div class="flex flex-center" v-else>
      <q-img
        :src="imageUrl"
        :ratio="4 / 3"
        spinner-color="primary"
        spinner-size="82px"
      ></q-img>
    </div>
    <q-card-section>
      <q-form
        @submit.prevent="onSubmit"
        @reset="onReset"
        class="q-gutter-md"
        ref="myRegisterForm"
      >
        <div>
          <q-input
            outlined
            v-model="fullname"
            label="Your Fullname"
            lazy-rules
            :rules="[(val) => (val && val.length > 0) || 'Field is required']"
          />
        </div>
        <div>
          <q-input
            v-model="email"
            outlined
            type="email"
            label="Your Email"
            lazy-rules
            :rules="[emailValidate, requiredValidate]"
          />
        </div>
        <div>
          <q-input
            outlined
            v-model="username"
            label="Your Username"
            lazy-rules
            :rules="[requiredValidate]"
          />

          <text-caption
            style="font-size: 0.9em"
            v-if="usernameCaption.show"
            :class="[
              usernameCaption.showClass ? 'text-positive' : 'text-negative',
            ]"
            ><q-icon size="1.5em" :name="usernameCaption.icon" />
            {{ usernameCaption.msg }}</text-caption
          >
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
          <!-- file size 1MB -->
          <q-file
            outlined
            v-model="upload_avatar"
            label="Your Avatar"
            accept=".jpg, .jpeg, .png"
            max-file-size="1048576"
            @rejected="onRejected"
            @update:model-value="updateFile()"
          >
            <template #append>
              <q-icon name="attach_file" />
            </template>
          </q-file>
        </div>
        <div>
          <q-btn label="Reset" type="reset" color="white" text-color="black" />
          <q-btn label="Submit" type="submit" color="primary" class="q-ml-sm" />
        </div>
      </q-form>
    </q-card-section>
  </q-card>
  <DialogComponent v-model="showDialog" :propDialog="dialog" />
  </q-page>
</template>

<script>
import { emailValidate, requiredValidate } from "../utils/validations";
import { useLoginUserStore } from "../stores/loginUserStore";
import DialogComponent from "../components/DialogComponent.vue";
import { Notify } from "quasar";
export default {
  name: "RegisterPage",
  components: {
    DialogComponent,
  },
  data() {
    return {
      password: null,
      username: null,
      fullname: null,
      isPwd: true,
      email: null,
      usernameCaption: {
        show: false,
        showClass: false,
        icon: null,
        msg: null,
      },
      upload_avatar: null,
      isShowIcon: true,
      imageUrl: "",
      storeLogUser: useLoginUserStore(),
      showDialog: false,
      dialog: {
        icon: "",
        msg: "",
        btnType: "",
        iconColor: "primary",
      },
    };
  },
  methods: {
    emailValidate,
    requiredValidate,
    submitData(filename) {
      const data = {
        fullname: this.fullname,
        email: this.email,
        username: this.username,
        password: this.password,
        img: filename,
      };
      this.$api
        .post("/auth/signup", data)
        .then((res) => {
          if (res.status == 200) {
            this.showDialog = true;
            this.dialog.icon = "task_alt";
            this.dialog.iconColor = "primary";
            this.dialog.msg =
              "<div class='text-h6'>Welcome,<br>" +
              res.data.fullname +
              "</div>Your account has been created successfully.";
            this.dialog.btnType = "Signup";
            this.storeLogUser.userid = res.data.id;
            this.storeLogUser.fullname = res.data.fullname;
            this.storeLogUser.accessToken = res.data.accessToken;
            if (res.data.img != null) {
              this.storeLogUser.avatar =
                this.$RESTURL + "/file/" + res.data.img;
            } else {
              this.storeLogUser.avatar = "default-avatar.png";
            }
          }
          this.$refs.myRegisterForm.reset();
        })
        .catch((err) => {
          console.log(err);
          this.showErrDialog(err);
        });
    },
    onSubmit() {
      // this.showErrDialog("This is an error");
      // return;
      if (this.upload_avatar == "") this.upload_avatar = null;
      if (this.upload_avatar) {
        const formData = new FormData();
        formData.append("singlefile", this.upload_avatar);
        this.$api
          .post("/file/upload", formData)
          .then((response) => {
            if (response.status === 200) {
              //continue submit data to db with file
              this.submitData(response.data.uploadFileName);
            }
          })
          .catch((err) => {
            console.log(err);
            this.showErrDialog(err);
          });
      } else {
        //continue submit data to db without file
        this.submitData(null);
      }
    },
    showErrDialog(err) {
      this.showDialog = true;
      this.dialog.icon = "error";
      this.dialog.iconColor = "negative";
      this.dialog.msg = err;
      this.dialog.btnType = "Error";
    },
    // closeDialog(type) {
    //   if (type == "Signup") this.$router.push("/user");
    //   else this.showDialog = false;
    // },
    onReset() {
      this.fullname = null;
      this.email = null;
      this.username = null;
      this.password = null;
      this.isPwd = true;
      this.resetUserCaption();

      this.upload_avatar = null;
      this.isShowIcon = true;
      this.imageUrl = "";      
    },
    resetUserCaption() {
      this.usernameCaption.show = false;
      this.usernameCaption.showClass = false;
      this.usernameCaption.icon = null;
      this.usernameCaption.msg = null;
    },
    usernameValidate() {
      if (this.username) {
        this.$api
          .get("/auth/" + this.username)
          .then((response) => {
            // console.log(JSON.stringify(response.data))
            if (response.data.valid) {
              this.usernameCaption.show = true;
              this.usernameCaption.showClass = true;
              this.usernameCaption.icon = "check_circle_outline";
              this.usernameCaption.msg = "This username is Available.";
            } else {
              this.usernameCaption.show = true;
              this.usernameCaption.showClass = false;
              this.usernameCaption.icon = "highlight_off";
              this.usernameCaption.msg = "This username is NOT Available.";
            }
          })
          .catch((err) => {
            console.log(err);
          });
      } else {
        this.resetUserCaption();
      }
    },
    updateFile() {
      this.isShowIcon = false;
      this.imageUrl = URL.createObjectURL(this.upload_avatar);
    },
    onRejected(rejectedEntries) {
      let msg;
      if (rejectedEntries[0].failedPropValidation == "accept")
        msg = "Only images (jpg, jpeg, png) are allowed";
      else if (rejectedEntries[0].failedPropValidation == "max-file-size")
        msg = "File size cannot be larger than 1MB";
      Notify.create({
        type: "negative",
        message: msg,
      });
    },
  },
  watch: {
    username() {
      this.usernameValidate();
    },
  },
};
</script>

<style></style>

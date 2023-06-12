<template>
  <q-page padding>
    <!-- <div><q-btn color="primary" icon="check" label="OK" 
    @click="showErrDialog('check err')" /></div> -->
    <div v-if="dataReady">
      <q-table
        title="List of Users"
        :rows="rows"
        :columns="columns"
        row-key="id"
        :pagination="paginations"
      >
        <template #body="props">
          <q-tr :props="props">
            <q-td key="id" :props="props">
              {{ props.row.id }}
            </q-td>
            <q-td key="fullname" :props="props">
              {{ props.row.fullname }}
            </q-td>
            <q-td key="email" :props="props">
              {{ props.row.email }}
            </q-td>
            <q-td key="action">
              <q-btn
                flat
                round
                color="primary"
                @click="editRecord(props.row)"
                icon="edit"
              ></q-btn>
              <q-btn
                color="primary"
                icon="delete"
                flat
                round
                @click="deleteRecord(props.row)"
              />
            </q-td>
          </q-tr>
        </template>
      </q-table>
    </div>
    <!-- Loading progress -->
    <div v-else class="flex flex-center">
      <q-circular-progress
        indeterminate
        size="60px"
        color="blue"
        class="q-ma-md"
      />
    </div>
    <!-- Edit Dialog -->
    <q-dialog v-model="form_edit" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar icon="edit" color="primary" text-color="white" />
          <span class="q-ml-sm text-h6">Edit user ID: {{ input.id }}</span>
        </q-card-section>
        <q-card-section>
          <q-input v-model="input.fullname" type="text" label="Fullname" />
          <q-input v-model="input.email" type="text" label="Email" />
          <q-img
            v-if="input.img"
            :src="input.img"
            :ratio="4 / 3"
            spinner-color="primary"
            spinner-size="82px"
          />
          <!-- 
            Maximum size of all files combined in bytes
            1024 = 1K
            1,048,576 = 1M 
           -->
          <q-file
            v-model="uploadFile"
            label="Choose new avatar"
            accept=".jpg, .jpeg, .png"
            max-file-size="1048576"
            @rejected="onRejected"
            @update:model-value="updateFile()"
          >
            <template #append>
              <q-icon name="attach_file" />
            </template>
          </q-file>
        </q-card-section>
        <q-card-actions align="right">
          <q-btn
            color="white"
            text-color="black"
            label="Cancel"
            v-close-popup
            @click="onCancleEdit()"
          />
          <q-btn label="Edit" color="primary" v-close-popup @click="onEdit()" />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- Delete Dialog -->
    <q-dialog v-model="form_delete" persistent>
      <q-card>
        <q-card-section class="row items-center">
          <q-avatar icon="delete" color="primary" text-color="white" />
          <span class="q-ml-sm text-h6">Delete user ID: {{ input.id }}</span>
        </q-card-section>
        <q-card-section>
          <span class="q-ml-sm">FullName: {{ input.fullname }}</span>
        </q-card-section>
        <q-card-actions align="right">
          <q-btn color="white" text-color="black" label="NO" v-close-popup />
          <q-btn
            label="YES"
            color="primary"
            v-close-popup
            @click="onDelete()"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <DialogComponent v-model="showDialog" :propDialog="dialog" />
  </q-page>
</template>

<script>
import { useLoginUserStore } from "../stores/loginUserStore";
import DialogComponent from "../components/DialogComponent.vue";
import { Notify } from "quasar";
const defaultAvatar = "default-avatar.png";
export default {
  name: "ListUserPage",
  components: {
    DialogComponent,
  },
  data() {
    return {
      storeLogUser: useLoginUserStore(),
      dataReady: false,
      // isLogged: false,
      columns: [
        {
          name: "id",
          label: "ID",
          align: "center",
          field: "id",
          sortable: true,
        },
        {
          name: "fullname",
          align: "left",
          label: "FullName",
          field: "fullname",
          sortable: true,
        },
        {
          name: "email",
          label: "Email",
          field: "email",
          sortable: true,
          align: "left",
        },
        {
          name: "action",
          label: "Actions",
          field: "action",
          align: "left",
        },
      ],
      rows: [],
      paginations: {
        rowsPerPage: 7,
      },
      input: [],
      form_edit: false,
      form_delete: false,
      uploadFile: null,
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
    getAllUsers() {
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .get("/auth", { headers })
        .then((response) => {
          if (response.status == 200) {
            response.data.forEach((item, key) => {
              // mod for logined acc
              // if(item.id == this.storeLogUser.getUserId){
              //   this.rows.push(item)
              // }

              if (item.img != null)
                response.data[key].img =
                  this.$RESTURL + "/file/" + response.data[key].img;
            });
            this.rows = response.data;
          }
        })
        .catch((err) => {
          console.log(err);
          Notify.create({
            type: "negative",
            message: "Unauthorized.",
          });
          this.storeLogUser.clearStorage();
          this.$router.push("/");
        });
    },
    editRecord(record) {
      this.input = record;
      this.form_edit = true;
      console.log(JSON.stringify(this.input))
    },
    deleteRecord(record) {
      this.input = record;
      this.form_delete = true;
    },
    updateFile() {
      this.input.img = URL.createObjectURL(this.uploadFile);
    },
    getFileName(filepath) {
      return filepath.substr(filepath.lastIndexOf("/") + 1);
    },
    submitEditData(filename) {
      let img = "";
      if (filename == null) {
        if (this.input.img == null) img = null;
        else img = this.getFileName(this.input.img);
      } else img = filename;
      const data = {
        fullname: this.input.fullname,
        email: this.input.email,
        img: img,
      };
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .put("/auth/" + this.input.id, data, { headers })
        .then((response) => {
          if (response.status == 200) {
            Notify.create({
              type: "positive",
              message: "Updated user ID: " + response.data.id,
            });
            if (this.storeLogUser.userid == response.data.id) {
              this.storeLogUser.fullname = response.data.fullname;
              if (response.data.img != null && this.uploadFile == null) {
                this.storeLogUser.avatar =
                  this.$RESTURL + "/file/" + response.data.img;
              } else {
                this.storeLogUser.avatar = defaultAvatar;
              }
            }
          }
        })
        .catch((err) => {
          console.log(err);
          this.showErrDialog(err);
        });
    },
    onEdit() {
      if (this.uploadFile == "") this.uploadFile = null;
      if (this.uploadFile) {
        const formData = new FormData();
        formData.append("singlefile", this.uploadFile);
        this.$api
          .post("/file/upload", formData)
          .then((response) => {
            if (response.status === 200) {
              //continue submit data to db
              this.submitEditData(response.data.uploadFileName);
              this.uploadFile = null;
            }
          })
          .catch((err) => {
            console.log(err);
            this.showErrDialog(err);
          });
      } else {
        this.submitEditData(null);
      }
      this.getAllUsers;
    },
    showErrDialog(err) {
      this.showDialog = true;
      this.dialog.icon = "error";
      this.dialog.iconColor = "negative";
      this.dialog.msg = err;
      this.dialog.btnType = "Error";
    },
    onCancleEdit() {
      this.getAllUsers();
    },
    onDelete() {
      const headers = {
        "x-access-token": this.storeLogUser.accessToken,
      };
      this.$api
        .delete("/auth/" + this.input.id, { headers })
        .then((response) => {
          if (response.status === 200) {
            Notify.create({
              type: "positive",
              message: "Deleted user ID: " + response.data.id,
            });
            if (this.storeLogUser.userid == response.data.id) {
              this.storeLogUser.clearStorage();
              this.$router.push("/");
            } else {
              this.getAllUsers();
            }
          }
        })
        .catch((err) => {
          console.log(err);
          this.showErrDialog(err);
        });
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
  async mounted() {
    await this.getAllUsers();
    this.dataReady = true;
  },
};
</script>

<style></style>

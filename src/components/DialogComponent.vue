<template>
<!-- :value="propDialogShow" @input="$emit('input')" -->
  <q-dialog  persistent>
    <q-card>
      <q-card-section align="center">
        <q-avatar
          size="100px"
          :icon="propDialog.icon"
          :text-color="propDialog.iconColor"
        />
        <q-space />
        <span class="q-ml-sm" v-html="propDialog.msg"></span>
      </q-card-section>
      <q-card-actions align="center">
        <q-btn
          label="OK"
          color="primary"
          v-close-popup
          @click="closeDialog(propDialog.btnType)"
        />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script>
export default {
  name: "DialogComponent",
//   data(){
//     return{
//       showDialog: true, 
//     }
//   },
  props: {
    propDialog: {
      icon: { type: String },
      msg: { type: String },
      btnType: { type: String },
      iconColor: { type: String },
    },
    // propDialogShow: { type: Boolean, default: false },
  },
  methods: {
    closeDialog(type) {
    //   this.$emit('input');
      if (type == "Signup")
        this.$router.push("/user");
    //   else this.showDialog = false;
    },
  },
};
</script>

<style></style>

<template>
  <div>
    <p>{{ label }}</p>
    <b-field v-if="file==null" class>
      <b-upload v-model="file" accept="image/*" drag-drop class>
        <section class="section full-width">
          <div class="has-text-centered">
            <p>
              <b-icon icon="upload" size="is-large"></b-icon>
            </p>
            <p>Drop your files here or click to upload</p>
          </div>
        </section>
      </b-upload>
    </b-field>
    <div v-if="value" class="preview">
      <img :src="value" />
      <button class="delete is-small" type="button" @click="file = null"></button>
    </div>
  </div>
</template>

<script lang="ts">
import Vue, { PropType } from "vue";
import { Chrome } from "vue-color";

// interface Object {
//   /** The initial value of Object.prototype.constructor is the standard built-in Object constructor. */
//   constructor: Function;
//   // some more properties here
// }

export default Vue.extend({
  components: {
    "chrome-picker": Chrome
  },

  props: {
    value: String,
    label: String
  },

  data() {
    return {
      file: null as File | null
    };
  },

  watch: {
    file() {
      if (this.file) {
        const reader = new FileReader();
        reader.onload = (e: any) => {
          this.$emit("input", e.target.result);
        };
        reader.readAsDataURL(this.file);
      } else {
        this.$emit("input", null);
      }
    }
  }
});
</script>

<style scoped>
.full-width {
  width: 100%;
}

.width400 {
  width: 400px;
}

.preview {
  border-radius: 2px;
  box-shadow: 0 0 2px rgba(0, 0, 0, 0.3), 0 4px 8px rgba(0, 0, 0, 0.3);
}
</style>

<style>
.upload.control,
.upload-draggable {
  width: 100%;
}
</style>
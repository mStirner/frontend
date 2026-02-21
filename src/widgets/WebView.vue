<script>
import { defineComponent } from "vue";
import Iframe from "../components/Iframe.vue";
import Modal from "../components/Modal.vue";

export default defineComponent({
  name: "WebView",
  props: {
    uuid: {
      type: String,
      required: true,
    },
    /*
    url: {
      type: String,
      required: true
    }
      */
  },
  components: {
    Iframe,
    Modal
  },
  data() {
    return {
      showTextbox: false,
      isModalVisible: false,
      url: "http://example.com",
      title: ""
    };
  },
  mounted() {

    let data = window.localStorage.getItem(`widget-${this.uuid}`);

    if (data) {

      let json = JSON.parse(data);

      this.url = json.url;
      this.title = json.title

    }

  },
  methods: {
    editURL() {
      this.isModalVisible = true;
    },
    saveSettings() {

      let data = JSON.stringify({
        url: this.url,
        title: this.title,
      });

      window.localStorage.setItem(`widget-${this.uuid}`, data);

    },
    onModalConfirm() {

      this.url = this.$refs.url.value;

      this.saveSettings();

      this.isModalVisible = false;

    },
    onCloseModal() {
      this.isModalVisible = false;
    },
    closeEdit() {
      this.showTextbox = false;
      this.saveSettings();
    },
    editTitle() {
      this.showTextbox = true;
    }
  },
  menu: [{
    title: "Edit URL",
    method: "editURL",
  }, {
    title: "Edit Title",
    method: "editTitle",
  }, {
    title: "Done editing",
    method: "closeEdit",
  }]
});
</script>


<template>

  <Modal :visible="isModalVisible" @close="onCloseModal" @confirm="onModalConfirm" title="Set URL to display">
    <template #body>
      <div>

        <input type="text" :value="url" placeholder="http://example.com" ref="url" class="w-100" />

      </div>
    </template>
  </Modal>

  <h3> WebView: <input type="text" v-model="title" v-if="showTextbox" />
    <span v-else>
      {{ title }}
    </span>
  </h3>

  <Iframe :src="url" />

</template>

<style scope></style>
<template>

  <Modal :visible="isModalVisible" @close="onCloseModal" @confirm="onModalConfirm" title="Select commands to display">
    <template #body>
      <div class="row" v-for="(commands, endpoint, index) in endpointCommands" :key="index">
        <div class="col">

          <h6> {{ getNameById(endpoint) }}</h6>

          <ul v-if="commands.length > 0">
            <li v-for="(command, i) in commands" :key="i">

              <div class="form-check form-switch">
                <label>

                  <input class="form-check-input" type="checkbox" role="switch" :value="command._id"
                    v-model="wantedCommands">
                  {{ command.name }}

                </label>
              </div>
              <!--
              <label>
                <input type="checkbox" :value="state._id" v-model="wantedCommands"> 
                {{ state.name }}
              </label>
            -->
            </li>
          </ul>
          <span v-else>
            <i class="text-secondary fw-light">no commands</i>
          </span>

          <!-- VISUAL BARRIER (skip for last entry) -->
          <hr v-if="index + 1 < Object.keys(endpointCommands).length" />
          <!-- VISUAL BARRIER (skip for last entry) -->

        </div>
      </div>
    </template>
  </Modal>

  <div class="p-2 h-100">

    <h3>
      Commands: <input type="text" v-model="title" v-if="showTitleTextbox" />
      <span v-else>
        {{ title }}
      </span>
    </h3>

    <div class="container-fluid p-0">
      <div class="h-100">
        <div class="row h-100 display-flex text-center">

          <div class="p-0 col" v-for="cmd in commands" @click="triggerCommand(cmd._id)">
            <Tile>

              <!-- INFO -->
              <span class="text-secondary fw-light">
                {{ getEndpointByCommand(cmd._id)?.name || "undefined" }} / {{ cmd.name }}
                <br />
                <small>
                  ({{ getRoomById(getEndpointByCommand(cmd._id)?.room)?.name || "room not set" }})
                </small>
              </span>
              <!-- INFO -->

              <!--
              <div class="row display-flex text-center" v-bind:key="index" v-for="(param, index) in cmd.params">
                <div>

                  <CommandParameter :param="param" :command="cmd" :shared="{}" @changed="triggerCommand(cmd._id)" />

                </div>
              </div>
              -->

            </Tile>
          </div>

        </div>
      </div>
    </div>

  </div>
</template>

<script>
import { defineComponent } from "vue";
import Modal from "../components/Modal.vue";
import Tile from "../components/Tile.vue";
import CommandParameter from "../components/CommandParameter.vue";
import { itemStore } from "../store.js";
import { request } from "../helper.js";

export default defineComponent({
  name: "Commands",
  setup() {

    const items = itemStore();

    return {
      items
    };

  },
  props: {
    uuid: {
      type: String,
      required: true,
    }
  },
  components: {
    Modal,
    Tile,
    CommandParameter
  },
  data() {
    return {
      title: "",
      isModalVisible: false,
      showTitleTextbox: false,
      endpoint: "",
      wantedCommands: []
    };
  },
  computed: {
    rooms() {
      return this.items.rooms;
    },
    endpoints() {
      return this.items.endpoints;
    },
    endpointCommands() {
      return this.endpoints.reduce((acc, item) => {

        // acc[item._id] = this.item.states; // 
        acc[item._id] = item?.commands || []; // 

        return acc;

      }, {});
    },
    commands() {
      return this.endpoints.map(({ commands }) => {
        return commands;
      }).flat().filter((cmd) => {
        return this.wantedCommands.includes(cmd._id);
      });
    }
  },
  mounted() {
    this.loadLocalStorage();
  },
  created() { },
  methods: {
    triggerCommand(_id) {

      let endpoint = this.getEndpointByCommand(_id);

      request(`/api/endpoints/${endpoint._id}/commands/${_id}`, {
        method: "POST"
      });

    },
    selectCommands() {
      this.isModalVisible = true;
    },
    onCloseModal() {
      this.isModalVisible = false;
    },
    onModalConfirm() {

      this.isModalVisible = false;
      this.saveLocalStorage();

    },
    dispatchEvent(event) {
      console.log("Dispatch event in child called", event);
    },
    getStateById_asdf24443(_id) {
      return this.endpoints.map(({ commands }) => {
        return commands;
      }).flat().find((command) => {
        return command._id === _id;
      });
    },
    getRoomById(_id) {
      return this.rooms.find((room) => {
        return room._id === _id;
      }) || {};
    },
    getEndpointById(_id) {
      return this.endpoints.find((endpoint) => {
        return endpoint._id === _id;
      });
    },
    getNameById(_id) {
      return this.getEndpointById(_id)?.name;
    },
    getEndpointByCommand(_id) {
      return this.endpoints.find(({ commands }) => {
        return commands.find((command) => {
          return command._id === _id;
        });
      });
    },
    editTitle() {
      this.showTitleTextbox = true;
    },
    closeEdit() {
      this.showTitleTextbox = false;
      this.saveLocalStorage();
    },
    saveLocalStorage() {

      let str = JSON.stringify({
        title: this.title,
        wantedCommands: this.wantedCommands
      });

      window.localStorage.setItem(`widget-${this.uuid}`, str);

    },
    loadLocalStorage() {

      let str = window.localStorage.getItem(`widget-${this.uuid}`);

      if (!str) {
        return;
      }

      let data = JSON.parse(str);

      this.title = data.title;
      this.wantedCommands = data.wantedCommands;

    }
  },
  menu: [{
    title: "Select commands",
    method: "selectCommands",
  }, {
    title: "Edit title",
    method: "editTitle",
  }, {
    title: "Done editing",
    method: "closeEdit",
  }]
});
</script>

<style scoped>
.row.display-flex {
  display: flex;
  flex-wrap: wrap;
}

.row.display-flex>[class*="col-"] {
  flex-grow: 0;
  /* 1) set max-height to shrink tiles */
  /*max-height: 300px;*/
}

small {
  font-size: 10px;
}
</style>
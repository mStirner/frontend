<script>
import { defineComponent } from "vue";

import { useNotificationStore } from "@dafcoe/vue-notification";
const { setNotification } = useNotificationStore();

export default defineComponent({
  name: "Stopwatch",
  props: {
    uuid: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      title: "",
      duration: 0,
      counter: 0,
      showTitleTextbox: false,
      showDurationTextbox: false,
      interval: null
    };
  },
  mounted() {

    let str = window.localStorage.getItem(`widget-${this.uuid}`);

    if (!str) {
      return;
    }

    let config = JSON.parse(str);

    this.title = config.title;
    this.duration = Number(config.duration || 0);
    this.counter = this.duration;

  },
  methods: {
    editTitle() {
      this.showTitleTextbox = true;
    },
    closeEdit() {

      this.showTitleTextbox = false;
      this.save();

    },
    dispatchEvent(event) {
      console.log("diuspatchEvent called in notes.vue", event);
    },
    save() {

      let data = JSON.stringify({
        title: this.title,
        counter: this.counter
      });

      window.localStorage.setItem(`widget-${this.uuid}`, data);

    },
    resetCounter() {
      this.counter = 0;
      clearInterval(this.interval);
    },
    startCounter() {

      this.closeEdit();

      this.interval = setInterval(() => {
        this.counter += 10;
      }, 10);

    },
    stopCounter() {
      clearInterval(this.interval);
      this.resetCounter();
      this.interval = null;
    },
    pauseCounter() {
      clearInterval(this.interval);
      this.interval = null;
    }
  },
  computed: {
    time() {

      const ms = this.counter;

      const hours = Math.floor(ms / 3600000);
      const minutes = Math.floor((ms % 3600000) / 60000);
      const seconds = Math.floor((ms % 60000) / 1000);
      const milliseconds = Math.floor((ms % 1000) / 10);;

      return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}.${milliseconds.toString().padStart(2, '0')}`;

    }
  },
  menu: [{
    title: "Edit Title",
    method: "editTitle",
  }, {
    title: "Done editing",
    method: "closeEdit",
  }],
});
</script>


<template>
  <div class="p-2">
    <h3>
      Stopwatch: <input type="text" v-model="title" v-if="showTitleTextbox" />
      <span v-else>
        {{ title }}
      </span>
    </h3>
    <div class="w-100">
      <div class="container text-center">
        <div class="row">
          <div class="col h-100">
            <button class="btn btn-outline-primary btn-block" @click="pauseCounter()" v-if="interval">Pause</button>
            <button class="btn btn-outline-primary btn-block" @click="startCounter()" v-if="!interval">Start</button>
          </div>
          <div class="col">
            <div class="container">

              <div class="row" v-if="!showDurationTextbox">
                <h3>{{ time }}</h3>
              </div>

            </div>
          </div>
          <div class="col">
            <button class="btn btn-outline-primary btn-block" @click="stopCounter()">Stop</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
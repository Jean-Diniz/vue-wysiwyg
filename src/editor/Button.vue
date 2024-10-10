<template lang="pug">
div(@mousedown="onBtnClick")
  a(:class="'vw-btn-'+module.title", v-html="module.icon")

  .dashboard(
    v-show="showDashboard",
    ref="dashboard"
  )
    component(
      v-if="module.render",
      v-once,
      ref="moduleDashboard",
      :is="module",
      @exec="exec",
      :uid="uid"
      :options="options"
      :translations="translations"
    )

</template>
<script>
import bus from 'src/editor/bus.js';

export default {
  props: ["module", "options", "translations"],

  data() {
    return {
      showDashboard: false,
    }
  },

  computed: {
    uid() {
      return this.$parent._uid
    }
  },

  methods: {
    closeDashboard() {
      this.showDashboard = false;
    },

    openDashboard() {
      this.showDashboard = true;
    },

    exec() {
      this.$parent.exec.apply(null, arguments)
    },

    detectBrowser() {
      var userAgent = navigator.userAgent;
      if (userAgent.indexOf("Edg") > -1) {
        return "Microsoft Edge";
      } else if (userAgent.indexOf("Chrome") > -1) {
        return "Chrome";
      } else if (userAgent.indexOf("Firefox") > -1) {
        return "Firefox";
      } else if (userAgent.indexOf("Safari") > -1) {
        return "Safari";
      } else if (userAgent.indexOf("Opera") > -1) {
        return "Opera";
      } else if (userAgent.indexOf("Trident") > -1 || userAgent.indexOf("MSIE") > -1) {
        return "Internet Explorer";
      }

      return "Unknown";
    },

    onBtnClick($event) {
      if (this.detectBrowser() === 'Firefox') {
        $event.preventDefault();
      }
      if (this.module.action !== undefined)
        this.exec.apply(null, this.module.action);

      else if (this.module.customAction !== undefined) {
        this.module.customAction(bus.utils).forEach(a => this.exec.apply(null, a));
      }

      else if (
        this.module.render !== undefined &&
        (!this.$refs.dashboard || !this.$refs.dashboard.contains($event.target))
      ) {
        this.showDashboard = !this.showDashboard;
        bus.emit(`${this.uid}_${this.showDashboard ? "show" : "hide"}_dashboard_${this.module.title}`);
        return;
      }
    }
  }
}
</script>

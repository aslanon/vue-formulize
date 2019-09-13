<template>
  <div id="app">
    <mycomponent :customElement="myElement" :error="{}"></mycomponent>
  </div>
</template>

<script>
import mycomponent from "@/components/mycomponent.vue";
export default {
  name: "app",
  components: {
    mycomponent
  },
  data() {
    return {
      error: { type: "error", message: "asdasd" },
      myElement: [
        { key: "onur", value: "aslan", element: "span", class: "aslancik" },
        {
          key: "buton",
          value: "buttoncuk",
          element: "button",
          class: "aslancik",
          event: {
            name: "onclick",
            function: function(e) {
              console.log(e);
            }
          }
        }
      ]
    };
  },
  methods: {
    set() {
      this.error = { type: "error", message: "455345" };
    }
  }
};
</script>
<style>
.aslancik {
  background: blue;
}
</style>


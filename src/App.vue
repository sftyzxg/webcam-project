<template>
  <div>
    <!-- Other page content -->
    <Webcam></Webcam>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import WebcamComponent from "../src/components/WebCam.vue";

export default defineComponent({
  components: {
    Webcam: WebcamComponent,
  },
  // Other component logic
});
</script>

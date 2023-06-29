<template>
  <div>
    <video
      id="myVideo"
      ref="videoRef"
      width="320"
      height="240"
      autoplay
    ></video>
    <button @click="startCamera">Start Camera</button>
    <button @click="stopCamera">Stop Camera</button>
    <!-- <button @click="takeSnapshot">Take Snapshot</button> -->
    <img
      class="snapshot"
      id="outImage"
      :src="ImageUrl"
      alt="Snapshot"
      :style="{ display: 'none' }"
    />
    <!-- <img id="outImage" src="../images/test.jpg" alt="Snapshot" /> -->
  </div>
  <div id="myFrame" class="myFrame">
    <div
      v-for="(data, index) in dataRef"
      :key="'frame' + index"
      :style="{
        position: 'absolute',
        border: '2px solid red',
        left: data.detection._box._x + videoPos.left + 'px',
        top: data.detection._box._y + videoPos.top + 'px',
        height: data.detection._box._height + 'px',
        width: data.detection._box._width + 'px',
        color: 'red',
      }"
    >
      {{ data.gender }},{{ parseInt(data.age) }},{{ data.finalExpressions }}
    </div>
    <div
      v-for="(data, index) in previewRef"
      :key="'frame' + index"
      :style="{
        position: 'absolute',
        border: '2px solid red',
        left: data.detection._box._x + inputImgPos.left + 'px',
        top: data.detection._box._y + inputImgPos.top + 'px',
        height: data.detection._box._height + 'px',
        width: data.detection._box._width + 'px',
        color: 'red',
      }"
    >
      {{ data.gender }},{{ parseInt(data.age) }},{{ data.finalExpressions }}
    </div>
  </div>
  <!-- <h2 v-for="(data, index) in dataRef" :key="'ageGender' + index">
      person{{ index }}:Age:{{ parseInt(data.age) }},Gender:{{ data.gender }}
    </h2> -->
  <div>
    <label for="uploadInput" class="upload-button">
      <span>Upload Image</span>
      <input
        id="uploadInput"
        type="file"
        accept="image/*"
        @change="handleFileUpload"
      />
    </label>
    <img id="previewImage" :src="previewUrl" />
  </div>
</template>

<script lang="ts" setup>
import { onMounted, ref, computed } from "vue";
import * as faceapi from "face-api.js";
const videoRef = ref<HTMLVideoElement | null>(null);
let mediaStream: MediaStream | null = null;
const ImageUrl = ref();
const dataRef = ref();
let timerId: number | undefined = undefined;
const previewUrl = ref("");
const previewRef = ref();
const videoPos = ref({ top: 0, left: 0 });
const inputImgPos = ref({ top: 0, left: 0 });

const startCamera = async () => {
  try {
    if (timerId) {
      clearInterval(timerId);
    }
    mediaStream = await navigator.mediaDevices.getUserMedia({ video: true });
    if (videoRef.value) {
      videoRef.value.srcObject = mediaStream;
      timerId = window.setInterval(takeSnapshot, 1000);
    }
  } catch (error) {
    console.error("Error starting camera:", error);
  }
};

const stopCamera = () => {
  if (mediaStream) {
    mediaStream.getTracks().forEach((track) => track.stop());
    mediaStream = null;
    clearInterval(timerId);
    if (videoRef.value) {
      videoRef.value.srcObject = null;
      dataRef.value = null;
    }
  }
};

const takeSnapshot = () => {
  if (videoRef.value) {
    const canvas = document.createElement("canvas");
    canvas.width = 320;
    canvas.height = 240;
    const ctx = canvas.getContext("2d", { willReadFrequently: true });
    if (ctx) {
      ctx.drawImage(videoRef.value, 0, 0, canvas.width, canvas.height);
      ImageUrl.value = canvas.toDataURL("image/jpeg");
      detectFace();
    }
  }
};

const detectFace = async () => {
  const input = document.getElementById("outImage")!;
  const videoElement = document.getElementById("myVideo");
  if (videoElement) {
    const rect = videoElement.getBoundingClientRect();
    videoPos.value.left = rect.left;
    videoPos.value.top = rect.top;
  }
  // console.log(input);

  const canvas = faceapi.createCanvasFromMedia(input as HTMLImageElement);

  const detection = await faceapi
    .detectAllFaces(canvas)
    .withFaceExpressions()
    .withAgeAndGender();
  // console.log("detecction:", detection);
  dataRef.value = detection;

  dataRef.value.map((data: any, index: number) => {
    // console.log(data);
    let maxPro = 0;
    let maxEmo = "";
    for (const emo in data.expressions) {
      if (data.expressions[emo] > maxPro) {
        maxPro = data.expressions[emo];
        maxEmo = emo;
      }
    }
    dataRef.value[index].finalExpressions = maxEmo;
  });
  // console.log(dataRef.value);
};

const loadModels = async () => {
  await faceapi.nets.ssdMobilenetv1.loadFromUri("/models");
  await faceapi.nets.faceExpressionNet.loadFromUri("/models");
  await faceapi.nets.ageGenderNet.loadFromUri("/models");
  // console.log(faceapi.nets);
};

const detectInputImg = async () => {
  const inputImg = document.getElementById("previewImage")!;
  await new Promise<void>((resolve) => {
    inputImg.addEventListener("load", () => {
      resolve();
    });
  });

  const rect = inputImg.getBoundingClientRect();
  inputImgPos.value.left = rect.left;
  inputImgPos.value.top = rect.top;
  const canvas = faceapi.createCanvasFromMedia(inputImg as HTMLImageElement);
  const detection = await faceapi
    .detectAllFaces(canvas)
    .withFaceExpressions()
    .withAgeAndGender();
  // console.log("Previewdetecction:", detection);
  previewRef.value = detection;

  previewRef.value.map((data: any, index: number) => {
    let maxPro = 0;
    let maxEmo = "";
    for (const emo in data.expressions) {
      if (data.expressions[emo] > maxPro) {
        maxPro = data.expressions[emo];
        maxEmo = emo;
      }
    }
    previewRef.value[index].finalExpressions = maxEmo;
  });
};

const handleFileUpload = (event: Event) => {
  const file =
    (event.target instanceof HTMLInputElement && event.target.files?.[0]) ||
    null;
  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      previewUrl.value = e.target?.result as string;
    };
    reader.readAsDataURL(file);
  }
  detectInputImg();
};

onMounted(async () => {
  await loadModels();
});
</script>

<style></style>

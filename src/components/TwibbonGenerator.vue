<template>
  <div class="container text-center mt-5 bg-white">
    <h1 class="fw-semibold">Twibbon Generator</h1>
    <input
      v-model="name"
      type="text"
      class="form-control my-3"
      placeholder="Masukkan Nama"
    />
    <button
      class="btn btn-primary btn-md d-flex align-items-center justify-content-center gap-2 mx-auto"
      @click="generateTwibbon"
      :disabled="isGenerating"
    >
      <span
        v-if="isGenerating"
        class="spinner-border spinner-border-sm"
        role="status"
        aria-hidden="true"
      ></span>
      {{ isGenerating ? "Generating..." : "Generate Twibbon" }}
    </button>

    <canvas ref="canvas" class="mt-3 d-none"></canvas>

    <div v-if="imageUrl" class="button-download gap-2 mt-2">
      <a
        :href="imageUrl"
        :download="downloadFileName"
        class="btn btn-success w-100"
        >Download</a
      >
      <button @click="shareImage" class="btn btn-secondary w-100">Share</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const name = ref("");
const imageUrl = ref(null);
const canvas = ref(null);
const isGenerating = ref(false);

const downloadFileName = computed(() => {
  const sanitized = name.value.trim().replace(/\s+/g, "-").toLowerCase();
  return sanitized
    ? `twibbon-idul-fitri-${sanitized}.png`
    : "twibbon-idul-fitri.png";
});

const generateTwibbon = () => {
  if (!name.value.trim()) return;
  isGenerating.value = true;

  const ctx = canvas.value.getContext("2d");

  const img = new Image();
  img.src = "/twibbon.jpg";
  img.onload = () => {
    const targetWidth = 410;
    const targetHeight = 512;

    const aspectRatio = img.width / img.height;
    let newWidth = targetWidth;
    let newHeight = targetHeight;

    if (img.width > img.height) {
      newHeight = targetWidth / aspectRatio;
    } else {
      newWidth = targetHeight * aspectRatio;
    }

    canvas.value.width = targetWidth;
    canvas.value.height = targetHeight;
    canvas.value.classList.remove("d-none");

    ctx.drawImage(img, 0, 0, newWidth, newHeight);

    const boxX = 80;
    const boxY = 395;
    const boxWidth = 250;
    const boxHeight = 40;

    // Fill name to image
    ctx.font = "bold 15px Montserrat";
    ctx.fillStyle = "yellow";
    ctx.textAlign = "center";
    // ctx.fillText(name.value, targetWidth / 2, targetHeight - 40);
    ctx.fillText(name.value, boxX + boxWidth / 2, boxY + boxHeight / 2 + 6);

    imageUrl.value = canvas.value.toDataURL("image/png");

    isGenerating.value = false;
  };
};

const shareImage = async () => {
  if (!imageUrl.value || !navigator.canShare) {
    alert("Sharing tidak didukung di perangkat ini.");
    return;
  }

  try {
    const response = await fetch(imageUrl.value);
    const blob = await response.blob();
    const file = new File([blob], downloadFileName.value, {
      type: "image/png",
    });

    if (navigator.canShare({ files: [file] })) {
      await navigator.share({
        title: "Twibbon Idul Fitri",
        text: "Selamat Idul Fitri! 🌙✨",
        files: [file],
      });
    } else {
      alert("Fitur berbagi tidak didukung.");
    }
  } catch (error) {
    console.error("Gagal berbagi:", error);
    alert("Terjadi kesalahan saat berbagi.");
  }
};
</script>

<style scoped>
img {
  max-width: 100%;
  height: auto;
}
div {
  font-family: "Montserrat", sans-serif;
  border: 2px solid #e3e6ed;
  box-shadow: none;
}
.container {
  padding: 20px;
  border-radius: 10px;
}
input.form-control {
  border: 1px solid #e3e6ed;
  box-shadow: none;
}
canvas {
  border-radius: 10px;
}
.button-download {
  border: none !important;
  box-shadow: none !important;
  background: transparent !important;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}
</style>

<template>
  <div class="container text-center mt-5 bg-white">
    <h1 class="fw-semibold">Twibbon Generator</h1>
    <input
      v-model="name"
      type="text"
      class="form-control my-3"
      placeholder="Insert your name"
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

    <div v-if="imageUrl.length > 0" class="mt-3">
      <div class="d-flex flex-column align-items-center">
        <div
          v-for="(url, index) in imageUrl"
          :key="index"
          class="text-center mb-3"
        >
          <img :src="url" class="img-thumbnail" width="250" />
          <div class="d-flex flex-column align-items-center gap-2">
            <a
              :href="url"
              :download="downloadFileName(index)"
              class="btn btn-success w-100"
              >Download</a
            >
            <button
              @click="shareImage(url, index)"
              class="btn btn-secondary w-100"
            >
              Share
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const name = ref("");
const imageUrl = ref([]);
const canvas = ref(null);
const isGenerating = ref(false);
const twibbonImages = ["/twibbon1.jpg", "/twibbon2.jpg", "/twibbon3.jpg"];

const downloadFileName = (index) => {
  const sanitized = name.value.trim().replace(/\s+/g, "-").toLowerCase();
  return sanitized
    ? `twibbon-${index + 1}-${sanitized}.png`
    : `twibbon-${index + 1}.png`;
};

const generateTwibbon = () => {
  if (!name.value.trim()) return;
  isGenerating.value = true;
  imageUrl.value = [];

  const ctx = canvas.value.getContext("2d");
  const generateImage = (src, index) => {
    return new Promise((resolve) => {
      const img = new Image();
      img.src = src;
      img.onload = () => {
        const targetWidth = 410;
        const targetHeight = 512;
        canvas.value.width = targetWidth;
        canvas.value.height = targetHeight;

        const boxX = 80;
        const boxY = 420;
        const boxWidth = 250;
        const boxHeight = 40;

        ctx.clearRect(0, 0, targetWidth, targetHeight);
        ctx.drawImage(img, 0, 0, targetWidth, targetHeight);

        ctx.font = "bold 17px Montserrat";
        ctx.fillStyle = "white";
        ctx.textAlign = "center";
        // ctx.fillText(name.value, targetWidth / 2, targetHeight - 40);
        ctx.fillText(name.value, boxX + boxWidth / 2, boxY + boxHeight / 2 + 6);

        imageUrl.value.push(canvas.value.toDataURL("images/png"));
        resolve();
      };
    });
  };

  Promise.all(
    twibbonImages.map((src, index) => generateImage(src, index))
  ).then(() => {
    isGenerating.value = false;
  });

  // img.onload = () => {
  //   const targetWidth = 410;
  //   const targetHeight = 512;

  //   const aspectRatio = img.width / img.height;
  //   let newWidth = targetWidth;
  //   let newHeight = targetHeight;

  //   if (img.width > img.height) {
  //     newHeight = targetWidth / aspectRatio;
  //   } else {
  //     newWidth = targetHeight * aspectRatio;
  //   }

  //   canvas.value.width = targetWidth;
  //   canvas.value.height = targetHeight;
  //   canvas.value.classList.remove("d-none");

  //   ctx.drawImage(img, 0, 0, newWidth, newHeight);

  //   const boxX = 80;
  //   const boxY = 395;
  //   const boxWidth = 250;
  //   const boxHeight = 40;

  //   ctx.font = "bold 15px Montserrat";
  //   ctx.fillStyle = "yellow";
  //   ctx.textAlign = "center";
  //   // ctx.fillText(name.value, targetWidth / 2, targetHeight - 40);
  //   ctx.fillText(name.value, boxX + boxWidth / 2, boxY + boxHeight / 2 + 6);

  //   imageUrl.value = canvas.value.toDataURL("image/png");

  //   isGenerating.value = false;
  // };
};

const shareImage = async (url, index) => {
  if (!navigator.canShare) {
    alert("Sharing tidak didukung di perangkat ini.");
    return;
  }

  try {
    const response = await fetch(url);
    const blob = await response.blob();
    const file = new File([blob], downloadFileName(index), {
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
.container {
  padding: 20px;
  border-radius: 10px;
  font-family: "Montserrat", sans-serif;
  border: 2px solid #e3e6ed;
  box-shadow: none;
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

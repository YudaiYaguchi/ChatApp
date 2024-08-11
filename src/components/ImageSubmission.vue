<template>
  <input class="Image" type="file" multiple @change="loadImages" />
</template>

<script>
import { inject } from "vue";

export default {
  setup() {
    const imageFiles = inject("imageFiles"); // injectでimageFilesを取得

    const loadImages = (event) => {
      const files = event.target.files;
      Array.from(files).forEach((file) => {
        if (file && file.type.startsWith("image/")) {
          const reader = new FileReader();
          reader.onload = (e) => {
            const imageSrc = e.target.result;
            imageFiles.value.unshift(imageSrc);
          };
          reader.readAsDataURL(file);
        } else {
          alert("画像ファイルを選択してください。");
        }
      });
    };

    return {
      loadImages,
    };
  },
};
</script>

<style scoped>
.Image {
  padding: 10px;
}

.ImageDisplay {
  width: 200px;
  height: 200px;
  object-fit: cover;
  border: 1px solid #ccc;
  margin: 5px;
}
</style>

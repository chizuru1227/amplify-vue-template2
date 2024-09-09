<template>
  <div
    class="drag-drop-area"
    @dragover.prevent
    @dragenter.prevent
    @dragleave.prevent
    @drop.prevent="handleDrop"
  >
    <p>ファイルをここにドラッグ＆ドロップしてください</p>
    <input type="file" ref="fileInput" @change="handleFileChange" hidden />
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import { uploadData } from "aws-amplify/storage";

const fileInput = ref<HTMLInputElement | null>(null);

function handleDrop(event: DragEvent) {
  const files = event.dataTransfer?.files;
  if (files && files.length) {
    uploadFile(files[0]);
  }
}

function handleFileChange(event: Event) {
  const input = event.target as HTMLInputElement;
  if (input.files && input.files.length) {
    uploadFile(input.files[0]);
  }
}

async function uploadFile(fileData: File) {
  const reader = new FileReader();
  reader.readAsDataURL(fileData);
  reader.onload = async (e) => {
    console.log("fileData", fileData);
    console.log("fileName", fileData.name);

    // ファイル名からスペース除去(これはいずれutilに)
    let noSpaces = Array.from(fileData.name)
      .filter((char) => char !== " ")
      .join("");
    console.log("noSpaces", noSpaces);

    const file = {
      data: fileData,
      filePath: noSpaces,
    };

    try {
      const result = await uploadData({
        data: file.data,
        path: `test/123456/${file.filePath}`,
      });
      console.log("result", result);
    } catch (e) {
      console.log("error", e);
    }
  };

  /*
  
    try {
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData,
      })
  
      if (!response.ok) {
        throw new Error('アップロードに失敗しました')
      }
  
      console.log('アップロード成功')
    } catch (error) {
      console.error(error)
    }
      */
}
</script>

<style scoped>
.drag-drop-area {
  border: 2px dashed #ccc;
  padding: 20px;
  text-align: center;
  cursor: pointer;
}
</style>

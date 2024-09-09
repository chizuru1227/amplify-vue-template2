<template>
  <main>
    <h1>My todos</h1>
    <button @click="createTodo">+ new</button>
    <ul>
      <li v-for="todo in todos" :key="todo.id" @click="deleteTodo(todo.id)">
        {{ todo.content }}
      </li>
    </ul>
    <div>
      🥳 App successfully hosted. Try creating a new todo.
      <br />
      <a href="https://docs.amplify.aws/gen2/start/quickstart/nextjs-pages-router/">
        Review next steps of this tutorial.
      </a>
    </div>
    <div>
      <p>ファイルをここにドラッグ＆ドロップしてください</p>
      <input type="file" ref="fileInput" @change="handleFileChange" />
    </div>
  </main>
</template>


<script setup lang="ts">
import '@/assets/main.css';
import { onMounted, ref } from 'vue';
import type { Schema } from '../../amplify/data/resource';
import { generateClient } from 'aws-amplify/data';

const client = generateClient<Schema>();

// create a reactive reference to the array of todos
const todos = ref<Array<Schema['Todo']["type"]>>([]);

function listTodos() {
  client.models.Todo.observeQuery().subscribe({
    next: ({ items, isSynced }) => {
      todos.value = items
    },
  });
}

function createTodo() {
  client.models.Todo.create({
    content: window.prompt("Todo content")
  }).then(() => {
    // After creating a new todo, update the list of todos
    listTodos();
  });
}

function deleteTodo(id: string) {
  client.models.Todo.delete({ id })
}

// fetch todos when the component is mounted
onMounted(() => {
  listTodos();
});


// 追加部分
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
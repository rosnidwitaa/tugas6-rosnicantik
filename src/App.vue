<template>
  <div>
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br />
      <textarea v-model="form.content" placeholder="Content" class="input"></textarea><br />
      <button type="submit" class="btn">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div class="article-content">
          <strong>{{ article.title }}</strong><br />
          {{ article.content }}
        </div>
        <div class="article-actions">
          <button @click="edit(article)" class="btn btn-edit">Edit</button>
          <button @click="remove(article.id)" class="btn btn-delete">Delete</button>
        </div>
      </li>
    </ul>
    <button @click="load" class="btn load-btn">Load Articles</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios({ url, method, data: form });
        
        // Assuming successful save, update list and reset form
        articles.value = articles.value.filter(article => article.id !== form.id);
        articles.value.push(response.data);
        
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    onMounted(load);

    return { form, articles, save, load, edit, remove };
  },
};
</script>

<style>
/* CSS Styling */
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
}

.form {
  margin-bottom: 20px;
}

.input {
  width: 100%;
  padding: 8px;
  margin-bottom: 10px;
  border: 1px solid #cccccc;
  border-radius: 5px;
  color: black;
}

.btn {
  padding: 8px 16px;
  background-color: #6f42c1;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn:hover {
  background-color: #5a3dc8;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  background-color: #ea00a0;
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 10px;
  margin-bottom: 10px;
}

.article-content {
  margin-bottom: 10px;
  color: black;
}

.article-actions {
  text-align: right;
}

.btn-edit {
  background-color: #28a745;
}

.btn-delete {
  background-color: #2e053b;
}

.load-btn {
  background-color: #007bff;
  margin-top: 20px;
}
</style>

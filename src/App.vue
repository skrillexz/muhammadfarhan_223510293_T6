<template>
  <div class="container">
    <h1>Manajemen Artikel</h1>
    <form @submit.prevent="save" class="article-form">
      <input type="text" v-model="form.title" placeholder="Judul" /><br />
      <textarea v-model="form.content" placeholder="Konten"></textarea><br />
      <button type="submit" class="btn save">Simpan</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <strong>{{ article.title }}</strong><br />
        <p>{{ article.content }}</p>
        <button v-if="article.title && article.content" @click="edit(article)" class="btn edit">Ubah</button>
        <button v-if="article.title && article.content" @click="deleteArticle(article.id)" class="btn delete">Hapus</button>
      </li>
    </ul>
    <button @click="load" class="btn load">Muat Artikel</button>
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
        console.log('Artikel dimuat:', response.data);
      } catch (error) {
        console.error('Kesalahan memuat artikel:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        console.log(`Menyimpan artikel dengan metode: ${method}, url: ${url}, data:`, form);
        const response = await axios({
          method: method,
          url: url,
          data: { title: form.title, content: form.content },
        });
        console.log('Artikel disimpan:', response.data);

        if (form.id) {
          // Perbarui artikel dalam array articles dengan data baru dari respon server
          articles.value = articles.value.map((article) =>
            article.id === form.id ? response.data : article
          );
        } else {
          // Tambahkan artikel baru dengan ID dari respon server
          articles.value.push(response.data);
        }

        // Reset form
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Kesalahan menyimpan artikel:', error);
      }
    }

    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
        console.log(`Artikel dengan id ${id} dihapus`);
      } catch (error) {
        console.error('Kesalahan menghapus artikel:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, load, deleteArticle };
  },
};
</script>

<style scoped>
body {
  font-family: Arial, sans-serif;
  background-color: #f0f2f5;
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.container {
  background-color: #fff;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  max-width: 600px;
  width: 100%;
}

h1 {
  color: #333;
  margin-bottom: 20px;
  text-align: center;
}

.article-form {
  margin-bottom: 20px;
}

.article-form input,
.article-form textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
}

.article-form button {
  width: 100%;
  padding: 10px;
  border: none;
  border-radius: 5px;
  background-color: #5cb85c;
  color: #fff;
  font-size: 16px;
  cursor: pointer;
}

.article-form button:hover {
  background-color: #4cae4c;
}

.article-list {
  list-style: none;
  padding: 0;
}

.article-item {
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 5px;
  margin-bottom: 10px;
  background-color: #f9f9f9;
}

.article-item strong {
  display: block;
  margin-bottom: 5px;
  font-size: 18px;
}

.article-item p {
  margin: 0 0 10px;
}

.btn {
  padding: 10px;
  border: none;
  border-radius: 5px;
  color: #fff;
  cursor: pointer;
  margin-right: 5px;
}

.btn.edit {
  background-color: #5bc0de;
}

.btn.edit:hover {
  background-color: #31b0d5;
}

.btn.delete {
  background-color: #d9534f;
}

.btn.delete:hover {
  background-color: #c9302c;
}

.btn.load {
  background-color: #0275d8;
  color: #fff;
  width: 100%;
}

.btn.load:hover {
  background-color: #025aa5;
}
</style>

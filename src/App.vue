<template>
  <div>
    <form @submit.prevent="save">
      <input type="text" v-model="form.title" placeholder="Judul"/><br />
      <textarea v-model="form.content" placeholder="Konten"></textarea><br />
      <button type="submit">Simpan</button>
    </form>
    <ul>
      <li v-for="article in articles" :key="article.id">
        <strong>{{ article.title }}</strong><br />
        {{ article.content }}<br />
        <button v-if="article.title && article.content" @click="edit(article)">Ubah</button>
        <button v-if="article.title && article.content" @click="deleteArticle(article.id)">Hapus</button>
      </li>
    </ul>
    <button @click="load">Muat</button>
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
div {
  color: white;
  background-color: black;
  padding: 20px;
}

input, textarea {
  width: 100%;
  margin-bottom: 10px;
}

button {
  margin: 5px 0;
}
</style>

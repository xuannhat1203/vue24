<template>
  <div class="container">
    <div class="header">
      <div class="search-container">
        <input
          v-model="search"
          placeholder="Nhập từ khóa tìm kiếm"
          class="search-input"
        />
        <select class="filter-select">
          <option>Lọc bài viết</option>
        </select>
      </div>
      <button @click="openModal('add')" class="add-btn">
        Thêm mới bài viết
      </button>
    </div>
    <table class="post-table">
      <thead>
        <tr>
          <th>STT</th>
          <th>Tiêu đề</th>
          <th>Hình ảnh</th>
          <th>Ngày viết</th>
          <th>Trạng thái</th>
          <th>Chức năng</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(post, index) in filterBook" :key="post.id">
          <td>{{ index + 1 }}</td>
          <td>{{ post.title }}</td>
          <td><img :src="post.image" alt="Post Image" class="post-image" /></td>
          <td>{{ post.date }}</td>
          <td>
            <span v-if="post.status">Đã xuất bản</span>
            <span v-else>Ngừng xuất bản</span>
          </td>
          <td>
            <button @click="blockPost(post.id)" class="block-btn">Chặn</button>
            <button @click="openModal('edit', post)" class="edit-btn">
              Sửa
            </button>
            <button @click="deletePost(post.id)" class="delete-btn">Xóa</button>
          </td>
        </tr>
      </tbody>
    </table>
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <div style="display: flex; background-color: red; justify-self: end">
          <button @click="showModal = false" class="close-btn">×</button>
        </div>
        <h2>
          {{ modalMode === "edit" ? "Sửa bài viết" : "Thêm mới bài viết" }}
        </h2>
        <form @submit.prevent="handleSubmit">
          <label for="title">Tên bài viết</label>
          <input v-model="newPost.title" id="title" required />

          <label for="image">Hình ảnh</label>
          <input
            v-model="newPost.image"
            id="image"
            placeholder="URL hình ảnh"
          />

          <label for="content">Nội dung</label>
          <div class="rich-text-editor">
            <textarea
              v-model="newPost.content"
              id="content"
              rows="5"
              placeholder="Nhập nội dung"
            ></textarea>
          </div>
          <div class="modal-footer">
            <button type="button" @click="handleReset" class="reset-btn">
              Làm mới
            </button>
            <button type="submit" class="publish-btn">
              {{ modalMode === "edit" ? "Cập nhật" : "Xuất bản" }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import axios from "axios";

const search = ref("");
const showModal = ref(false);
const modalMode = ref("add");
const currentPostId = ref(null);
const newPost = ref({
  title: "",
  image: "",
  content: "",
  status: "Đã xuất bản",
  date: new Date().toLocaleDateString(),
});

const posts = ref([]);
const getData = async () => {
  const response = await axios.get("http://localhost:3000/post");
  posts.value = response.data;
};

onMounted(() => {
  getData();
});
const handleSubmit = async () => {
  if (modalMode.value === "edit") {
    await axios.put(
      `http://localhost:3000/post/${currentPostId.value}`,
      newPost.value
    );
  } else {
    const newBook = {
      id: posts.value.length + 1,
      title: newPost.value.title,
      image: newPost.value.image,
      content: newPost.value.content,
      status: true,
      date: new Date().toLocaleDateString(),
    };
    await axios.post("http://localhost:3000/post", newBook);
  }
  showModal.value = false;
  getData();
};
const openModal = (mode, post = null) => {
  modalMode.value = mode;
  if (mode === "edit" && post) {
    currentPostId.value = post.id;
    newPost.value = { ...post };
  } else {
    handleReset();
  }
  showModal.value = true;
};

const deletePost = async (id) => {
  await axios.delete(`http://localhost:3000/post/${id}`);
  getData();
};

const handleReset = () => {
  newPost.value = {
    title: "",
    image: "",
    content: "",
    status: "Đã xuất bản",
    date: new Date().toLocaleDateString(),
  };
};
const blockPost = async (id) => {
  const updatePost = {
    status: false,
  };
  const res = await axios.put(`http://localhost:3000/post/${id}`, updatePost);
  getData();
};
const filterBook = computed(() => {
  return posts.value.filter(post => 
    post.title && post.title.toLowerCase().includes(search.value.toLowerCase())
  );
});
</script>

<style scoped>
.container {
  font-family: Arial, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.search-container {
  display: flex;
  gap: 10px;
}

.search-input,
.filter-select {
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.search-input {
  width: 300px;
}

.filter-select {
  width: 150px;
}

.add-btn {
  padding: 8px 16px;
  background-color: #1a73e8;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.post-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.post-table th,
.post-table td {
  border: 1px solid #e0e0e0;
  padding: 12px;
  text-align: left;
}

.post-table th {
  background-color: #f8f9fa;
  font-weight: bold;
}

.post-image {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
}

.published {
  background-color: #defec8;
  color: #07892c;
  padding: 4px 8px;
  border-radius: 5px;
  font-size: 15px;
  border: 1px solid black;
}

.draft {
  background-color: #fce8e6;
  color: #d93025;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
}

.block-btn,
.edit-btn,
.delete-btn {
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 5px;
}

.block-btn {
  background-color: #fbbc04;
  color: white;
}

.edit-btn {
  background-color: #1a73e8;
  color: white;
}

.delete-btn {
  background-color: #ea4335;
  color: white;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 4px;
  width: 400px;
  position: relative;
}

/* .close-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
} */

.modal-footer {
  display: flex;
  justify-content: space-between;
  margin-top: 20px;
}

.reset-btn {
  background-color: #f1c40f;
  border: none;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}

.publish-btn {
  background-color: #28a745;
  border: none;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}
</style>

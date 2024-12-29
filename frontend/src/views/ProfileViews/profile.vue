<!-- eslint-disable vue/multi-word-component-names -->
<template>
  <div class="profile">
    <div class="profile_header">
      <div class="profile_picture">
        <img src="../../../public/assets/유메노아카링.png" alt="프로필 사진" class="profile_img" />
      </div>
      <div class="profile_info">
        <div class="uuu">
          <h2 class="username">사용자 이름</h2>
          <button class="edit_profile_button">프로필 편집</button>
        </div>
        <div class="profile_stats">
          <span>게시물 0</span>
          <span>팔로워 0</span>
          <span>팔로우 0</span>
        </div>
        <p class="name">이름</p>
      </div>
    </div>
    <div class="tjstjs">
      <div class="post_gallery">
        <h3>게시물</h3>
      </div>
    </div>
  </div>
</template>

<script setup></script>

<style scoped>
.profile {
  padding: 20px;
  margin-left: 600px;
}

.profile_header {
  display: flex;
  align-items: center;
  gap: 20px;
}

.profile_picture,
.profile_img {
  margin-right: 50px;
  margin-top: 10px;
  width: 200px;
  height: 200px;
  border-radius: 50%;
}

.profile_info {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 5px;
}

.username {
  width: 150px;
  font-size: 24px;
  font-weight: bold;
}

.name {
  font-size: 18px;
  color: gray;
}

.profile_stats {
  display: flex;
  width: 400px;
  gap: 40px;
  font-size: 18px;
  color: #333;
}

.edit_profile_button {
  padding: 10px;
  width: 140px;
  background-color: #d6d6d6;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 18px;
}

.post_gallery {
  margin-top: 20px;
}

.post_grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

.post {
  width: 100%;
  height: auto;
  border-radius: 5px;
}
.uuu {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
</style>

<template>
  <div class="community-view">
    <!-- 게시물 검색 -->
    <div class="search-bar">
      <input 
        v-model="searchQuery" 
        @input="searchPosts" 
        type="text" 
        placeholder="게시물 검색" />
    </div>

    <!-- 게시물 리스트 -->
    <div class="post-list">
      <div v-for="post in filteredPosts" :key="post.id" class="post-item">
        <h3>{{ post.title }}</h3>
        <p class="post-author">작성자: {{ post.author }}</p>
        <p class="post-date">{{ post.date }}</p>
      </div>
    </div>

    <!-- 글쓰기 버튼 -->
    <div class="write-post">
      <button @click="openWritePostModal">글쓰기</button>
    </div>

    <!-- 글쓰기 모달 -->
    <div v-if="isWritePostModalOpen" class="write-post-modal">
      <div class="modal-content">
        <h3>새 글 작성</h3>
        <textarea v-model="newPostContent" placeholder="내용을 입력하세요..."></textarea>
        <button @click="submitPost">작성 완료</button>
        <button @click="closeWritePostModal">닫기</button>
      </div>
    </div>
  </div>
</template>

  
<script>
export default {
  name: "CommunityView",
  data() {
    return {
      posts: [], // 게시물 목록
      searchQuery: "", // 검색 쿼리
      newPostContent: "", // 새 게시물 내용
      isWritePostModalOpen: false, // 글쓰기 모달 열기 여부
    };
  },
  computed: {
    // 검색된 게시물 리스트
    filteredPosts() {
      return this.posts.filter(post =>
        post.title.toLowerCase().includes(this.searchQuery.toLowerCase()) ||
        post.author.toLowerCase().includes(this.searchQuery.toLowerCase())
      );
    },
  },
  methods: {
    // 게시물 목록 불러오기 (API 연동 필요)
    fetchPosts() {
      // 여기서 API 호출하여 게시물 목록을 가져오는 작업
      // 예시:
      // this.posts = await axios.get("/api/community/posts");
    },

    // 게시물 검색
    searchPosts() {
      // 검색 중 실시간으로 게시물 필터링
      // computed 속성에 의해 자동으로 필터링됨
    },

    // 글쓰기 모달 열기
    openWritePostModal() {
      this.isWritePostModalOpen = true;
    },

    // 글쓰기 모달 닫기
    closeWritePostModal() {
      this.isWritePostModalOpen = false;
      this.newPostContent = "";
    },

    // 게시물 작성 완료
    submitPost() {
      // 새 게시물 API 호출
      // 예시:
      // await axios.post("/api/community/post", { content: this.newPostContent });
      this.posts.push({
        id: Date.now(), // 임시 ID (추후 DB에서 받아오기)
        title: `새 글 ${this.posts.length + 1}`,
        author: "작성자",
        date: new Date().toLocaleDateString(),
        content: this.newPostContent,
      });
      this.closeWritePostModal();
    },
  },
  created() {
    this.fetchPosts(); // 컴포넌트가 로드되면 게시물 목록을 가져옴
  },
};
</script>

<style scoped>
.community-view {
  padding: 20px;
}

.search-bar {
  margin-bottom: 20px;
}

.search-bar input {
  width: 100%;
  padding: 8px;
  font-size: 16px;
}

.post-list {
  margin-bottom: 20px;
}

.post-item {
  border: 1px solid #ddd;
  padding: 10px;
  margin-bottom: 10px;
}

.write-post button {
  padding: 10px 20px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
}

.write-post button:hover {
  background-color: #45a049;
}

.write-post-modal {
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
  border-radius: 5px;
  width: 400px;
}

textarea {
  width: 100%;
  height: 150px;
  margin-bottom: 20px;
}
</style>
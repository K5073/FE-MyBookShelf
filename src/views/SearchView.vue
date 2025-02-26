<template>
  <div class="search-view">
    <h1>검색 결과</h1>
    
    <!-- 검색 결과 리스트 -->
    <ul>
      <li v-for="(book, index) in books" :key="index" class="search-result-item">
        <div class="book-item">
          <div class="book-cover">
            <img :src="book.thumbnail" alt="책 표지" />
          </div>
          <div class="book-info">
            <p class="book-title">{{ book.bookTitle }}</p>
            <p class="book-author">{{ book.bookAuthor }}</p>
            <p class="book-publisher">{{ book.bookPublisher }}</p>
            <button @click="selectBook(book)" class="select-book-button">선택</button>
          </div>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios';
axios.defaults.baseURL = 'http://localhost:8081'; // 기본 API 주소 설정

export default {
  data() {
    return {
      books: [], // 검색된 책 목록
      searchQuery: this.$route.query.query || '', // 검색어
    };
  },
  async mounted() {
    // 페이지가 로드될 때 검색어로 책 검색
    if (this.searchQuery) {
      await this.searchBooks(this.searchQuery);
    }
  },
  watch: {
    '$route.query.query'(newQuery) {
      // 검색어가 변경될 때마다 책을 새로 검색
      if (newQuery) {
        this.searchBooks(newQuery);
      }
    }
  },
  methods: {
    async searchBooks() {
      try {
        // fetch를 사용한 책 검색 API 호출
        const response = await axios.get(`/api/books/search`, {
          params: { query: this.manualTitle },
        });

        // 응답 상태 코드 확인
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        // JSON으로 변환
        const data = await response.json();

        // 검색된 책 목록 처리
        this.books = data.books.map(book => ({
          title: book.title,
          author: book.author,
          publisher: book.publisher,
          isbn: book.isbn,
          cover: book.cover // 책 표지 URL
        }));
      } catch (error) {
        console.error('책 검색 실패:', error);
      }
    },
  },
};
</script>

<style scoped>
.search-view {
  padding: 20px;
}

.search-result-item {
  display: flex;
  margin-bottom: 20px;
}

.book-item {
  display: flex;
  align-items: center;
  border: 1px solid #ddd;
  padding: 10px;
  border-radius: 5px;
}

.book-cover {
  width: 80px;
  height: 120px;
  margin-right: 15px;
}

.book-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 4px;
}

.book-info {
  flex: 1;
}

.book-title {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 5px;
}

.book-author,
.book-publisher {
  font-size: 14px;
  margin-bottom: 5px;
}

.select-book-button {
  padding: 5px 15px;
  font-size: 14px;
  cursor: pointer;
  background-color: #FFA500;
  color: white;
  border: none;
  border-radius: 4px;
  margin-top: 10px;
}

.select-book-button:hover {
  background-color: #ff8c00;
}
</style>
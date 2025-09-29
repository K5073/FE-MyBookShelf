<template>
  <div class="search-view">
    <h2>검색 결과</h2>

    <div v-if="loading" class="loading">검색 중...</div>

    <!-- 검색 결과 -->
    <div v-if="loading">검색 중...</div>
    <div v-else-if="searchResults.length === 0">검색 결과가 없습니다.</div>
    <div v-else class="results-grid">
      <div v-for="book in paginatedResults" :key="book.isbn" class="book-card">
        <img :src="book.cover" alt="책 표지" class="book-cover" />
        <h4>{{ book.title }}</h4>
        <p>{{ book.author }}</p>
        <button @click="openBookshelfModal(book)">저장</button>
      </div>
    </div>

    <!-- 📌 책장 선택 모달 -->
    <div v-if="showBookshelfModal" class="modal-overlay">
      <div class="modal-content">
        <h3>책장을 선택하세요</h3>
        <div v-if="bookshelves.length === 0">책장이 없습니다.</div>
        <div v-else class="bookshelf-list">
          <select v-model="selectedBookshelf">
            <option disabled value="">책장을 선택해주세요</option>
            <option v-for="shelf in bookshelves" :key="shelf.bookshelfId" :value="shelf.bookshelfId">
              {{ shelf.bookshelfName }}
            </option>
          </select>
        </div>
        <div class="modal-actions">
          <button @click="saveBookToShelf">저장하기</button>
          <button @click="closeBookshelfModal">취소</button>
        </div>
      </div>
    </div>

    <div v-if="searchResults.length === 0 && !loading" class="no-result">
      검색 결과가 없습니다.
    </div>

    <!-- 페이지네이션 -->
    <div v-if="totalPages > 1" class="pagination">
      <button @click="goToPage(currentPage - 1)" :disabled="currentPage === 1">
        이전
      </button>
      <span>{{ currentPage }} / {{ totalPages }}</span>
      <button @click="goToPage(currentPage + 1)" :disabled="currentPage === totalPages">
        다음
      </button>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      searchQuery: "",
      searchResults: [],
      loading: false,
      bookshelfList: [],            // 책장 목록
      selectedBookshelf: {},        // 각 책별 선택된 책장
      currentPage: 1,
      itemsPerPage: 20,
      bookshelves: [],           // 책장 목록
      // 📌 책장 모달 관련
      showBookshelfModal: false,
      selectedBook: null,
      selectedBookshelves: [], // 체크된 책장 ID 배열
    };
  },
  created() {
    // 라우터에서 전달된 검색어(query 파라미터) 읽기
    this.searchQuery = this.$route.query.query || "";
    if (this.searchQuery) {
      this.fetchBooks();
    }
    this.fetchBookshelves();
  },
  watch: {
    '$route.query.query'(newQuery) {
      this.searchQuery = newQuery;
      this.fetchBooks();
    }
  },
  computed: {
    paginatedResults() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.searchResults.slice(start, end);
    },
    totalPages() {
      return Math.ceil(this.searchResults.length / this.itemsPerPage);
    },
  },

  mounted() {
    this.searchQuery = this.$route.query.query || "";
    if (this.searchQuery) {
      this.fetchBooks();
    }
    this.fetchBookshelves();  // ✅ 추가
  },

  methods: {
    // 책 검색
    async fetchBooks() {
      this.loading = true;
      this.searchResults = [];
      try {
        const response = await axios.get(`/api/books/search`, {
          params: { query: this.searchQuery },
        });

        if (response.data.books.length === 0) {
          alert("검색 결과가 없습니다.");
        }

        this.searchResults = response.data.books.map(book => ({
          title: book.title,
          author: book.author,
          publisher: book.publisher,
          isbn: book.isbn,
          cover: book.cover,
        }));
      } catch (error) {
        console.error("검색 실패:", error);
      } finally {
        this.loading = false;
      }
    },

    goToPage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page;
      }
    },
    async fetchBookshelves() {
      const user = JSON.parse(localStorage.getItem('user'));
      const userId = user ? user.userId : null;
      try {
        const response = await axios.get(`/api/bookshelf/${userId}`);
        if (response.data.isSuccess) {
          this.bookshelves = response.data.result;
        } else {
          console.error("책장 불러오기 실패:", response.data.message);
        }
      } catch (error) {
        console.error("책장 목록 조회 실패:", error);
      }
    },

    // 📌 선택한 책장에 책 저장
    async saveBookToShelf() {
      if (!this.selectedBook || !this.selectedBookshelf) {
        alert("책장을 선택해주세요!");
        return;
      }

      try {
        await axios.post(`/api/bookshelf/add`, {
          bookshelfId: this.selectedBookshelf,
          title: this.selectedBook.title,
          author: this.selectedBook.author,
          publisher: this.selectedBook.publisher,
          isbn: this.selectedBook.isbn,
          cover: this.selectedBook.cover,
        });

        alert("책이 선택한 책장에 저장되었습니다!");
        this.closeBookshelfModal();
      } catch (error) {
        console.error("책 저장 실패:", error);
      }
    },
    openBookshelfModal(book) {
      this.selectedBook = book;       // 어떤 책을 저장할지 기억
      this.selectedBookshelf = "";    // 책장 선택 초기화
      this.showBookshelfModal = true; // 모달 열기
    },
    closeBookshelfModal() {
      this.showBookshelfModal = false;
    },
  },
};
</script>

<style scoped>
.search-container {
  max-width: 900px;
  margin: auto;
  padding: 20px;
}
.search-bar {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
.results-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 15px;
}
.book-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 10px;
  text-align: center;
  background: #fff;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}
.book-cover {
  width: 100px;
  height: 150px;
  object-fit: cover;
  margin-bottom: 10px;
}
.book-card button {
  margin-top: 10px;
  padding: 6px 12px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
.book-card button:hover {
  background: #0056b3;
}

/* 📌 모달 스타일 */
.modal-overlay {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background: rgba(0,0,0,0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}
.modal-content {
  background: #fff;
  padding: 20px;
  border-radius: 12px;
  width: 400px;
  max-height: 80vh;
  overflow-y: auto;
}
.bookshelf-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin: 15px 0;
}
.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}
.modal-actions button {
  padding: 8px 14px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
.modal-actions button:first-child {
  background: #28a745;
  color: #fff;
}
.modal-actions button:last-child {
  background: #dc3545;
  color: #fff;
  }
  .pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 12px;
  margin-top: 20px;
}
.pagination button {
  padding: 6px 12px;
  border: none;
  border-radius: 6px;
  background: #007bff;
  color: #fff;
  cursor: pointer;
}
.pagination button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

</style>
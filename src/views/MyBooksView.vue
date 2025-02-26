<template>
  <div class="my-books">
    <!-- 책장 헤더 -->
    <div class="bookshelf-header">
      <div class="bookshelf-controls">
        <div v-if="isRenaming">
          <input 
            v-model="newBookshelfName" 
            @keyup.enter="renameBookshelf" 
            class="rename-input" 
            type="text" 
            placeholder="새 책장 이름 입력"
          />
        </div>

        <!-- 책장 리스트 -->
        <select v-else v-model="currentBookshelf" @change="selectBookshelf" class="bookshelf-select">
          <option value="null" disabled>---------- 책장을 추가해주세요 ----------</option>
          <option v-for="shelf in bookshelves" :key="shelf.bookshelfId" :value="shelf.bookshelfId">
            {{ shelf.bookshelfName }}
          </option>
        </select>

        <button @click="toggleRenameMode" class="rename-button" :disabled="isNoBookshelf">
          {{ isRenaming ? "저장" : "이름 변경" }}
        </button>
        <button @click="openAddBookshelfModal" class="add-bookshelf-button">+</button>
        <button @click="deleteBookshelf" class="delete-bookshelf-button" :disabled="isNoBookshelf">🗑</button>
        <button @click="openSidebar" class="add-book-button" :disabled="!currentBookshelf">책 등록</button>

        <!-- 추가된 버튼들 -->
        <button @click="openRecommendationModal" class="recommend-button">추천받기</button>
        <button @click="deleteBooksFromShelf" class="edit-button">편집</button>
      </div>
    </div>

    <!-- 네모난 책장 폼 -->
    <div class="bookshelf">
      <div class="book-grid">
        <div
          v-for="(book, index) in currentBookshelfBooks"
          :key="index"
          class="book-placeholder">
          <div v-if="book.cover" class="book-cover">
            <img :src="book.cover || 'default-cover.jpg'" alt="책 표지" />
          </div>
        </div>
      </div>
    </div>

    <!-- 책장 추가 모달 -->
    <div v-if="isAddBookshelfModalOpen" class="add-bookshelf-modal">
      <div class="add-bookshelf-modal-content">
        <label for="new-bookshelf-name">책장 이름</label>
        <input
          type="text"
          id="new-bookshelf-name"
          v-model="newBookshelfNameForModal"
          placeholder="책장 이름 입력" />
        <button @click="addBookshelf" class="create-bookshelf-button">생성하기</button>
        <button @click="closeAddBookshelfModal" class="close-modal-button">취소</button>
      </div>
    </div>

    <!-- 추천받기 모달 -->
    <div v-if="isRecommendationModalOpen" class="recommendation-modal">
      <div class="recommendation-modal-content">
        <h3>추천받기</h3>
        <div class="recommendation-options">
          <button @click="setRecommendationType('age')" :class="{ active: recommendationType === 'age' }">연령별 대출순 추천</button>
          <button @click="setRecommendationType('rating')" :class="{ active: recommendationType === 'rating' }">평점 추천</button>
          <button @click="setRecommendationType('keyword')" :class="{ active: recommendationType === 'keyword' }">키워드 추천</button>
        </div>
        <div class="recommendation-actions">
          <button @click="fetchRecommendations" class="recommend-books-button">책 5권 추천받기</button>
          <button @click="closeRecommendationModal" class="close-modal-button">취소</button>
        </div>
      </div>
    </div>

    <!-- 사이드바 -->
    <div v-if="isSidebarOpen" class="sidebar">
      <div class="sidebar-content">
        <button class="close-button" @click="closeSidebar">✖</button>
        <h3>책 등록</h3>
        <div class="registration-options">
          <button @click="setRegisterType('manual')" :class="{ active: registerType === 'manual' }">
            직접 등록
          </button>
          <button @click="setRegisterType('isbn')" :class="{ active: registerType === 'isbn' }">
            ISBN 등록
          </button>
          <button @click="setRegisterType('photo')" :class="{ active: registerType === 'photo' }">
            사진 등록
          </button>
        </div>

        <!-- 직접 등록 폼 -->
        <div v-if="registerType === 'manual'" class="manual-form">
          <label for="title">책 제목</label>
          <input type="text" id="title" v-model="manualTitle" placeholder="책 제목 입력" />
          <button @click="searchManual">검색</button>
        </div>

        <!-- ISBN 등록 폼 -->
        <div v-if="registerType === 'isbn'" class="isbn-form">
          <label for="isbn">ISBN</label>
          <input type="text" id="isbn" v-model="isbn" placeholder="ISBN 입력" />
          <button @click="searchISBN">검색</button>
        </div>

        <!-- 사진 등록 폼 -->
        <div v-if="registerType === 'photo'" class="photo-options">
          <button @click="openFileInput" class="file-upload-button">첨부파일</button>
          <button @click="openCamera" class="camera-button">사진 촬영</button>
        </div>

        <!-- 검색된 책들 -->
        <div v-if="searchResults.length" class="search-results">
          <h4>검색된 책들:</h4>
          <ul>
            <li v-for="(book, index) in paginatedResults" :key="index">
              <div class="search-book-item">
                <div class="book-cover">
                  <img :src="book.cover" alt="책 표지" />
                </div>
                <div class="book-info">
                  <p class="book-title" :title="book.title">
                    {{ book.title.length > 25 ? book.title.slice(0, 25) + '...' : book.title }}
                  </p>
                  <p class="book-author">{{ book.author }}</p>
                  <button @click="selectBook(book)" class="select-book-button">선택</button>
                </div>
              </div>
            </li>
          </ul>

          <!-- 페이지네이션 버튼 -->
          <div class="pagination">
            <button @click="changePage(currentPage - 1)" :disabled="currentPage === 1">이전</button>
            <span>{{ currentPage }} / {{ totalPages }}</span>
            <button @click="changePage(currentPage + 1)" :disabled="currentPage === totalPages">다음</button>
          </div>
        </div>

        <div class="sidebook-grid">
          <div
            v-for="book in books"
            :key="book.id"
            class="sidebook-item"
            :class="{ selected: selectedBooks.includes(book) }"
            @click="toggleSelection(book)"
          >
            <img :src="book.cover" alt="book cover" />
            <p>{{ book.title }}</p>
            <p>{{ book.author }}</p>
          </div>
        </div>

        <!-- 확인 모달 -->
        <div v-if="showConfirmModal" class="confirm-modal">
          <div class="confirm-modal-content">
            <p>'{{ selectedBook.title }}' 을 저장하시겠습니까?</p>
            <button @click="confirmAddBook">예</button>
            <button @click="cancelAddBook">아니요</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

axios.defaults.baseURL = 'http://localhost:8081'; // 기본 API 주소 설정

export default {
  name: "MyBooksView",
  data() {
    return {
      bookshelves: [], // 기본값은 빈 배열로 설정
      currentBookshelf: null, // 기본값은 'null'로 설정
      isRenaming: false,
      newBookshelfName: "",  // 입력 필드에 사용할 새 책장 이름
      isNoBookshelf: false, // 선택된 책장이 없을 때 처리할 상태
      newBookshelfNameForModal: "", // 모달에 입력할 새 책장 이름
      isSidebarOpen: false,
      registerType: "manual",
      manualTitle: "",
      isbn: "",
      isAddBookshelfModalOpen: false, // 책장 추가 모달 열기 여부
      isRecommendationModalOpen: false, // 추천받기 모달 열기 여부
      recommendationType: "", // 추천 타입
      searchResults: [], // 검색된 책 정보
      books: [], // 책 배열 초기화
      currentPage: 1, // 현재 페이지
      booksPerPage: 6, // 페이지당 책 개수
      selectedBook: null,
      showConfirmModal: false, // 책 선택버튼 누르면 뜨는 창
    };
  },
  created() {
    this.fetchBookshelves();
  },

  computed: {
    currentBookshelfBooks() {
      const shelf = this.bookshelves.find(
        // (shelf) => shelf.name === this.currentBookshelf
        (shelf) => shelf.bookshelfId === this.currentBookshelf // ID로 비교
      );
      return shelf ? shelf.book : []; // 책장에 등록된 책 목록 반환
    },

    // 사이드바 결과 페이지 쪽수
    paginatedResults() {
      const start = (this.currentPage - 1) * this.booksPerPage;
      const end = this.currentPage * this.booksPerPage;
      return this.searchResults.slice(start, end);
    },
    totalPages() {
      return Math.ceil(this.searchResults.length / this.booksPerPage); // 총 페이지 수
    },
  },

  methods: {
    // 특정 사용자의 책장 불러오기 (책장 목록 조회 API)
    async fetchBookshelves() {
      const user = JSON.parse(localStorage.getItem('user'));
      const userId = user ? user.userId : null; // userId를 가져옵니다.

      try {
        const response = await axios.get(`/api/bookshelf/${userId}`); // userId를 URL에 포함
        this.bookshelves = response.data.result; // result 필드를 사용하여 책장과 책 정보를 포함한 배열로 설정
      } catch (error) {
        console.error('책장 목록 조회 실패:', error);
      }
    },

    // 책장 생성 API
    async addBookshelf() {
      const user = JSON.parse(localStorage.getItem('user'));
      const userId = user ? user.userId : null;  // userId를 가져옵니다.

      if (!this.newBookshelfNameForModal.trim()) {
        alert("책장 이름을 입력해 주세요.");
        return;
      }

      try {
        const response = await axios.post('/api/bookshelf/create', {
          userId: userId,
          bookshelfName: this.newBookshelfNameForModal,
        });

        if (response.data.isSuccess) { // 응답 상태를 isSuccess로 확인
          alert(`${this.newBookshelfNameForModal} 책장이 추가되었습니다!`); // 알림 메시지
          
          // 책장 목록을 다시 불러옵니다.
          await this.fetchBookshelves(); 

          this.newBookshelfNameForModal = "";
          this.isAddBookshelfModalOpen = false; // 모달 닫rl
        } else {
          alert("책장 추가 실패: " + response.data.message);
        }
      } catch (error) {
        console.error("책장 추가 중 오류 발생:", error);
      }
    },

    // 책장 이름 변경 API
    async renameBookshelf() {
      if (!this.currentBookshelf) return; // 선택된 책장이 없을 경우 처리
      if (!this.newBookshelfName.trim()) {
        alert("새 이름을 입력해 주세요.");
        return; // 새 이름이 비어있을 경우 처리
      }

      try {
        const response = await axios.patch("/api/bookshelf/edit", {
          bookshelfId: this.currentBookshelf, // 수정할 책장 ID
          bookshelfName: this.newBookshelfName, // 새 책장 이름
        });

        if (response.data.isSuccess) {
          alert("책장 이름이 수정되었습니다!");
          await this.fetchBookshelves(); // 변경된 데이터 다시 가져오기
          this.isRenaming = false; // 이름 변경 모드 종료
          this.newBookshelfName = ""; // 입력 필드 초기화
        } else {
          alert("책장 이름 수정 실패: " + response.data.message);
        }
      } catch (error) {
        console.error('책장 이름 수정 실패:', error);
      }
    },

    toggleRenameMode() {
      if (this.currentBookshelf === null) {
        alert("책장을 먼저 선택해주세요.");
        return;
      }

      if (this.isRenaming) {
        this.renameBookshelf();  // 이름 변경 모드에서 저장 진행
      } else {
        // 이름 변경 모드 시작
        const shelf = this.bookshelves.find(shelf => shelf.bookshelfId === this.currentBookshelf);
        if (shelf) {
          this.newBookshelfName = shelf.bookshelfName; // 현재 선택된 책장 이름으로 초기화
        }
      }
      this.isRenaming = !this.isRenaming; // 모드 토글
    },

    // 책장 삭제 API
    async deleteBookshelf() {
      if (!this.selectedBookshelf) return; // 선택된 책장이 없을 경우 처리
      
      if (!confirm('정말 이 책장을 삭제하시겠습니까?')) return;

      try {
        const response = await axios.delete(`/api/bookshelf/delete/${this.selectedBookshelf}`);

        if (response.data.isSuccess) { // isSuccess로 확인
          alert("책장이 삭제되었습니다!");
          this.fetchBookshelves(); // 변경된 데이터 다시 가져오기
        } else {
          alert("책장 삭제 실패: " + response.data.message);
        }
      } catch (error) {
        console.error('책장 삭제 실패:', error);
      }
    },

    selectBookshelf() {
      this.selectedBookshelf = this.currentBookshelf; // 현재 선택된 책장 ID를 저장
    },
    openAddBookshelfModal() {
      this.isAddBookshelfModalOpen = true;
    },
    closeAddBookshelfModal() {
      this.isAddBookshelfModalOpen = false;
      this.newBookshelfNameForModal = "";
    },
    openSidebar() {
      this.isSidebarOpen = true;
    },
    closeSidebar() {
      this.isSidebarOpen = false;
      this.manualTitle = "";
      this.isbn = "";
    },

    // 추천받기 모달 열기
    openRecommendationModal() {
      this.isRecommendationModalOpen = true;
    },

    // 추천받기 모달 닫기
    closeRecommendationModal() {
      this.isRecommendationModalOpen = false;
    },

    setRecommendationType(type) {
      this.recommendationType = type;
    },

    async fetchRecommendations() {
      // 추천받기 로직을 여기에 구현
      // 예: API 호출 후 추천받은 책 목록을 표시
      alert(`추천받은 책 ${this.recommendationType}에 따라 책을 추천받았습니다.`);
      // 추천받은 책을 books 배열에 추가할 수 있습니다.
      this.closeRecommendationModal(); // 모달 닫기
    },

    // 검색된 책을 책장에 넣는 작업
    async selectBook(book) {
      this.selectedBook = book; // 선택한 책 정보를 저장
      this.showConfirmModal = true; // 모달 표시
    },

    confirmAddBook() {
      if (!this.currentBookshelf) {
        alert("책장을 먼저 선택해주세요.");
        return;
      }

      // 선택된 책장의 책 개수 확인
      const currentBookshelfBooks = this.currentBookshelfBooks; // 현재 책장에 있는 책 목록
      if (currentBookshelfBooks.length >= 10) {
        alert("한 책장에는 최대 10권의 책만 추가할 수 있습니다.");
        return; // 10권 이상일 경우 추가하지 않음
      }

      try {
        // API 요청: 선택한 책의 ISBN을 이용해 책장에 추가
          axios.post(`/api/bookshelf/${this.currentBookshelf}/register`, null, {
            params: { isbn13: this.selectedBook.isbn } // this.selectedBook 사용
        });

        // 책장 목록을 다시 불러와 최신 상태 반영
        this.fetchBookshelves(); 

        // 책장에 추가된 책을 화면에 즉시 반영
        alert(`'${this.selectedBook.title}' 책이 책장에 추가되었습니다.`);
      } catch (error) {
        console.error('책 추가 실패:', error);
        alert("책 추가 중 오류가 발생했습니다.");
      }
    },

    cancelAddBook() {
      this.showConfirmModal = false; // 모달 닫기
    },

    // 책장 검색 쪽수수
    changePage(page) {
      if (page < 1 || page > this.totalPages) return;
      this.currentPage = page;
    },

    setRegisterType(type) {
      this.registerType = type;
    },

    // 알라딘 도서 검색 API (제목 검색)
    async searchManual() {
      try {
        const response = await axios.get(`/api/books/search`, {
          params: { query: this.manualTitle },
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
        this.currentPage = 1; // 검색 후 페이지를 1로 초기화
      } catch (error) {
        console.error("책 검색 오류:", error);
      }
    },

    // 알라딘 도서 검색(ISBN)
    async searchISBN() {
      try {
        const response = await axios.get(`/api/books/search`, {
          params: { query: this.isbn },
        });

        if (response.data.books.length === 0) {
          alert("검색 결과가 없습니다.");
        }

        const book = response.data.books[0];
        this.searchResults = [{
          title: book.title,
          author: book.author,
          publisher: book.publisher,
          isbn: book.isbn,
          cover: book.cover,
        }];
        this.currentPage = 1; // 검색 후 페이지를 1로 초기화
      } catch (error) {
        console.error("ISBN 검색 오류:", error);
      }
    },

    openFileInput() {
      const fileInput = document.createElement('input');
      fileInput.type = 'file';
      fileInput.accept = 'image/*';  // 이미지 파일만 선택
      fileInput.click();
      
      fileInput.addEventListener('change', () => {
        const file = fileInput.files[0];
        if (file) {
          console.log("첨부된 파일:", file);
        }
      });
    },
    openCamera() {
      if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices.getUserMedia({ video: true })
          .then(() => {
            console.log("카메라가 열렸습니다.");
          })
          .catch((err) => {
            console.error("카메라 연결 실패:", err);
          });
      } else {
        alert("모바일에서만 지원됩니다.");
      }
    },

    // 현재 선택된 책장에 저장된 책들 중 삭제할 책 선택
    deleteBooksFromShelf() {
    const selectedBooks = this.books.filter(book => book.selected);

    if (selectedBooks.length === 0) {
      alert("삭제할 책을 선택해 주세요.");
      return;
    }

    if (confirm("선택한 책을 삭제하시겠습니까?")) {
      selectedBooks.forEach(book => {
        // 각 책 삭제 API 호출
        axios.delete(`/api/bookshelf/${this.currentBookshelf}/delete`, {
          params: { isbn: book.isbn } // ISBN을 통해 책 삭제
        }).then(response => {
          if (response.data.isSuccess) {
            alert(`${book.title}이(가) 삭제되었습니다.`);
            // 책장 목록 업데이트
            this.fetchBookshelves();
          } else {
            alert("책 삭제 실패: " + response.data.message);
          }
        }).catch(error => {
          console.error("책 삭제 중 오류 발생:", error);
        });
      });
    }
  }},
};
</script>

<style scoped>
.my-books {
  padding: 20px;
}

.bookshelf-header {
  margin-bottom: 20px;
}

.bookshelf-controls {
  display: flex;
  align-items: center;
}

.bookshelf-select {
  padding: 5px;
  margin-right: 10px;
  width: 300px;
}

.rename-button,
.add-bookshelf-button,
.delete-bookshelf-button,
.add-book-button,
.photo-registration-button {
  background-color: #ffa500;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 5px 10px;
  cursor: pointer;
  margin-right: 5px;
}

.add-bookshelf-button {
  background-color: #28a745;
  font-size: 12.5pt;
}

.delete-bookshelf-button {
  background-color: #dc3545;
  font-size: 10.5pt;
}

.add-book-button {
  background-color: #007bff; /* 책 등록 버튼 색 */
}

.photo-registration-button {
  background-color: #ff5722; /* 사진 등록 버튼 색 */
}

.bookshelf {
  background-color: #f8f9fa;
  padding: 20px;
  border-radius: 8px;
  border: 1px solid #ddd;
}

.book-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr); /* 5개씩 배치 */
  gap: 30px; /* 그리드 간의 간격 조정 */
}

.book-grid-container {
  margin-top: 20px; /* 버튼과 그리드 간의 간격 조정 */
}

.book-placeholder {
  width: 100%; /* 너비를 100%로 설정하여 그리드에 맞게 조정 */
  padding-top: 150%; /* 높이를 비율로 설정 (예: 1.5:1 비율) */
  background-color: #e9ecef;
  border: 1px solid #ddd;
  border-radius: 8px;
  position: relative;
  display: flex;
  justify-content: center; /* 가로 중앙 정렬 */
  align-items: center; /* 세로 중앙 정렬 */
  overflow: hidden; /* 자식 요소가 넘칠 경우 숨기기 */
}

.search-results ul {
  display: grid;
  grid-template-columns: repeat(2, 1fr); /* 2개씩 배치 */
  gap: 10px;
}

.search-book-item {
  display: flex;
  align-items: stretch;
  gap: 10px;
  border: 1px solid #ddd;
  padding: 10px;
  background: white;
  border-radius: 5px;
}

.book-cover {
  display: flex;
  align-items: stretch;
  width: 100px; /* 원하는 너비 설정 */
}

.book-cover img {
  width: 100%; /* 너비를 100%로 설정 */
  height: 100%; /* 높이를 100%로 설정 */
  object-fit: cover; /* 비율을 유지하면서 꽉 차게 설정 */
}

.book-info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  flex-grow: 1;
}

.book-title {
  font-weight: bold;
  font-size: 14px;
}

.book-author {
  font-size: 12px;
  color: gray;
}

.select-book-button {
  margin-top: 5px;
  padding: 5px 10px;
  background-color: #007bff;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 4px;
}

.select-book-button:hover {
  background-color: #0056b3;
}

/* 사이드바 스타일 */
.sidebar {
  position: fixed;
  top: 0;
  right: 0;
  width: 50vw;
  height: 100%;
  background-color: #fff;
  box-shadow: -2px 0 4px rgba(0, 0, 0, 0.1);
  z-index: 1000;
  padding: 25px;
}

.sidebar button {
  background-color: #FFA500; /* 주황색 */
  color: white; /* 글씨 색은 흰색 */
  border: none; /* 기본 border 제거 */
  border-radius: 4px; /* 버튼 모서리 둥글게 */
  padding: 8px 10px; /* 버튼 크기 조정 */
  cursor: pointer; /* 마우스 커서가 버튼에 올려지면 손가락 모양으로 변경 */
  margin-bottom: 20px; /* 버튼들 간의 간격 */
}

.rename-button:hover {
  background-color: #e68900; /* 어두운 주황색 */
}

.add-bookshelf-button:hover {
  background-color: #218838; /* 어두운 초록색 */
}

.delete-bookshelf-button:hover {
  background-color: #c82333; /* 어두운 빨간색 */
}

.add-book-button:hover {
  background-color: #0056b3; /* 어두운 파란색 */
}

.photo-registration-button:hover {
  background-color: #e64a19; /* 어두운 오렌지색 */
}

/* 파일 입력 스타일 */
.file-input,
.camera-button {
  margin-top: 15px;
  background-color: #007bff;
  color: white;
  border: none;
  padding: 8px 15px;
  border-radius: 4px;
  cursor: pointer;
}

.file-input:hover,
.camera-button:hover {
  background-color: #0056b3;
}

/* 사이드바 버튼 hover 효과 */
.sidebar button:hover {
  background-color: #e69500; /* 어두운 주황색 */
}

.rename-input {
  padding: 5px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.add-bookshelf-modal {
  position: fixed; /* 화면 중앙에 고정 */
  top: 50%; /* 수직 중앙 */
  left: 50%; /* 수평 중앙 */
  transform: translate(-50%, -50%); /* 정확한 중앙 정렬을 위해 변환 */
  width: 300px; /* 모달의 너비 */
  padding: 20px; /* 내부 여백 */
  background-color: white; /* 배경색 */
  border: 1px solid #ddd; /* 테두리 스타일 */
  border-radius: 8px; /* 모서리 둥글게 */
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2); /* 그림자 효과 */
  z-index: 1000; /* 다른 요소 위에 표시되도록 설정 */
}

.add-bookshelf-modal-content {
  display: flex; /* 플렉스 컨테이너로 설정 */
  flex-direction: column; /* 세로 방향으로 배치 */
}

.add-bookshelf-modal input {
  margin-bottom: 30px; /* 아래쪽 여백 */
  padding: 5px; /* 내부 여백 */
  border: 1px solid #ddd; /* 테두리 스타일 */
  border-radius: 4px; /* 모서리 둥글게 */
}

.create-bookshelf-button {
  background-color: #28a745; /* 버튼 색상 */
  color: white; /* 글자 색상 */
  border: none; /* 기본 테두리 제거 */
  padding: 8px; /* 내부 여백 */
  border-radius: 4px; /* 모서리 둥글게 */
  cursor: pointer; /* 마우스 커서 변경 */
  margin-bottom: 10px; /* 아래쪽 여백 */
}

.close-modal-button {
  background-color: #dc3545;
  color: white;
  border: none;
  padding: 8px;
  border-radius: 4px;
  cursor: pointer;
}

.add-bookshelf-modal input {
  margin-bottom: 30px; /* 아래쪽 여백 */
  padding: 5px; /* 내부 여백 */
  border: 1px solid #ddd; /* 테두리 스타일 */
  border-radius: 4px; /* 모서리 둥글게 */
}

.create-bookshelf-button {
  background-color: #28a745; /* 버튼 색상 */
  color: white; /* 글자 색상 */
  border: none; /* 기본 테두리 제거 */
  padding: 8px; /* 내부 여백 */
  border-radius: 4px; /* 모서리 둥글게 */
  cursor: pointer; /* 마우스 커서 변경 */
  margin-bottom: 10px; /* 아래쪽 여백 */
}

/* 추천받기 모달 스타일 */
.recommendation-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1001; /* 다른 요소 위에 표시 */
}

.recommendation-modal-content {
  background-color: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

/* 추천받기 모달 스타일 */
.recommendation-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1001; /* 다른 요소 위에 표시 */
}

.recommendation-modal-content {
  background-color: white;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
}

/* 추천 기준 버튼 스타일 */
.recommendation-options {
  display: flex;
  justify-content: center; /* 가운데 정렬 */
  gap: 10px; /* 버튼 간의 간격 조정 */
  margin-bottom: 20px; /* 아래쪽 여백 */
}

.recommend-books-button {
  background-color: #28a745; /* 추천받기 버튼 색상 */
  color: white;
  border: none;
  border-radius: 4px;
  padding: 10px;
  cursor: pointer;
}

.recommend-books-button:hover {
  background-color: #218838; /* 어두운 초록색 */
}

/* 취소 버튼 스타일 */
.close-modal-button {
  background-color: #dc3545; /* 취소 버튼 색상 */
  color: white;
  border: none;
  border-radius: 4px;
  padding: 10px;
  cursor: pointer;
  display: block; /* 블록으로 설정하여 중앙 정렬 */
  margin: 0 auto; /* 가운데 정렬 */
}

.close-modal-button:hover {
  background-color: #c82333; /* 어두운 빨간색 */
}

/* 편집 버튼 스타일 */
.edit-button {
  background-color: #007bff; /* 편집 버튼 색상 */
  color: white;
  border: none;
  border-radius: 4px;
  padding: 5px 10px;
  cursor: pointer;
}

.edit-button:hover {
  background-color: #0056b3; /* 어두운 파란색 */
}

/* 버튼을 나란히 배치 */
.sidebar .registration-options {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  margin-top: 20px;
}

.sidebar .registration-options button {
  padding: 12px 24px;
  border: 1px solid #ddd;
  border-radius: 6px;
  cursor: pointer;
  background-color: #ffffff;
  transition: background-color 0.3s, border-color 0.3s;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-grow: 1; /* 버튼들이 고르게 배치되도록 함 */
}

.sidebar .registration-options button:hover {
  background-color: #f5f5f5;
  border-color: #ccc;
}

/* active 버튼에 스타일 추가 */
.sidebar .registration-options button.active {
  background-color: #4caf50;
  color: white;
  border-color: #45a049;
}

/* 비활성화된 버튼 색상 */
.sidebar .registration-options button:not(.active):not(:disabled) {
  background-color: #f0f0f0;
  border-color: #ccc;
  color: #888; /* 연한 회색 */
}

/* 비활성화된 버튼 상태 */
.sidebar .registration-options button:disabled {
  background-color: #ddd;
  cursor: not-allowed;
  border-color: #bbb;
  color: #bbb; /* 연한 회색으로 글자 색상 */
}

/* input 및 버튼 스타일 */
.sidebar .manual-form input,
.sidebar .isbn-form input {
  width: 100%;
  padding: 12px;
  margin-top: 15px;
  border: 1px solid #ddd;
  border-radius: 6px;
}

.sidebar .file-upload-button,
.sidebar .camera-button {
  padding: 12px 24px;
  background-color: #2196f3;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.3s;
  display: block;
  width: 100%;
}

.sidebar .file-upload-button:hover,
.sidebar .camera-button:hover {
  background-color: #1976d2;
}

.sidebook-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* 한 줄에 3개씩 */
  gap: 16px; /* 책들 간 간격 */
  max-height: 400px; /* 사이드바 최대 높이 */
  overflow-y: auto; /* 스크롤 가능하게 */
  padding: 10px;
}

.sidebook-item {
  background-color: #f5f5f5;
  border: 1px solid #ddd;
  padding: 10px;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.sidebook-item.selected {
  background-color: #87ceeb; /* 선택된 책 색상 */
}

.sidebook-item:hover {
  background-color: #e0e0e0; /* 마우스 hover 시 색상 변화 */
}

.pagination {
  display: flex;
  justify-content: center;
  margin-top: 10px; /* 페이지네이션과 위쪽 요소 간의 간격을 조절 */
}

.pagination button {
  margin: 0 5px; /* 버튼 간의 간격을 조절 */
}

.confirm-modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  z-index: 1001; /* 사이드바보다 위에 표시 */
}

.confirm-modal-content {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.confirm-modal button {
  margin-top: 10px;
  padding: 8px 16px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.confirm-modal button:hover {
  background-color: #0056b3;
}

</style>
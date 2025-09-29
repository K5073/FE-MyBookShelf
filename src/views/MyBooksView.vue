<template>
  <div class="my-books">
    <!-- 책장 헤더 -->
    <div class="bookshelf-header">
      <div class="bookshelf-controls">
        <div v-if="isRenaming">
          <input v-model="newBookshelfName" @keyup.enter="renameBookshelf"
          class="rename-input" type="text" placeholder="새 책장 이름 입력"/>
        </div>
        <!-- 책장 리스트 -->
        <select v-else v-model="currentBookshelf" @change="selectBookshelf" class="bookshelf-select">
          <option value="null" disabled> ------- 책장을 추가해주세요 ------- </option>
          <option v-for="shelf in bookshelves" :key="shelf.bookshelfId" :value="shelf.bookshelfId">
            {{ [ shelf.bookshelfName ] }}
          </option>
        </select>
        <button @click="openAddBookshelfModal" class="add-bookshelf-button">+ 책장 생성</button>
        <button @click="deleteBookshelf" class="delete-bookshelf-button" :disabled="isNoBookshelf">🗑 책장 삭제</button>
        <button @click="openSidebar" class="add-book-button" :disabled="!currentBookshelf">도서 등록</button>
        <button @click="openRecommendationModal" class="recommend-button">도서 추천 받기</button>
        <button @click="toggleEditMode" class="edit-button">도서 삭제</button>
      </div>
    </div>

    <!-- 네모난 책장 폼 -->
    <div class="bookshelf">
      <div class="book-grid">
        <div v-for="(book, index) in currentBookshelfBooks" 
          :key="index" class="book-placeholder" @contextmenu.prevent="showContextMenu($event, book)">
          <div v-if="book.cover" class="bookshelfbook-cover">
            <img :src="book.cover || 'default-cover.jpg'" alt="책 표지" />
          </div>
          <div class="bookshelf-info">
            <div class="bookshelf-title">{{ truncateTitleBeforeSpecialChar(book.title) }}</div>
            <div class="bookshelf-author">{{ book.author.length > 20 ? book.author.slice(0, 20) + '...' : book.author }}</div>
            <div class="bookshelf-category">{{ book.categoryName }}</div>
            <button v-if="isEditing" @click="removeBook(book)" class="remove-book-button">-</button>
          </div>
        </div>
      </div>
      <!-- 컨텍스트 메뉴 -->
      <div v-if="contextMenuVisible" class="context-menu" :style="{ top: `${contextMenuY}px`, left: `${contextMenuX}px` }">
        <button @click="viewBookInfo()">책 상세보기</button>
        <button @click="removeBookFromContextMenu">책 삭제</button>
      </div>
    </div>

    <!-- 책장 추가 모달 -->
    <div v-if="isAddBookshelfModalOpen" class="add-bookshelf-modal">
      <div class="add-bookshelf-modal-content">
        <label for="new-bookshelf-name">책장 이름</label>
        <input type="text" id="new-bookshelf-name" v-model="newBookshelfNameForModal" placeholder="책장 이름 입력" />
        <button @click="addBookshelf" class="create-bookshelf-button">책장 생성하기</button>
        <button @click="closeAddBookshelfModal" class="close-modal-button">취소</button>
      </div>
    </div>

    <!-- 로딩 모달 -->
    <div v-if="isLoading" class="loading-modal">
      <div class="loading-modal-content">
        <div class="spinner"></div> <!-- 여기에 CSS로 애니메이션 -->
        <p>도서 추천 목록 불러오는 중... (약 1~2분 소요)</p>
        <button @click="cancelRecommendation" class="cancel-recommendation-button">추천 취소</button>
      </div>
    </div>

    <!-- 추천받기 모달 -->
    <div v-if="isRecommendationModalOpen" class="recommendation-modal">
      <div class="recommendation-modal-content">
        <h3>맞춤형 도서 추천받기</h3>
        
        <!-- 상단 추천 타입 선택 -->
        <div class="plus-recommendation-options"> 
          <button @click="setRecommendationType('preference')" :class="{ active: recommendationType === 'preference' }">내 취향 추천</button>
          <button @click="setRecommendationType('ai')" :class="{ active: recommendationType === 'ai' }">통합 추천</button>
          <button @click="setRecommendationType('survey')" :class="{ active: recommendationType === 'survey' }">설문 추천</button>
        </div>

        <!-- 취향 추천 -->
        <div v-if="recommendationType === 'preference'" class="recommendation-options">
          <p>내 취향(선호 장르)에 맞춘 도서를 추천받으시겠습니까?</p>
          <div class="button-container">
            <button @click="recommendPreference" class="recommend-books-button">추천받기</button>
            <button @click="closeRecommendationModal" class="close-modal-button">취소</button>
          </div>
        </div>

        <!-- AI 추천 -->
        <div v-if="recommendationType === 'ai'" class="recommendation-options">
          <p>AI 기반 키워드 분석을 포함한 정밀 추천을 받으시겠습니까?</p>
          <div class="button-container">
            <button @click="recommendAI" class="recommend-books-button">추천받기</button>
            <button @click="closeRecommendationModal" class="close-modal-button">취소</button>
          </div>
        </div>

        <!-- 설문 추천 -->
        <div v-if="recommendationType === 'survey'" class="recommendation-options survey-section">
          <p class="survey-title">{{ currentStep.title }}</p>

          <div class="progress-indicator">
            <span v-for="n in 3" :key="n" :class="{ active: step >= n }" class="dot"></span>
          </div>

          <div class="option-grid">
            <button
              v-for="(option, index) in currentStep.options"
              :key="index"
              @click="toggleSurveyOption(option, currentStepIndex)" 
              :class="{ selected: isSelected(option, currentStepIndex) }"
              class="survey-option-button"
            >
              {{ option.text }}
            </button>
          </div>
          <div class="button-container">
            <button v-if="step > 1" @click="prevSurveyStep" class="prev-button">이전</button>
            <button @click="nextSurveyStep" class="next-button">
              {{ step < 2 ? '다음' : '추천받기' }}
            </button>
            <button @click="closeRecommendationModal" class="close-modal-button">취소</button>
          </div>
        </div>
      </div>
    </div>

    <!-- 추가 평점 모달 -->
    <div v-if="isAdditionalRatingModalOpen" class="additional-rating-modal">
      <div class="additional-rating-modal-content">
        <div class="additional-rating-header">도서 추천 결과</div>
        <div class="rating-list">
          <div class="rating-item" v-for="book in recommendations" :key="book.isbn">
            <div class="rating-cover">
              <img :src="book.cover" alt="책 표지" />
            </div>
            <div class="rating-info">
              <div class="rating-title">{{ book.title.length > 30 ? book.title.slice(0, 30) + '...' : book.title }}</div>
              <div class="rating-author">저자: {{ book.author.length > 20 ? book.author.slice(0, 20) + '...' : book.author }}</div>
              <div class="rating-category">카테고리: {{ book.categoryName }}</div>
            </div>
            <div class="rating-save-button-area">
              <button @click="selectBook(book)" class="rating-save-button">등록</button>
            </div>
            <div>
              <div class="rating-details">
                <template v-if="recommendationType === 'preference'">
                  <div class="rating-score">가중평점: {{ book.weightedRatingScore?.toFixed(2) ?? 'N/A' }}</div>
                  <div class="rating-loan">대출 점수: {{ book.loanScore?.toFixed(2) ?? 'N/A' }}</div>
                  <div class="rating-best">베스트셀러 점수: {{ book.bestsellerScore }}</div>
                  <div class="rating-total">총점: {{ book.totalScore?.toFixed(2) ?? 'N/A' }}</div>
                </template>
                <template v-else>
                  <div class="rating-gerne">장르 유사 점수: {{ book.similarityScore }}</div>
                  <div class="rating-prefer">선호 장르 점수: {{ book.preferenceScore }}</div>
                  <div class="rating-keyword">키워드 점수: {{ book.keywordScore }}</div>
                  <div class="rating-score">가중평점: {{ book.weightedRatingScore?.toFixed(2) ?? 'N/A' }}</div>
                  <div class="rating-loan">대출 점수: {{ book.loanScore.toFixed(2) ?? 'N/A' }}</div>
                  <div class="rating-best">베스트셀러 점수: {{ book.bestsellerScore }}</div>
                  <div class="rating-total">총점: {{ book.totalScore?.toFixed(2) ?? 'N/A' }}</div>
                </template>
              </div>
            </div>
          </div>
        </div>
        <button @click="closeAdditionalRatingModal" class="close-rating-modal-button">닫기</button>
      </div>
    </div>

    <!-- 확인 모달 -->
    <div v-if="showConfirmModal" class="confirm-modal">
      <div class="confirm-modal-content">
        <p>'{{ selectedBook.title }}' 을 등록하시겠습니까?</p>
        <div class="confirm-modal-button-container">
          <button @click="confirmAddBook">예</button>
          <button @click="cancelAddBook">아니요</button>
        </div>
      </div>
    </div>

    <!-- 사이드바 -->
    <div v-if="isSidebarOpen" class="sidebar">
      <div class="sidebar-content">
        <button class="close-button" @click="closeSidebar">✖</button>
        <div class="registration-options">
          <button @click="setRegisterType('manual')" :class="{ active: registerType === 'manual' }">직접 등록</button>
          <button @click="setRegisterType('isbn')" :class="{ active: registerType === 'isbn' }">ISBN 등록</button>
        </div>

        <!-- 직접 등록 폼 -->
        <div v-if="registerType === 'manual'" class="manual-form">
          <input type="text" id="title" v-model="manualTitle" placeholder="책 제목 입력" />
          <button @click="searchManual">검색</button>
        </div>

        <!-- ISBN 등록 폼 -->
        <div v-if="registerType === 'isbn'" class="isbn-form">
          <input type="text" id="isbn" v-model="isbn" placeholder="ISBN 입력" />
          <button @click="searchISBN">검색</button>
        </div>

        <!-- 검색된 책들 -->
        <div v-if="searchResults.length" class="search-results">
          <h4>검색된 책들:</h4>
          <ul>
            <li v-for="(book, index) in paginatedResults" :key="index">
              <div class="search-book-item">
                <div class="sidebook-cover">
                  <img :src="book.cover" alt="책 표지" />
                </div>
                <div class="sidebook-info">
                  <p class="sidebook-title" :title="book.title">
                    {{ book.title.length > 25 ? book.title.slice(0, 25) + '...' : book.title }}
                  </p>
                  <p class="sidebook-author">{{ book.author }}</p>
                  <button @click="selectBook(book)" class="sideselect-book-button">선택</button>
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

        <!-- 확인 모달 -->
        <div v-if="showConfirmModal" class="confirm-modal">
          <div class="confirm-modal-content">
            <p>'{{ selectedBook.title }}' 을 저장하시겠습니까?</p>
            <div class="confirm-modal-button-container">
              <button @click="confirmAddBook">예</button>
              <button @click="cancelAddBook">아니요</button>
            </div>
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
      selectedBookshelf: null,
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
      isAdditionalRatingModalOpen: false,
      recommendationType: "preference", // 추천 타입
      searchResults: [], // 검색된 책 정보
      books: [], // 책 배열 초기화
      currentPage: 1, // 현재 페이지
      booksPerPage: 6, // 페이지당 책 개수
      showConfirmModal: false, // 책 선택버튼 누르면 뜨는 창
      isEditing: false,  // 책장 편집 모드 상태 추가
      contextMenuVisible: false, // 컨텍스트 메뉴 표시 여부
      contextMenuX: 0, // 컨텍스트 메뉴 X 좌표
      contextMenuY: 0, // 컨텍스트 메뉴 Y 좌표
      selectedBook: null, // 선택된 책
      recommendations: [], // 추천받은 책 목록
      isBooksModalOpen: false, // 책 추천 결과 모달 상태
      isLoading: false,
      loadingMessage: "도서 추천 목록 불러오는 중... (약 1~2분 소요)", // 로딩 모달 멘트
      abortController: null,
      step: 1,
      selectedSurveyOptions: [],
      surveySteps: [
        {
          title: '1. 어떤 장르를 원하시나요?',
          options: [
            { text: '책장과 비슷한 장르의 책이었으면 좋겠어요', value: 'genreSimilarity' },
            { text: '선호 장르 위주로 추천 받고 싶어요', value: 'preferredGenre' },
          ],
        },
        {
          title: '2. 인기도나 대중성은 어떤가요?',
          options: [
            { text: '베스트셀러인 책 위주로 추천 받고 싶어요', value: 'bestseller' },
            { text: '평점이 높은 책 위주로 추천 받고 싶어요', value: 'highRating' },
          ],
        },
      ],
    };
  },

  created() {
    this.fetchBookshelves();
  },

  // 마우스 클릭 시 메뉴를 닫기 위해 이벤트 추가
  mounted() {
    document.addEventListener('click', this.closeContextMenu);
  },
  beforeUmount() {
    document.removeEventListener('click', this.closeContextMenu);
  },

  computed: {
    currentStep() {
      return this.surveySteps[this.step - 1];
    },

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
    toggleSurveyOption(option, stepIndex) {
      // 해당 질문(stepIndex)에 대한 기존 선택 제거
      this.selectedSurveyOptions = this.selectedSurveyOptions.filter(
        o => o.stepIndex !== stepIndex
      );

      // 같은 걸 다시 클릭한 경우 → 선택 취소 허용
      const alreadySelected = this.selectedSurveyOptions.find(
        o => o.value === option.value && o.stepIndex === stepIndex
      );

      if (!alreadySelected) {
        this.selectedSurveyOptions.push({
          ...option,
          stepIndex
        });
      }
    },
    isSelected(option, stepIndex) {
      return this.selectedSurveyOptions.some(
        o => o.value === option.value && o.stepIndex === stepIndex
      );
    },
    nextSurveyStep() {
      if (this.step < this.surveySteps.length) {
        this.step++;
      } else {
        this.submitSurveyRecommendation(); // API 호출
      }
    },
    prevSurveyStep() {
      if (this.step > 1) {
        this.step--;
      }
    },

    recommendSurvey() {
      console.log("설문 결과:", this.selectedSurveyOptions);
      this.closeRecommendationModal();
    },

    submitSharedBookshelf(postData) {
      // 실제 게시물 등록 API 호출
      axios.post('/api/posts', {
        title: postData.title,
        content: postData.content,
        isAnonymous: postData.isAnonymous,
        boardType: postData.boardType,
      }).then((res) => {
        if (res.data.isSuccess) {
          alert('게시물이 등록되었습니다!');
          this.showShareModal = false;
        } else {
          alert('게시물 등록 실패: ' + res.data.message);
        }
      }).catch((error) => {
        console.error('게시물 등록 중 오류:', error);
        alert('오류 발생');
      });
    },

    cancelRecommendation() {
      if (this.abortController) {
        this.abortController.abort(); // ✅ 요청 취소
        this.isLoading = false;
        this.abortController = null; // 정리
      }
    },

    closeBooksModal() {
      this.isBooksModalOpen = false; // 책 추천 결과 모달 닫기
    },
    
    truncateTitleBeforeSpecialChar(title) {
    // 특정 특수 기호가 나타나는 위치를 찾음
    const index = title.search(/[-:/]/); // '-', ':', '/' 중 첫 번째 문자의 인덱스

    // 특수 기호가 없으면 전체 제목을 반환하고, 있으면 그 이전까지 반환
    return index === -1 ? title : title.slice(0, index);
    },

    // 오른쪽 클릭 시 컨텍스트 메뉴 표시
    showContextMenu(event, book) {
      this.selectedBook = book; // 선택된 책 저장
      this.menuX = event.pageX + 'px';
      this.menuY = event.pageY + 'px';
      this.contextMenuVisible = true; // 메뉴 표시
    },

    // 책 정보 뷰로 이동
    viewBookInfo() {
      if (this.selectedBook && this.selectedBook.isbn) {
        this.$router.push({
          name: 'BookDetails',
          params: { isbn: this.selectedBook.isbn } 
        });
      } else {
        console.error('isbn이 존재하지 않습니다:', this.selectedBook);
      }
      this.contextMenuVisible = false;
    },

    // 클릭 외부 시 메뉴 닫기
    closeContextMenu() {
      this.contextMenuVisible = false;
    },
    
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

    toggleRenameMode() {
      if (this.currentBookshelf === null) {
        alert("책장을 먼저 선택해주세요.");
        return;
      }

      if (this.isRenaming) {
        this.renameBookshelf();  // 저장만 진행
      } else {
        const shelf = this.bookshelves.find(shelf => shelf.bookshelfId === this.currentBookshelf);
        if (shelf) {
          this.newBookshelfName = shelf.bookshelfName;
        }
        this.isRenaming = true; // 이름 변경 모드 시작
      }
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

    closeAdditionalRatingModal(){
      this.isAdditionalRatingModalOpen = false;
    },

    setRecommendationType(type) {
      this.recommendationType = type;
    },

    // 내 취향 기반 추천 (책장 없이)
    async recommendPreference() {
      const userStr = localStorage.getItem('user');
      const user = userStr ? JSON.parse(userStr) : null;
      const userId = user?.userId;

      if (!userId) {
        console.error("userId가 유효하지 않습니다.");
        return;
      }

      this.isLoading = true;

      if (this.abortController) {
        this.abortController.abort();
      }
      this.abortController = new AbortController();

      try {
        const response = await axios.get(`/api/recommend/${userId}/genre`, {
          signal: this.abortController.signal,
        });

        if (response.data.isSuccess) {
          this.recommendations = response.data.result;
          this.isAdditionalRatingModalOpen = true; 
          this.closeRecommendationModal();
        } else {
          console.error("추천받기 실패:", response.data.message);
          alert("추천받기 실패: " + response.data.message);
        }
      } catch (error) {
        if (error.name === "CanceledError" || error.code === "ERR_CANCELED") {
          console.log("추천 요청이 취소되었습니다.");
        } else {
          console.error("추천받기 오류:", error);
          alert("추천 중 오류가 발생했습니다.");
        }
      } finally {
        this.isLoading = false;
      }
    },

    // AI 기반 키워드 정밀 추천 API
    async recommendAI() {
      this.isLoading = true;

      if (this.abortController) {
        this.abortController.abort();
      }
      this.abortController = new AbortController();

      try {
        const response = await axios.get(`/api/recommend/total/${this.selectedBookshelf}`, {
          signal: this.abortController.signal,
        });

        if (response.data.isSuccess) {
          this.recommendations = response.data.result;
          this.isAdditionalRatingModalOpen = true; 
          this.closeRecommendationModal();
        } else {
          console.error("추천받기 실패:", response.data.message);
          alert("추천받기 실패: " + response.data.message);
        }
      } catch (error) {
        if (error.name === "CanceledError" || error.code === "ERR_CANCELED") {
          console.log("추천 요청이 취소되었습니다.");
        } else {
          console.error("추천받기 오류:", error);
          alert("추천 중 오류가 발생했습니다.");
        }
      } finally {
        this.isLoading = false;
      }
    },

    // 설문 추천
    async submitSurveyRecommendation() {
      if (!this.currentBookshelf) {
        alert("책장을 먼저 선택해주세요.");
        return;
      }

      const bookshelfId = Number(this.currentBookshelf);

      // 선택한 설문 옵션을 boolean 값으로 변환
      const payload = {
        genreSimilarity: this.selectedSurveyOptions.some(opt => opt.value === 'genreSimilarity'),
        preferredGenre: this.selectedSurveyOptions.some(opt => opt.value === 'preferredGenre'),
        bestseller: this.selectedSurveyOptions.some(opt => opt.value === 'bestseller'),
        highRating: this.selectedSurveyOptions.some(opt => opt.value === 'highRating'),
      };

      console.log("설문 요청 payload:", payload);

      this.isLoading = true;

      try {
        const response = await axios.post(`/api/recommend/survey/${bookshelfId}`, payload);

        if (response.data.isSuccess) {
          this.recommendations = response.data.result;
          this.recommendationType = 'result';
          this.isAdditionalRatingModalOpen = true; 
          this.closeRecommendationModal();
        } else {
          console.error("추천 실패:", response.data.message);
          alert("추천 실패: " + response.data.message);
        }
      } catch (error) {
        console.error("추천 API 호출 실패:", error);
        alert("추천 중 오류가 발생했습니다.");
      } finally {
        this.isLoading = false;
      }
    },

    // 검색된 책을 책장에 넣는 작업
    async selectBook(book) {
      this.selectedBook = book; // 선택한 책 정보를 저장
      this.showConfirmModal = true; // 모달 표시
    },

    async selectRating(book) {
      this.selectedRating = book;
      this.showConfirmModal = true;
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
        return;
      }

      try {
        // API 요청: 선택한 책의 ISBN을 이용해 책장에 추가
        axios.post(`/api/bookshelf/${this.currentBookshelf}/register`, null, {
          params: { isbn13: this.selectedBook.isbn } // this.selectedBook 사용
        }).then(() => {
          this.fetchBookshelves(); 
          alert(`'${this.selectedBook.title}' 책이 책장에 추가되었습니다.`);
          this.showConfirmModal = false; // 모달을 닫습니다.
        });
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
    
    // 책장 편집 모드 토글
    toggleEditMode() {
      this.isEditing = !this.isEditing;
    },

    // 책 삭제 메서드
    async removeBook(book) {
      if (confirm(`'${book.title}' 책을 삭제하시겠습니까?`)) {
        try {
          const response = await axios.delete(`/api/bookshelf/delete/book/${this.selectedBookshelf}/${book.bookId}`, {
          });

          if (response.data.isSuccess) {
            alert(`${book.title}이(가) 삭제되었습니다.`);
            // 책장 목록 업데이트
            this.fetchBookshelves();
          } else {
            alert("책 삭제 실패: " + response.data.message);
          }
        } catch (error) {
          console.error("책 삭제 중 오류 발생:", error);
        }
      }
    },

    // 컨텍스트메뉴에서 책 삭제
    async removeBookFromContextMenu() {
      if (this.selectedBook && confirm(`'${this.selectedBook.title}' 책을 삭제하시겠습니까?`)) {
        try {
          const response = await axios.delete(`/api/bookshelf/delete/book/${this.selectedBookshelf}/${this.selectedBook.bookId}`);

          if (response.data.isSuccess) {
            alert(`${this.selectedBook.title}이(가) 삭제되었습니다.`);
            // 책장 목록 업데이트
            this.fetchBookshelves();
          } else {
            alert("책 삭제 실패: " + response.data.message);
          }
        } catch (error) {
          console.error("책 삭제 중 오류 발생:", error);
        }
      }
    },

  }
};
</script>

<style src="../css/mybooks.css"></style>
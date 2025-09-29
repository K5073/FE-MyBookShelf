<template>
  <div v-if="book && book.cover" class="book-result">
    <div class="book-info">
      <div class="book-cover-wrapper">
        <img :src="book.cover" alt="책 표지" class="book-cover" />
      </div>
      <div class="book-details">
        <h2 class="book-title">{{ book.title }}</h2>
        <p class="book-author"><strong>저자:</strong> {{ book.author }}</p>
        <p class="book-publisher"><strong>출판사:</strong> {{ book.publisher }}</p>
        <p class="book-category"><strong>카테고리:</strong> {{ book.categoryName }}</p>
        <p class="book-review-rank"><strong>평점:</strong> {{ book.customerReviewRank }}</p>
      </div>
    </div>

    <!-- 책 설명 -->
    <div class="book-summary">
      <h3>책 설명</h3>
      <p>{{ book.description }}</p>
    </div>

    <!-- 독후감 작성 -->
    <div class="report-section">
      <h3>독후감 작성</h3>
      <div class="report-form">
        <div class="form-left">
          <input v-model="reportTitle" type="text" placeholder="제목을 입력하세요" class="input-title" />
          <textarea v-model="reportContent" placeholder="내용을 입력하세요" class="input-content"></textarea>
        </div>
        <div class="form-right">
          <button @click="submitReport" class="submit-button">작성</button>
        </div>
      </div>
    </div>

    <!-- 독후감 목록 -->
    <div class="report-list">
      <h3>등록된 독후감</h3>
      <div v-if="reports.length">
        <div v-for="report in reports" :key="report.id" class="report-item">
          <p class="report-title">📌 {{ report.title }}</p>
          <p class="report-content">“{{ report.content }}”</p>
          <p class="report-meta">- {{ report.userNickname }} | {{ new Date(report.createdAt).toLocaleDateString() }}</p>
        </div>
      </div>
      <div v-else>
        <p>아직 등록된 독후감이 없습니다.</p>
      </div>
    </div>
  </div>

  <div v-else>
    <p>📚 책 정보를 불러오는 중입니다...</p>
  </div>
</template>

<script>
import axios from 'axios';
axios.defaults.baseURL = 'http://localhost:8081'; // API 기본 주소

export default {
  name: "BookDetails",
  data() {
    return {
      book: null,
      reportTitle: "",
      reportContent: "",
      reports: []
    };
  },

  created() {
    this.fetchBookInfo();
  },

  methods: {
    async fetchBookInfo() {
      const isbn = this.$route.params.isbn;
      console.log("📘 요청할 ISBN:", isbn);

      try {
        const response = await axios.get(`/api/book/${isbn}`);
        console.log("📦 응답:", response.data);

        // 응답이 성공이고, book 데이터가 있을 때만 할당
        if (response.data?.isSuccess && response.data?.result) {
          this.book = response.data.result;
        } else {
          console.warn("❗ 책 정보를 찾을 수 없습니다:", response.data);
          this.book = null;
        }
      } catch (error) {
        console.error("❗ 책 정보를 불러오는 중 오류 발생:", error);
        this.book = null;
      }
    },

    async fetchReports(bookId) {
      try {
        const res = await axios.get(`/api/reports/book/${bookId}`);
        this.reports = res.data || [];
      } catch (e) {
        console.error("❗ 독후감 불러오기 실패:", e);
      }
    },

    async submitReport() {
      if (!this.reportTitle || !this.reportContent) {
        alert("제목과 내용을 모두 입력해주세요.");
        return;
      }

      try {
        const payload = {
          bookId: this.book.id,
          title: this.reportTitle,
          content: this.reportContent,
        };
        await axios.post("/api/reports", payload);
        alert("독후감이 등록되었습니다.");
        this.reportTitle = "";
        this.reportContent = "";
        this.fetchReports(this.book.id); // 새로고침
      } catch (e) {
        console.error("❗ 독후감 등록 실패:", e);
        alert("등록 중 오류가 발생했습니다.");
      }
    }
  },
};
</script>

<style>
.book-result {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  max-width: 900px;
  margin: auto;
  background-color: #f9f9f9;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.book-info {
  display: flex;
  margin-bottom: 20px;
}

.book-cover-wrapper {
  width: 300px; 
  height: 400px; 
  margin-right: 20px;
  flex-shrink: 0;
}

.book-cover {
  width: 100%;
  max-width: 250px; 
  /* width: 250px; */
  height: auto;
  border-radius: 5px;
  object-fit: cover;
}

.book-details {
  max-width: 600px;
}

.book-title {
  font-size: 24px;
  font-weight: bold;
  margin: 0;
}

.book-author,
.book-publisher,
.book-publication-date,
.book-category,
.book-source,
.book-review-rank {
  margin: 5px 0;
  font-size: 16px;
}

.book-summary {
  width: 100%;
  background-color: #fff;
  padding: 15px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.book-summary h3 {
  margin: 0 0 10px;
}

.book-rating {
  width: 100%;
  margin: 20px 0;
}

.report-section {
  width: 100%;
  margin-top: 30px;
  background-color: #fff;
  padding: 15px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.report-form {
  display: flex;
  gap: 20px;
}

.form-left {
  flex: 2;
  display: flex;
  flex-direction: column;
}

.input-title {
  font-size: 16px;
  padding: 10px;
  margin-bottom: 10px;
}

.input-content {
  height: 120px;
  font-size: 14px;
  padding: 10px;
  resize: vertical;
}

.form-right {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
}

.submit-button {
  padding: 10px 20px;
  font-size: 16px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.report-list {
  width: 100%;
  margin-top: 20px;
  background-color: #fff;
  padding: 15px;
  border-radius: 5px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.report-item {
  margin-bottom: 15px;
  border-bottom: 1px solid #eee;
  padding-bottom: 10px;
}

.report-title {
  font-weight: bold;
  margin-bottom: 5px;
}

.report-content {
  font-style: italic;
}

.report-meta {
  font-size: 12px;
  color: gray;
}

</style>
<template>
    <div class="post-editor">
      <h3>게시글 작성</h3>
  
      <!-- 게시판 선택 -->
      <select v-model="selectedBoard">
        <option disabled value="">게시판 선택</option>
        <option v-for="board in boards" :key="board" :value="board">
          {{ board }}
        </option>
      </select>
  
      <label>
        <input type="checkbox" v-model="isAnonymous" />
        익명으로 작성
      </label>

      <!-- 책장 선택 드롭다운 -->
      <select v-model="selectedShelfId" @change="onBookshelfChange">
        <option disabled value="">공유할 책장을 선택하세요</option>
        <option v-for="shelf in bookshelves" :key="shelf.bookshelfId" :value="shelf.bookshelfId">
          {{ shelf.bookshelfName }}
        </option>
      </select>

      <!-- 제목 입력 -->
      <input v-model="title" type="text" placeholder="제목" />
  
      <!-- 내용 입력 -->
      <textarea v-model="content" placeholder="내용"></textarea>
  
      <!-- 버튼 -->
      <div class="buttons">
        <button @click="submit">등록</button>
        <button @click="$emit('cancel')">취소</button>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'PostEditor',
    data() {
      return {
        boards: ['홍보 게시판', '자유 게시판', '정보 게시판'],
        selectedBoard: this.initialBoard,
        title: this.initialTitle || '',
        content: this.initialContent || '',
        isAnonymous: false,
        selectedShelfId: this.defaultShelfId || null,
      };
    },
    props: {
      initialTitle: String,
      initialContent: String,
      initialBoard: String,
      bookshelves: Array,
      defaultShelfId: Number
    },
    methods: {
      submit() {
        if (!this.selectedBoard || !this.title.trim() || !this.content.trim()) {
          return alert('게시판, 제목, 내용을 모두 입력해주세요.');
        }
        this.$emit('submit', {
          boardType: this.selectedBoard,
          title: this.title,
          isAnonymous: this.isAnonymous,
          content: this.content,
          bookshelfId: this.selectedShelfId,
          createdAt: new Date().toISOString()
        });
        this.title = '';
        this.content = '';
        this.selectedBoard = '';
      },

      onBookshelfChange() {
        const shelf = this.bookshelves.find(s => s.bookshelfId === this.selectedShelfId);
        if (!shelf) return;

        this.title = `[책장 공유] ${shelf.bookshelfName}`;
        this.content = shelf.book.map(book => `📘 ${book.title} - ${book.author}`).join('\n');
      },
    },
  };
  </script>
  
  <style scoped>
  .post-editor {
    position: fixed;
    top: 20%;
    left: 25%;
    width: 50%;
    background: white;
    border: 1px solid #ddd;
    padding: 20px;
    box-shadow: 0 0 10px #aaa;
    z-index: 100;
  }
  select, input, textarea {
    width: 100%;
    margin-bottom: 10px;
    padding: 8px;
  }
  textarea {
    height: 120px;
  }
  .buttons {
    display: flex;
    gap: 10px;
  }
  button {
    padding: 6px 12px;
  }
  </style>
  
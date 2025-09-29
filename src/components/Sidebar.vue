<template>
  <div class="sidebar">
    <ul>
      <li v-for="board in boards" :key="board" @click="$emit('select-board', board)">
        {{ board }}
      </li>
    </ul>
  </div>
 </template>
  
 <script>
  export default {
    name: 'Sidebar_com',
    data() {
      return {
        boards: ['전체 게시판', '홍보 게시판', '정보 게시판', '자유 게시판'],
      };
    },
  };
</script>
  
<style scoped>
/* 사이드바 컨테이너 */
.sidebar {
  width: 240px;
  background: linear-gradient(180deg, #ffffff, #f4f4f4);
  padding: 24px;
  height: calc(100vh - 100px); /* 상단 메뉴바 높이 고려 */
  box-sizing: border-box;
  border-right: 1px solid #ddd;
  margin-top: 100px; /* 상단 메뉴바 만큼 내려줌 */
  box-shadow: 2px 0 8px rgba(0, 0, 0, 0.05);
  font-family: 'Helvetica Neue', sans-serif;
}

/* 리스트 스타일 */
.sidebar ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

/* 각 사이드바 항목 */
.sidebar li {
  cursor: pointer;
  padding: 12px 16px;
  border-radius: 8px;
  margin-bottom: 8px;
  transition: background-color 0.3s ease, transform 0.2s ease;
  color: #333;
  font-size: 16px;
}

/* hover 효과: 배경색과 텍스트 색, 약간의 오른쪽 이동 */
.sidebar li:hover {
  background-color: #ff9900;
  color: #fff;
  transform: translateX(4px);
}

/* 활성 상태 (필요 시 클래스 추가) */
.sidebar li.active {
  background-color: #ff9900;
  color: #fff;
  box-shadow: inset 0 0 8px rgba(0, 0, 0, 0.15);
}

</style>
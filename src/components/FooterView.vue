<template>
  <footer class="site-footer">
    <div class="container">
      <h2 class="footer-heading"><a href="#" class="logo">MyBookShelf</a></h2>
      <p class="menu">
        <router-link to="/" class="item" exact-active-class="active">Home</router-link>
        <router-link to="/guidelines" class="item" exact-active-class="active">Guideline</router-link>
        <router-link to="/community" class="item" exact-active-class="active">Community</router-link>
        <router-link to="/mybooks" class="item" exact-active-class="active">BookShelf</router-link>
      </p>
    </div>
  </footer>
</template>

<script setup>
// 특별한 스크립트 없음
</script>

<style scoped>
.site-footer {
  background-color: rgba(0, 0, 0, 0.904);
  color: white;
  text-align: center;
  padding: 30px 20px;
}

.site-footer a {
  color: white;
  text-decoration: none;
  margin: 0 10px;
}

.site-footer a:hover {
  text-decoration: underline;
}

.footer-heading {
  margin-bottom: 20px;
}

.menu {
  margin-bottom: 20px;
}

.menu a {
  display: inline-block;
  margin: 0 8px;
  font-weight: normal;
}
</style>
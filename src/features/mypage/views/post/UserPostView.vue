<script setup>
import { ref, computed, onMounted } from 'vue'
import { useAuthStore } from '@/stores/auth'
import MyPageHeader from '@/features/mypage/components/common/MyPageHeader.vue'
import { fetchUserBoard, fetchUserComment } from '@/features/mypage/api.js'
import UserPostList from '@/features/mypage/components/post/UserPostList.vue'
import UserCommentList from '@/features/mypage/components/post/UserCommentList.vue'
import PaginationBar from '@/components/common/PaginationBar.vue'

const tab = ref('post')
const boardList = ref([])
const commentList = ref([])
const currentPage = ref(1)
const pageSize = 20

const authStore = useAuthStore()
const accessToken = authStore.accessToken

const fetchData = async () => {
  if (!accessToken) return
  try {
    const [boardRes, commentRes] = await Promise.all([
      fetchUserBoard(accessToken),
      fetchUserComment(accessToken)
    ])
    boardList.value = boardRes.data.data.boardList
    commentList.value = commentRes.data.data.commentList
  } catch (err) {
    console.error('마이페이지 게시글/댓글 조회 실패:', err)
  }
}

onMounted(fetchData)

const changeTab = (type) => {
  tab.value = type
  currentPage.value = 1
}

const currentItems = computed(() => {
  const items = tab.value === 'post' ? boardList.value : commentList.value
  const start = (currentPage.value - 1) * pageSize
  return items.slice(start, start + pageSize)
})

const totalPages = computed(() => {
  const total = tab.value === 'post' ? boardList.value.length : commentList.value.length
  return Math.ceil(total / pageSize)
})
</script>

<template>
  <MyPageHeader />
  <div class="user-post-view">
    <h2 class="text-center fw-bold fs-3 mb-5">
      {{ tab === 'post' ? '내가 쓴 동행글' : '내가 쓴 댓글' }}
    </h2>

    <div class="tab-buttons">
      <button :class="{ active: tab === 'post' }" @click="changeTab('post')">게시글</button>
      <button :class="{ active: tab === 'comment' }" @click="changeTab('comment')">댓글</button>
    </div>

    <UserPostList v-if="tab === 'post'" :items="currentItems" />
    <UserCommentList v-else :items="currentItems" />

    <div class="pagination-bar">
      <PaginationBar
        :current-page="currentPage"
        :total-pages="totalPages"
        @update:page="(p) => (currentPage.value = p)"
      />
    </div>
  </div>
</template>


<style scoped>
.user-post-view {
  background: #fdfbf5;
  padding: 32px 60px;
}

.tab-buttons {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-bottom: 10px;
}

.tab-buttons button {
  padding: 6px 14px;
  font-size: 0.875rem;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  background-color: #eee;
  color: #555;
}

.tab-buttons .active {
  background-color: #333;
  color: white;
}

.user-post-view {
  background: #fdfbf5;
  padding: 32px 60px;
}

.tab-buttons {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-bottom: 10px;
}

.tab-buttons button {
  padding: 6px 14px;
  font-size: 0.875rem;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  background-color: #eee;
  color: #555;
}

.tab-buttons .active {
  background-color: #333;
  color: white;
}

.pagination-bar {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}
</style>

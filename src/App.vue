<template>
  <div id="app" class="w-full min-h-screen bg-gray-100">
    <div class="grid grid-cols-1 gap-4 p-4 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4">
      <div
        v-for="bookmark in bookmarks"
        :key="bookmark.id"
        class="relative p-4 overflow-hidden bg-white rounded-lg shadow-md hover:bg-gray-50 group"
      >
        <div class="flex flex-col">
          <a
            :href="bookmark.url"
            target="_blank"
            class="block text-blue-600 truncate hover:text-blue-800"
            :title="bookmark.title ? bookmark.title : bookmark.url"
          >
            {{ bookmark.title ? bookmark.title : bookmark.url }}
          </a>
          <span class="text-xs text-gray-500 truncate" :title="bookmark.url">{{
            bookmark.url
          }}</span>
        </div>
      </div>
    </div>
    <!-- 返回顶部按钮 -->
    <BackToTop />
    <!-- 导出按钮 -->
    <div class="fixed z-10 bottom-8 right-4">
      <button
        @click="exportBookmarks"
        class="p-3 text-white transition-colors duration-300 bg-green-500 shadow-md hover:bg-green-600"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          class="w-6 h-6"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"
          />
        </svg>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import BackToTop from './components/BackToTop.vue'

// 创建一个响应式数组来存储书签
const bookmarks = ref([])

// 组件挂载后获取书签
onMounted(() => {
  getBookmarks()
})

/* 导出书签数据为 JSON 文件 */
function exportBookmarks() {
  if (bookmarks.value.length === 0) {
    alert('没有书签数据')
    return
  }
  const bookmarksJson = JSON.stringify(bookmarks.value, null, 2)
  const blob = new Blob([bookmarksJson], { type: 'application/json' })
  const url = URL.createObjectURL(blob)

  const a = document.createElement('a')
  a.href = url
  a.download = 'bookmarks.json'
  document.body.appendChild(a)
  a.click()
  document.body.removeChild(a)
  URL.revokeObjectURL(url)
}

/* 获取所有书签 */
function getBookmarks() {
  // 使用 Chrome 书签 API 获取书签树结构
  chrome.bookmarks.getTree((bookmarkTreeNodes) => {
    bookmarks.value = flattenBookmarks(bookmarkTreeNodes)
  })
}

/* 将书签树扁平化为一维数组 */
function flattenBookmarks(bookmarkNodes) {
  let flattened = []
  for (let node of bookmarkNodes) {
    if (node.children) {
      // 如果节点有子节点，递归处理
      flattened = flattened.concat(flattenBookmarks(node.children))
    } else if (node.url) {
      // 如果节点是叶子节点且有 URL，添加到结果数组
      flattened.push(node)
    }
  }
  return flattened
}

// /* 删除指定 ID 的书签 */
// function deleteBookmark(id) {
//   if (confirm('确定要删除这个书签吗？')) {
//     chrome.bookmarks.remove(id, () => {
//       // 删除成功后重新获取书签列表
//       getBookmarks()
//     })
//   }
// }
</script>

<style scoped>
ul {
  list-style-type: none;
  padding: 0;
}
li {
  margin-bottom: 10px;
}
a {
  text-decoration: none;
  color: #2c3e50;
}
button {
  margin-left: 10px;
}
</style>

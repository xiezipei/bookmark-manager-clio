<template>
  <div id="app" class="min-h-screen w-full bg-gray-100">
    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4 p-4">
      <div
        v-for="bookmark in bookmarks"
        :key="bookmark.id"
        class="bg-white rounded-lg shadow-md overflow-hidden p-4 hover:bg-gray-50"
      >
        <a
          :href="bookmark.url"
          target="_blank"
          class="text-blue-600 hover:text-blue-800 truncate block mb-2"
        >
          {{ bookmark.title ? bookmark.title : bookmark.url }}
        </a>
        <div class="relative group">
          <div
            class="invisible group-hover:visible absolute inset-0 flex items-center justify-center"
          >
            <button
              @click="deleteBookmark(bookmark.id)"
              class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded transition-colors duration-300"
            >
              删除
            </button>
          </div>
        </div>
      </div>
    </div>
    <!-- 添加返回顶部按钮 -->
    <BackToTop />
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

/* 删除指定 ID 的书签 */
function deleteBookmark(id) {
  chrome.bookmarks.remove(id, () => {
    // 删除成功后重新获取书签列表
    getBookmarks()
  })
}
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

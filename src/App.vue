<script setup>
import { ref, computed, onMounted } from 'vue'

// 状态
const questions = ref([])
const currentView = ref('list') // list, detail, ask
const currentQuestion = ref(null)
const searchQuery = ref('')

// 从 localStorage 加载数据
const loadData = () => {
  const saved = localStorage.getItem('career-qa-questions')
  if (saved) {
    questions.value = JSON.parse(saved)
  } else {
    // 初始示例数据
    questions.value = [
      {
        id: 1,
        title: '30岁转行做程序员还来得及吗？',
        content: '我做了5年销售，现在想转行做前端开发，想问问大家的建议。',
        author: '迷茫的打工人',
        tags: ['转行', '程序员', '前端'],
        answers: [
          { id: 1, content: '完全来得及！我32岁转行，现在在阿里做前端，分享一下我的经验...', author: '过来人老王', isPaid: false },
          { id: 2, content: '这是我的付费回答，包含详细的转行路径和学习资源...', author: '职业规划师小李', isPaid: true, price: 19.9, paid: false }
        ],
        createTime: Date.now() - 3600000,
        views: 256
      },
      {
        id: 2,
        title: '如何平衡工作和考证学习？',
        content: '想考PMP证书，但每天下班都很累，求建议。',
        author: '考证困难户',
        tags: ['考证', '时间管理', 'PMP'],
        answers: [
          { id: 1, content: '建议早起学习，每天30-60分钟，坚持3个月就够了', author: '早起达人', isPaid: false }
        ],
        createTime: Date.now() - 7200000,
        views: 128
      }
    ]
    saveData()
  }
}

// 保存到 localStorage
const saveData = () => {
  localStorage.setItem('career-qa-questions', JSON.stringify(questions.value))
}

// 过滤问题
const filteredQuestions = computed(() => {
  if (!searchQuery.value) return questions.value
  const query = searchQuery.value.toLowerCase()
  return questions.value.filter(q =>
    q.title.toLowerCase().includes(query) ||
    q.tags.some(t => t.toLowerCase().includes(query))
  )
})

// 新问题表单
const newQuestion = ref({
  title: '',
  content: '',
  tags: ''
})

// 提交问题
const submitQuestion = () => {
  if (!newQuestion.value.title.trim()) return

  const question = {
    id: Date.now(),
    title: newQuestion.value.title,
    content: newQuestion.value.content,
    author: '匿名用户',
    tags: newQuestion.value.tags.split(',').map(t => t.trim()).filter(t => t),
    answers: [],
    createTime: Date.now(),
    views: 0
  }

  questions.value.unshift(question)
  saveData()

  newQuestion.value = { title: '', content: '', tags: '' }
  currentView.value = 'list'
}

// 新回答
const newAnswer = ref('')

// 提交回答
const submitAnswer = () => {
  if (!newAnswer.value.trim() || !currentQuestion.value) return

  currentQuestion.value.answers.push({
    id: Date.now(),
    content: newAnswer.value,
    author: '匿名回答者',
    isPaid: false
  })

  saveData()
  newAnswer.value = ''
}

// 付费解锁回答
const unlockAnswer = (answer) => {
  answer.paid = true
  saveData()
}

// 查看问题详情
const viewQuestion = (question) => {
  currentQuestion.value = question
  question.views++
  saveData()
  currentView.value = 'detail'
}

// 格式化时间
const formatTime = (timestamp) => {
  const diff = Date.now() - timestamp
  if (diff < 60000) return '刚刚'
  if (diff < 3600000) return `${Math.floor(diff / 60000)}分钟前`
  if (diff < 86400000) return `${Math.floor(diff / 3600000)}小时前`
  return `${Math.floor(diff / 86400000)}天前`
}

onMounted(loadData)
</script>

<template>
  <div class="min-h-screen bg-gray-50">
    <!-- 顶部导航 -->
    <header class="bg-white shadow-sm sticky top-0 z-10">
      <div class="max-w-4xl mx-auto px-4 py-4 flex items-center justify-between">
        <h1 class="text-xl font-bold text-blue-600 cursor-pointer" @click="currentView = 'list'">
          💼 职业问答社区
        </h1>
        <button
          @click="currentView = 'ask'"
          class="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition"
        >
          提问
        </button>
      </div>
    </header>

    <!-- 问题列表 -->
    <div v-if="currentView === 'list'" class="max-w-4xl mx-auto px-4 py-6">
      <!-- 搜索框 -->
      <div class="mb-6">
        <input
          v-model="searchQuery"
          type="text"
          placeholder="搜索问题或标签..."
          class="w-full px-4 py-3 rounded-lg border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
      </div>

      <!-- 问题列表 -->
      <div class="space-y-4">
        <div
          v-for="q in filteredQuestions"
          :key="q.id"
          @click="viewQuestion(q)"
          class="bg-white rounded-lg shadow-sm p-4 cursor-pointer hover:shadow-md transition"
        >
          <h3 class="text-lg font-medium text-gray-800 mb-2">{{ q.title }}</h3>
          <p class="text-gray-500 text-sm mb-3 line-clamp-2">{{ q.content }}</p>
          <div class="flex items-center justify-between">
            <div class="flex gap-2">
              <span
                v-for="tag in q.tags"
                :key="tag"
                class="text-xs bg-blue-50 text-blue-600 px-2 py-1 rounded"
              >
                {{ tag }}
              </span>
            </div>
            <div class="text-xs text-gray-400">
              <span>{{ q.answers.length }} 回答</span>
              <span class="mx-2">·</span>
              <span>{{ q.views }} 浏览</span>
              <span class="mx-2">·</span>
              <span>{{ formatTime(q.createTime) }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 空状态 -->
      <div v-if="filteredQuestions.length === 0" class="text-center py-12 text-gray-400">
        暂无问题，点击右上角"提问"发起第一个问题吧
      </div>
    </div>

    <!-- 提问页面 -->
    <div v-else-if="currentView === 'ask'" class="max-w-4xl mx-auto px-4 py-6">
      <div class="bg-white rounded-lg shadow-sm p-6">
        <h2 class="text-xl font-bold mb-6">发起提问</h2>

        <div class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">问题标题</label>
            <input
              v-model="newQuestion.title"
              type="text"
              placeholder="用一句话描述你的问题"
              class="w-full px-4 py-2 rounded-lg border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">问题详情</label>
            <textarea
              v-model="newQuestion.content"
              rows="5"
              placeholder="详细描述你的问题背景、困惑点..."
              class="w-full px-4 py-2 rounded-lg border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500"
            ></textarea>
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">标签（用逗号分隔）</label>
            <input
              v-model="newQuestion.tags"
              type="text"
              placeholder="例如：转行, 程序员, 薪资"
              class="w-full px-4 py-2 rounded-lg border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>

          <div class="flex gap-4 pt-4">
            <button
              @click="submitQuestion"
              class="flex-1 bg-blue-500 text-white py-3 rounded-lg hover:bg-blue-600 transition font-medium"
            >
              发布问题
            </button>
            <button
              @click="currentView = 'list'"
              class="px-6 py-3 border border-gray-300 rounded-lg hover:bg-gray-50 transition"
            >
              取消
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- 问题详情 -->
    <div v-else-if="currentView === 'detail' && currentQuestion" class="max-w-4xl mx-auto px-4 py-6">
      <button
        @click="currentView = 'list'"
        class="text-blue-500 hover:text-blue-600 mb-4 flex items-center gap-1"
      >
        ← 返回列表
      </button>

      <div class="bg-white rounded-lg shadow-sm p-6 mb-4">
        <h2 class="text-xl font-bold mb-2">{{ currentQuestion.title }}</h2>
        <p class="text-gray-600 mb-4">{{ currentQuestion.content }}</p>
        <div class="flex items-center justify-between text-sm text-gray-400">
          <div class="flex gap-2">
            <span
              v-for="tag in currentQuestion.tags"
              :key="tag"
              class="bg-blue-50 text-blue-600 px-2 py-1 rounded"
            >
              {{ tag }}
            </span>
          </div>
          <div>
            <span>{{ currentQuestion.views }} 浏览</span>
            <span class="mx-2">·</span>
            <span>{{ formatTime(currentQuestion.createTime) }}</span>
          </div>
        </div>
      </div>

      <!-- 回答列表 -->
      <div class="mb-4">
        <h3 class="font-bold text-lg mb-4">{{ currentQuestion.answers.length }} 个回答</h3>

        <div class="space-y-4">
          <div
            v-for="answer in currentQuestion.answers"
            :key="answer.id"
            class="bg-white rounded-lg shadow-sm p-4"
          >
            <div v-if="answer.isPaid && !answer.paid" class="text-center py-6">
              <div class="text-4xl mb-2">🔒</div>
              <p class="text-gray-500 mb-3">这是付费回答，需支付 ¥{{ answer.price }} 解锁</p>
              <button
                @click="unlockAnswer(answer)"
                class="bg-orange-500 text-white px-6 py-2 rounded-lg hover:bg-orange-600 transition"
              >
                支付 ¥{{ answer.price }} 解锁
              </button>
              <p class="text-xs text-gray-400 mt-2">（模拟支付，实际未扣款）</p>
            </div>
            <div v-else>
              <p class="text-gray-700 mb-3">{{ answer.content }}</p>
              <div class="text-sm text-gray-400">
                <span>{{ answer.author }}</span>
                <span v-if="answer.isPaid" class="ml-2 text-orange-500">💰 付费回答</span>
              </div>
            </div>
          </div>
        </div>

        <!-- 无回答 -->
        <div v-if="currentQuestion.answers.length === 0" class="text-center py-8 text-gray-400">
          暂无回答，成为第一个回答者吧
        </div>
      </div>

      <!-- 写回答 -->
      <div class="bg-white rounded-lg shadow-sm p-4">
        <h4 class="font-medium mb-3">写下你的回答</h4>
        <textarea
          v-model="newAnswer"
          rows="4"
          placeholder="分享你的经验和见解..."
          class="w-full px-4 py-2 rounded-lg border border-gray-200 focus:outline-none focus:ring-2 focus:ring-blue-500 mb-3"
        ></textarea>
        <button
          @click="submitAnswer"
          class="bg-blue-500 text-white px-6 py-2 rounded-lg hover:bg-blue-600 transition"
        >
          提交回答
        </button>
      </div>
    </div>
  </div>
</template>

<style>
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>

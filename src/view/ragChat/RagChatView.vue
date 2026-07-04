<template>
  <div class="chat-container">
    <el-card class="box-card">
      <div class="chat-messages" ref="messageContainer">
        <div v-for="(message, index) in messages" :key="index" 
             :class="['message', message.role === 'user' ? 'user-message' : 'assistant-message']">
          <div class="message-wrapper">
            <div class="message-content" :class="{ 'typing': message.isTyping }" v-html="renderMarkdown(message.content)">
            </div>
            
            <el-button
              class="copy-button"
              type="text"
              size="small"
              @click="copyMessage(message.content)"
            >
              <el-icon><Document /></el-icon>
            </el-button>
          </div>
        </div>
      </div>

      <div class="input-container">
        <el-input
          v-model="userInput"
          type="textarea"
          :rows="3"
          placeholder="请输入您的问题..."
          @keyup.enter="handleRagSend"
        />
      </div>

      <div class="button-group">
        <el-button type="primary" @click="handleRagSend" :loading="isLoading">RAG回答</el-button>
        <el-button type="warning" @click="clearMessages">清空对话</el-button>
      </div>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, nextTick } from 'vue'
import { marked } from 'marked'
import { Document } from '@element-plus/icons-vue'
import { ElMessage } from 'element-plus'
import { ChatApi, type ChatMessage } from '@/api/ChatApi'
import { getStreamChat } from '@/api/StreamApi'
import { queryFileApi } from '@/api/KnowHubApi'
import { StoreFile } from '@/api/data'
 

const messages = ref<ChatMessage[]>([])
const userInput = ref('')
const isLoading = ref(false)
const messageContainer = ref<HTMLElement | null>(null)
const knowledgeFiles = ref<StoreFile[]>([])
const selectedFiles = ref<string[]>([])

// 加载知识库文件列表
const loadKnowledgeFiles = () => {
  const params = { 
    page: 0, 
    pageSize: 100, // 假设一页最多加载100个文件
    fileName: "" 
  }
  
  queryFileApi(params)
    .then((res) => {
      if (res.code == 0) {
        const data = res.data;
        knowledgeFiles.value = data.records || [];
      } else {
        ElMessage({
          type: 'error',
          message: res.message,
        });
      }
    })
    .catch((err) => {
      ElMessage({
        type: 'error',
        message: err,
      });
    });
};

// 处理普通对话
const handleSend = async () => {
  if (!userInput.value.trim() || isLoading.value) return
  await sendMessage(ChatApi.Chat)
}

// 处理RAG对话
const handleRagSend = async () => {
  if (!userInput.value.trim() || isLoading.value) return
  await sendMessage(ChatApi.RagChat, selectedFiles.value)
}

// 发送消息通用方法
const sendMessage = async (url: string, selectedFileIds: string[] = []) => {
  messages.value.push({
    role: 'user',
    content: userInput.value
  })

  const currentInput = userInput.value
  userInput.value = ''
  isLoading.value = true

  messages.value.push({
    role: 'assistant',
    content: '正在思考中...', // 建议初始设为空字符串或思考中，看你需求
    isTyping: true
  })
   // 2. 关键步骤：从 messages.value 数组中获取刚才 push 进去的【响应式对象】
   // 这样修改它时，Vue 才能监听到
  const lastIndex = messages.value.length - 1
  const reactiveMessage = messages.value[lastIndex]
  // --- 修改结束 ---

  // 如果初始内容是"正在思考中..."，在收到第一个字符时可能需要清空它，或者保留它
  // 这里假设你想保留"正在思考中..."直到流开始，或者直接覆盖，下面逻辑稍微调整一下:
  // 建议：初始 content 设为 ''，或者在收到第一个包时清空 '正在思考中...'
  let isFirstChunk = true;
 
  // 获取选中的文件名
  const fileSources = selectedFileIds.map(id => {
    const file = knowledgeFiles.value.find(f => f.id === id)
    return file ? file.fileName : ''
  }).filter(name => name !== '')
  
  // 使用更新后的getStreamChat函数，它始终使用POST请求
  if (fileSources.length > 0) {
    // 有文件选择时，传递sources参数
    getStreamChat(currentInput, url, (value) => {
      const text = value.data;
      
      // 如果是第一包数据，且当前内容是占位符，先清空
      if (isFirstChunk && reactiveMessage.content === '正在思考中...') {
        reactiveMessage.content = '';
        isFirstChunk = false;
      }

      // 3. 修改响应式对象
      reactiveMessage.content += text 
      
      // 滚动到底部
      scrollToBottom()

    }, (error) => {
      window.console.error('Error:', error)
      reactiveMessage.content = '抱歉，发生了错误，请稍后重试。'
    }, () => { 
      isLoading.value = false
      reactiveMessage.isTyping = false
    }, fileSources)
  } else {
    // 无文件选择时，不传递sources参数
    getStreamChat(currentInput, url, (value) => {
      const text = value.data;
      
      // 如果是第一包数据，且当前内容是占位符，先清空
      if (isFirstChunk && reactiveMessage.content === '正在思考中...') {
        reactiveMessage.content = '';
        isFirstChunk = false;
      }

      // 3. 修改响应式对象
      reactiveMessage.content += text 
      
      // 滚动到底部
      scrollToBottom()

    }, (error) => {
      window.console.error('Error:', error)
      reactiveMessage.content = '抱歉，发生了错误，请稍后重试。'
    }, () => { 
      isLoading.value = false
      reactiveMessage.isTyping = false
    })
  }
};

// 滚动到底部
const scrollToBottom = () => {
  if (!messageContainer.value) return
  
  const container = messageContainer.value
  
  // 1. 立即尝试滚动（应对普通更新）
  container.scrollTop = container.scrollHeight
  
  // 2. 延迟再次滚动（应对流式输出中的浏览器重绘延迟）
  setTimeout(() => {
    container.scrollTop = container.scrollHeight
  }, 100) // 100ms 的延迟通常能解决渲染滞后问题
}

// 复制消息
const copyMessage = async (content: string) => {
  try {
    await navigator.clipboard.writeText(content)
    ElMessage({
      message: '复制成功',
      type: 'success',
      duration: 2000
    })
  } catch (err) {
    ElMessage({
      message: '复制失败',
      type: 'error',
      duration: 2000
    })
  }
}

// 清空对话
const clearMessages = () => {
  messages.value = [{
    role: 'assistant',
    content: '你好！我是AI助手，请问有什么可以帮助你的吗？'
  }]
}

// Markdown渲染
const renderMarkdown = (content: string) => {
  try {
    return marked(content, {
      breaks: true,
      gfm: true
    })
  } catch (error) {
    console.error('Markdown parsing error:', error)
    return content
  }
}

// 处理文件选择变化
const handleFileSelectionChange = (value: string[]) => {
  selectedFiles.value = value
}

// 移除选中的文件
const removeSelectedFile = (fileId: string) => {
  const index = selectedFiles.value.indexOf(fileId)
  if (index > -1) {
    selectedFiles.value.splice(index, 1)
  }
}

// 根据文件ID获取文件名
const getFileNameById = (fileId: string) => {
  const file = knowledgeFiles.value.find(f => f.id === fileId)
  return file ? file.fileName : ''
}

// 根据文件ID获取截断的文件名
const getFileNameByLength = (fileId: string, maxLength: number) => {
  const fileName = getFileNameById(fileId)
  if (fileName.length <= maxLength) {
    return fileName
  }
  return fileName.substring(0, maxLength) + '...'
}

onMounted(() => {
  messages.value.push({
    role: 'assistant',
    content: '你好！我是AI助手，请问有什么可以帮助你的吗？'
  })
  
  // 加载知识库文件列表
  loadKnowledgeFiles()
})
</script>

<style scoped lang="less">
.chat-container {
  height: calc(100vh - 24px);
  padding: 18px;
  box-sizing: border-box;
  overflow: hidden;

  .box-card {
    height: 100%;
    display: flex;
    flex-direction: column;
    
    :deep(.el-card__body) {
      flex: 1;
      display: flex;
      flex-direction: column;
      padding: 18px;
      overflow: hidden;
    }
  }
}

.file-selection-container {
  margin-bottom: 15px;
  padding: 10px;
  background: rgba(15, 23, 42, 0.66);
  border: 0;
  border-radius: 8px;
  box-shadow: inset 0 0 0 1px rgba(103, 232, 249, 0.055);

  .file-selection-header {
    margin-bottom: 10px;

    .file-selection-label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
      color: #cfe7ff;
    }
  }

  .selected-files-info {
    margin-top: 10px;
    padding: 10px;
    background-color: rgba(34, 211, 238, 0.10);
    border: 0;
    border-radius: 8px;
    box-shadow: inset 0 0 0 1px rgba(103, 232, 249, 0.08);

    span {
      display: block;
      margin-bottom: 8px;
      font-weight: 500;
      color: #67e8f9;
    }
  }
}

.file-selection-inline {
  display: flex;
  align-items: center;
  padding: 5px;
  flex-wrap: nowrap;
}

.file-selection-inline .file-selection-label {
  margin-right: 10px;
  font-weight: 500;
  color: #cfe7ff;
  font-size: 14px;
}

.selected-files-popover {
  max-height: 200px;
  overflow-y: auto;
}

.chat-messages {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 14px;
  padding: 18px;
  border: 0;
  border-radius: 8px;
  background:
    radial-gradient(circle at 100% 0%, rgba(139, 92, 246, 0.10), transparent 18rem),
    linear-gradient(180deg, rgba(3, 7, 18, 0.58), rgba(8, 13, 28, 0.72));
  box-shadow: inset 0 0 0 1px rgba(103, 232, 249, 0.055);
  min-height: 0;

  &::-webkit-scrollbar {
    width: 6px;
  }

  &::-webkit-scrollbar-thumb {
    background: linear-gradient(180deg, rgba(34, 211, 238, 0.74), rgba(139, 92, 246, 0.66));
    border-radius: 3px;
  }

  &::-webkit-scrollbar-track {
    background-color: transparent;
  }
}

.message {
  margin-bottom: 16px;
  max-width: 80%;

  &.user-message {
    margin-left: auto;
    text-align: right;

    .message-wrapper {
      flex-direction: row-reverse;
      justify-content: flex-start;
    }

    .message-content {
      background: linear-gradient(135deg, rgba(34, 211, 238, 0.92), rgba(139, 92, 246, 0.76));
      color: #03101f;
      box-shadow: 0 0 34px rgba(34, 211, 238, 0.22), inset 0 0 0 1px rgba(125, 249, 255, 0.12);
    }
  }

  &.assistant-message {
    margin-right: auto;
    text-align: left;
  }
}

.message-wrapper {
  display: flex;
  align-items: flex-start;
  gap: 8px;
}

.message-content {
  display: inline-block;
  padding: 12px 15px;
  border: 0;
  border-radius: 8px;
  background:
    linear-gradient(145deg, rgba(15, 23, 42, 0.92), rgba(8, 13, 28, 0.86));
  color: #dff4ff;
  word-break: break-word;
  font-size: 14px;
  line-height: 1.65;
  box-shadow: 0 14px 34px rgba(0, 0, 0, 0.28), inset 0 0 0 1px rgba(103, 232, 249, 0.06);

  :deep(p) {
    margin: 0;
    line-height: 1.5;
  }

  :deep(pre) {
    background-color: rgba(3, 7, 18, 0.78);
    border: 0;
    box-shadow: inset 0 0 0 1px rgba(103, 232, 249, 0.06);
    padding: 10px;
    border-radius: 4px;
    overflow-x: auto;
    font-size: 13px;
  }

  :deep(code) {
    font-family: Consolas, Monaco, 'Andale Mono', monospace;
    background-color: rgba(34, 211, 238, 0.10);
    color: #a5f3fc;
    padding: 2px 4px;
    border-radius: 3px;
    font-size: 13px;
  }

  :deep(ul), :deep(ol) {
    padding-left: 20px;
    margin: 8px 0;
  }

  :deep(blockquote) {
    margin: 8px 0;
    padding-left: 10px;
    border-left: 3px solid rgba(34, 211, 238, 0.42);
    color: #bcd4ef;
  }

  &.typing {
    &::after {
      content: '...';
      animation: ellipsis 1.5s infinite;
    }
  }
}

@keyframes ellipsis {
  0% { content: '.'; }
  33% { content: '..'; }
  66% { content: '...'; }
  100% { content: '.'; }
}

.copy-button {
  opacity: 0;
  transition: opacity 0.3s;
  padding: 4px;
  height: auto;

  &:hover {
    opacity: 1;
  }
}

.input-container {
  margin-top: auto;
  display: flex;
  gap: 10px;
  padding: 12px;
  background: rgba(3, 7, 18, 0.58);
  border: 0;
  border-radius: 8px;
  box-shadow: inset 0 0 0 1px rgba(103, 232, 249, 0.055), 0 0 28px rgba(34, 211, 238, 0.08);
  min-height: 100px;

  .el-textarea {
    flex: 1;
  }

  :deep(.el-textarea__inner) {
    min-height: 96px !important;
    color: #eaf6ff;
    border: 0;
    background:
      radial-gradient(circle at 100% 0%, rgba(34, 211, 238, 0.08), transparent 12rem),
      rgba(3, 7, 18, 0.72);
    box-shadow: inset 0 0 0 1px rgba(103, 232, 249, 0.045), 0 0 24px rgba(34, 211, 238, 0.08);
  }

  :deep(.el-textarea__inner:focus) {
    border: 0;
    box-shadow: 0 0 0 1px rgba(34, 211, 238, 0.24) inset, 0 0 34px rgba(34, 211, 238, 0.16);
  }
}

.button-group {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: 10px;
  margin-top: 10px;
  padding: 8px 0 0;
  flex-wrap: wrap;

  .el-button {
    width: 128px;
    height: 40px;
    font-size: 14px;
  }
}
</style>

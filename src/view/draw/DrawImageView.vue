<template>
  <div class="draw-image">
    <div class="input-section">
      <textarea
          v-model="prompt"
          placeholder="描述你所想象的图片..."
          class="prompt-input"
      />
<!--      <div class="ratio-select">-->
<!--        <label v-for="option in ratioOptions" :key="option.value" class="ratio-option">-->
<!--          <input-->
<!--              type="radio"-->
<!--              v-model="ratio"-->
<!--              :value="option.value"-->
<!--          />-->
<!--          {{ option.label }}-->
<!--        </label>-->
<!--      </div>-->
      <div class="button-group">
        <button @click="generateImage" :disabled="isLoading" class="generate-btn">
          {{ isLoading ? '生成中...' : '生成图片' }}
        </button>
        <button v-if="generatedImage" @click="downloadImage" class="download-btn">
          下载图片
        </button>
      </div>
    </div>

    <div class="image-preview" v-if="generatedImage">
      <div class="preview-container">
        <img :src="generatedImage" alt="生成的图片" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import {BASE_URL} from "@/http/config.ts";

const prompt = ref('')
const generatedImage = ref('')
const isLoading = ref(false)


const generateImage = async () => {
  if (!prompt.value) return
  
  isLoading.value = true
  try {
    const response = await fetch(BASE_URL+`/draw/image?prompt=${encodeURIComponent(prompt.value)}`)
    if (response.ok) {
      const blob = await response.blob()
      generatedImage.value = URL.createObjectURL(blob)
    }
  } catch (error) {
    console.error('生成图片失败:', error)
  } finally {
    isLoading.value = false
  }
}

const downloadImage = () => {
  if (!generatedImage.value) return
  
  // 创建一个临时的a标签用于下载
  const link = document.createElement('a')
  link.href = generatedImage.value
  link.download = `generated-image-${Date.now()}.png` // 使用时间戳确保文件名唯一
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
}
</script>

<style scoped lang="less">
.draw-image {
  min-height: calc(100vh - 24px);
  padding: 24px;
  max-width: 1180px;
  margin: 0 auto;
  
  .input-section {
    margin-bottom: 22px;
    padding: 22px;
    border: 0;
    border-radius: 8px;
    background:
      radial-gradient(circle at 100% 0%, rgba(139, 92, 246, 0.14), transparent 18rem),
      linear-gradient(180deg, rgba(15, 23, 42, 0.82), rgba(8, 13, 28, 0.74));
    box-shadow: 0 22px 56px rgba(0, 0, 0, 0.34), 0 0 34px rgba(34, 211, 238, 0.08), inset 0 0 0 1px rgba(103, 232, 249, 0.045);
  }
  
  .prompt-input {
    width: 100%;
    min-height: 132px;
    padding: 14px 16px;
    border: 0;
    border-radius: 8px;
    resize: vertical;
    margin-bottom: 16px;
    color: #eaf6ff;
    background: rgba(3, 7, 18, 0.52);
    outline: none;
    box-shadow: inset 0 0 0 1px rgba(103, 232, 249, 0.045);
    transition: box-shadow .18s ease;

    &:focus {
      box-shadow: inset 0 0 0 1px rgba(125, 249, 255, 0.24), 0 0 0 4px rgba(34, 211, 238, 0.10), 0 0 28px rgba(34, 211, 238, 0.12);
    }
  }
  
  .ratio-select {
    display: flex;
    flex-wrap: wrap;
    gap: 12px;
    margin-bottom: 16px;
    
    .ratio-option {
      display: flex;
      align-items: center;
      gap: 4px;
      cursor: pointer;
    }
  }
  
  .button-group {
    display: flex;
    gap: 12px;
    align-items: center;
  }
  
  .generate-btn {
    padding: 12px 24px;
    background: linear-gradient(135deg, #22d3ee, #8b5cf6);
    color: #04111f;
    border: none;
    border-radius: 7px;
    cursor: pointer;
    box-shadow: 0 0 34px rgba(34, 211, 238, 0.26);
    transition: transform .18s ease, box-shadow .18s ease;

    &:hover:not(:disabled) {
      transform: translateY(-1px);
      box-shadow: 0 0 42px rgba(34, 211, 238, 0.34);
    }
    
    &:disabled {
      background: rgba(71, 85, 105, 0.72);
      color: rgba(226, 232, 240, 0.62);
      cursor: not-allowed;
    }
  }
  
  .download-btn {
    padding: 12px 24px;
    background: rgba(15, 23, 42, 0.88);
    color: white;
    border: none;
    border-radius: 7px;
    cursor: pointer;
    transition: background 0.3s, transform .18s ease;
    
    &:hover {
      background: rgba(30, 41, 59, 0.96);
      transform: translateY(-1px);
    }
  }
  
  .image-preview {
    margin-top: 20px;
    display: flex;
    justify-content: center;
    
    .preview-container {
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 16px;
      
      img {
        max-width: 100%;
        max-height: 520px;
        width: auto;
        height: auto;
        border-radius: 8px;
        border: 0;
        box-shadow: 0 24px 74px rgba(0, 0, 0, 0.42), 0 0 38px rgba(34, 211, 238, 0.12), inset 0 0 0 1px rgba(103, 232, 249, 0.045);
      }
    }
  }
}
</style>

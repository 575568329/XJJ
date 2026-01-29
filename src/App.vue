<script setup>
import { ref, onMounted, computed } from 'vue'
import config from './config.js'

// 从配置文件读取模式
const mode = ref(config.mode)

// 当前视频类型
const currentType = ref(mode.value === 'simple' ? 'BianZhuang' : config.videoTypes[0].value)

// 视频URL
const videoUrl = ref('')

// 加载状态
const loading = ref(false)

// 视频类型列表（从配置文件读取）
const videoTypes = config.videoTypes

// 检查是否在utools环境中
const isUtools = computed(() => {
  return typeof window !== 'undefined' && window.utools
})

// 加载视频
const loadVideo = () => {
  loading.value = true
  // 添加时间戳防止缓存
  const timestamp = Date.now()
  videoUrl.value = `http://api.mmp.cc/api/ksvideo?type=mp4&id=${currentType.value}&t=${timestamp}`

  // 视频加载完成后隐藏loading
  const video = document.getElementById('videoPlayer')
  if (video) {
    video.onloadeddata = () => {
      loading.value = false
    }
    video.onerror = () => {
      loading.value = false
    }
  }
}

// 刷新视频
const refreshVideo = () => {
  loadVideo()
}

// 类型改变
const onTypeChange = () => {
  loadVideo()
}

// 获取utools窗口大小
const getUtoolsWindowSize = () => {
  if (isUtools.value) {
    try {
      // 隐藏子输入框，让界面更简洁
      if (window.utools.setSubInput) {
        window.utools.setSubInput(() => {}, '', false)
      }
    } catch (e) {
      console.log('初始化utools窗口失败:', e)
    }
  }
}

// 页面加载时获取第一个视频
onMounted(() => {
  getUtoolsWindowSize()
  loadVideo()
})
</script>

<template>
  <div class="app-container">
    <!-- 头部 -->
    <header class="header">
      <h1 class="title">🌸 小姐姐短视频</h1>
    </header>

    <!-- 主内容区 -->
    <main class="main-content">
      <!-- 视频播放器 -->
      <div class="video-container">
        <div v-if="loading" class="loading-overlay">
          <div class="spinner"></div>
          <p>加载中...</p>
        </div>
        <video
          id="videoPlayer"
          class="video-player"
          controls
          autoplay
          loop
          muted
          :src="videoUrl"
        ></video>
      </div>

      <!-- 控制区 -->
      <div class="controls">
        <button class="refresh-btn" @click="refreshVideo" title="换一个">
          <span class="icon-wrapper">
            <span class="refresh-icon">🔄</span>
          </span>
          <span class="btn-text">换一个</span>
        </button>

        <!-- 类型选择器（仅付费模式显示） -->
        <div v-if="mode === 'premium'" class="type-selector" title="选择类型">
          <span class="icon-wrapper">
            <span class="type-icon">🎬</span>
          </span>
          <div class="type-content">
            <label for="videoType">类型：</label>
            <select
              id="videoType"
              v-model="currentType"
              class="type-select"
              @change="onTypeChange"
            >
              <option v-for="type in videoTypes" :key="type.value" :value="type.value">
                {{ type.label }}
              </option>
            </select>
          </div>
        </div>
      </div>
    </main>

  </div>
</template>

<style scoped>
.app-container {
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  overflow: hidden;
}

.header {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  padding: 0.5rem 1rem;
  text-align: center;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  flex-shrink: 0;
}

.title {
  margin: 0;
  color: #fff;
  font-size: 1.1rem;
  font-weight: bold;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.main-content {
  flex: 1;
  padding: 0.75rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.video-container {
  position: relative;
  background: #000;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
  margin-bottom: 0.75rem;
  flex: 1;
  min-height: 0;
}

.video-player {
  width: 100%;
  height: 100%;
  display: block;
  object-fit: contain;
}

.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #fff;
  z-index: 10;
}

.spinner {
  width: 35px;
  height: 35px;
  border: 3px solid rgba(255, 255, 255, 0.3);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 0.5rem;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.controls {
  position: fixed;
  top: 50%;
  right: 1rem;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.75rem;
  z-index: 100;
}

.refresh-btn {
  padding: 0;
  border: none;
  border-radius: 50%;
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: #fff;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 20px rgba(245, 87, 108, 0.5);
  display: grid;
  place-items: center;
  backdrop-filter: blur(10px);
  width: 45px;
  height: 45px;
  overflow: hidden;
  white-space: nowrap;
  position: relative;
}

.refresh-btn:hover {
  width: auto;
  padding: 0.6rem 1.5rem;
  border-radius: 25px;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  gap: 0.6rem;
  transform: translateX(-5px);
  box-shadow: 0 5px 25px rgba(245, 87, 108, 0.7);
}

.refresh-btn:active {
  transform: translateX(-5px) scale(0.95);
}

.icon-wrapper {
  display: grid;
  place-items: center;
  position: absolute;
  inset: 0;
}

.refresh-btn:hover .icon-wrapper {
  position: static;
  inset: auto;
}

.refresh-icon {
  font-size: 1.3rem;
  display: block;
}

.refresh-btn:hover .refresh-icon {
  animation: rotate 2s linear infinite;
}

.btn-text {
  opacity: 0;
  transition: opacity 0.2s ease;
  white-space: nowrap;
}

.refresh-btn:hover .btn-text {
  opacity: 1;
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.type-selector {
  display: grid;
  place-items: center;
  gap: 0;
  background: rgba(0, 0, 0, 0.6);
  padding: 0;
  border-radius: 50%;
  backdrop-filter: blur(15px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  width: 45px;
  height: 45px;
  overflow: hidden;
  transition: all 0.3s ease;
  cursor: pointer;
  position: relative;
}

.type-selector:hover {
  width: auto;
  padding: 0.5rem 0.75rem;
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  gap: 0.5rem;
  transform: translateX(-5px);
}

.type-selector .icon-wrapper {
  position: absolute;
  inset: 0;
}

.type-selector:hover .icon-wrapper {
  position: static;
  inset: auto;
}

.type-icon {
  font-size: 1.3rem;
  display: block;
}

.type-content {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  opacity: 0;
  transition: opacity 0.2s ease;
  white-space: nowrap;
}

.type-selector:hover .type-content {
  opacity: 1;
}

.type-selector label {
  color: #fff;
  font-weight: 600;
  font-size: 0.85rem;
}

.type-select {
  padding: 0.4rem 1.25rem;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.9);
  color: #333;
  font-size: 0.85rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  outline: none;
  min-width: 110px;
}

.type-select:hover {
  border-color: rgba(255, 255, 255, 0.5);
  background: #fff;
}

.type-select:focus {
  border-color: #f093fb;
  box-shadow: 0 0 0 3px rgba(240, 147, 251, 0.3);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .header {
    padding: 0.4rem 0.75rem;
  }

  .title {
    font-size: 1rem;
  }

  .main-content {
    padding: 0.5rem;
  }

  .controls {
    right: 0.75rem;
    gap: 0.5rem;
  }

  .refresh-btn {
    width: 40px;
    height: 40px;
  }

  .refresh-btn:hover {
    padding: 0.5rem 1.25rem;
  }

  .icon-wrapper {
    width: 40px;
    height: 40px;
  }

  .refresh-icon {
    font-size: 1.1rem;
  }

  .type-selector {
    width: 40px;
    height: 40px;
  }

  .type-selector:hover {
    max-width: calc(100vw - 3rem);
    padding: 0.4rem 0.6rem;
  }

  .type-selector .icon-wrapper {
    width: 40px;
    height: 40px;
  }

  .type-icon {
    font-size: 1.1rem;
  }

  .type-select {
    font-size: 0.8rem;
    min-width: 100px;
  }
}

/* 小屏幕优化 */
@media (max-height: 500px) {
  .header {
    padding: 0.3rem 0.5rem;
  }

  .title {
    font-size: 0.9rem;
  }

  .main-content {
    padding: 0.4rem;
  }

  .video-container {
    margin-bottom: 0.5rem;
    border-radius: 8px;
  }

  .controls {
    right: 0.5rem;
    gap: 0.4rem;
  }

  .refresh-btn {
    width: 38px;
    height: 38px;
  }

  .refresh-btn:hover {
    padding: 0.4rem 1rem;
  }

  .icon-wrapper {
    width: 38px;
    height: 38px;
  }

  .refresh-icon {
    font-size: 1rem;
  }

  .type-selector {
    width: 38px;
    height: 38px;
  }

  .type-selector:hover {
    padding: 0.35rem 0.5rem;
  }

  .type-selector .icon-wrapper {
    width: 38px;
    height: 38px;
  }

  .type-icon {
    font-size: 1rem;
  }

  .type-selector label {
    font-size: 0.75rem;
  }

  .type-select {
    padding: 0.35rem 0.85rem;
    font-size: 0.75rem;
    min-width: 90px;
  }
}
</style>

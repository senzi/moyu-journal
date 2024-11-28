<template>
  <div class="app-container">
    <header>
      <div class="title-section">
        <div class="main-title" @click="editUsername = true">
          <span v-if="!editUsername" class="username">{{ username || '某某某' }}</span>
          <input
            v-else
            ref="usernameInput"
            v-model="username"
            class="username-input"
            @blur="saveUsername"
            @keyup.enter="saveUsername"
            placeholder="某某某"
          >
          <span class="title-separator">的</span>
          <span class="title-main">摸鱼纪</span>
        </div>
        <div class="subtitle">「上班不摸鱼，和咸鱼有什么区别」</div>
      </div>
      <div class="menu-wrapper">
        <button class="menu-trigger" @click="isMenuOpen = !isMenuOpen">
          <span class="material-icons">more_vert</span>
        </button>
        <div class="menu-dropdown" v-show="isMenuOpen" @click.stop="isMenuOpen = false">
          <input type="file" accept=".jsonl" @change="importFile" style="display: none" ref="fileInput">
          <button @click="$refs.fileInput.click()">导入文件</button>
          <button @click="importFromURL">从URL导入</button>
          <button @click="exportData">导出数据</button>
          <div class="delete-wrapper">
            <button @click.stop="showClearConfirm" class="danger">清空数据</button>
            <div class="delete-confirm" v-if="showClearConfirmDialog" @click.stop>
              <span class="confirm-text">确定清空</span>
              <button class="confirm-btn confirm-yes" @click.stop="confirmClear">
                <span class="material-icons">check</span>
              </button>
              <button class="confirm-btn confirm-no" @click.stop="cancelClear">
                <span class="material-icons">close</span>
              </button>
            </div>
          </div>
        </div>
      </div>
    </header>

    <main class="main-content">
      <div class="toolbar">
        <button class="new-project" @click="createNew">
          <div class="fish-pond">
            <span class="fish">🐟</span>
            <span class="bubble bubble-1">。</span>
            <span class="bubble bubble-2">。</span>
            <span class="bubble bubble-3">。</span>
          </div>
          <span class="fish-text">摸一条新的鱼</span>
          <span class="fish-wave">〰〰〰</span>
        </button>
      </div>

      <div v-if="projects.length === 0" class="empty-state">
        还没有任何摸鱼项目，开始摸鱼或导入数据吧！
      </div>

      <draggable v-model="sortedProjects" v-else class="project-list" ghost-class="ghost" 
        :item-key="'id'"
        @start="drag = true"
        @end="drag = false">
        <template #item="{ element: project }">
          <div class="project-card">
            <div class="card-header" :class="{ 'edit-mode': editMode === project.id }">
              <div class="project-title">
                <div v-if="editMode !== project.id" class="emoji-icon">{{ project.emoji_icon || '🚀' }}</div>
                <div v-else class="emoji-picker-wrapper">
                  <div class="emoji-trigger" @click="openEmojiPicker(project.id)">
                    {{ project.emoji_icon || '🚀' }}
                  </div>
                  <div v-if="showEmojiPicker && currentProjectId === project.id" class="emoji-picker-container">
                    <div class="emoji-picker-overlay" @click="showEmojiPicker = false"></div>
                    <EmojiPicker @select="onEmojiSelect" />
                  </div>
                </div>
                <h3 v-if="editMode !== project.id">{{ project.title || '未命名项目' }}</h3>
                <input v-else v-model="project.title" class="title-input" placeholder="项目名称" @keyup.enter="saveEdit(project)">
              </div>
              <div class="card-actions">
                <button class="edit-btn" @click="toggleEditMode(project)">
                  <span class="material-icons">{{ editMode === project.id ? 'done' : 'edit' }}</span>
                </button>
                <div class="delete-wrapper" v-if="editMode === project.id">
                  <button class="delete-btn" @click.stop="showDeleteConfirm(project.id)">
                    <span class="material-icons">delete</span>
                  </button>
                  <div class="delete-confirm" v-if="deleteConfirmId === project.id" @click.stop>
                    <span class="confirm-text">确定删除</span>
                    <button class="confirm-btn confirm-yes" @click.stop="confirmDelete(project.id)">
                      <span class="material-icons">check</span>
                    </button>
                    <button class="confirm-btn confirm-no" @click.stop="cancelDelete">
                      <span class="material-icons">close</span>
                    </button>
                  </div>
                </div>
              </div>
            </div>

            <div class="project-content">
              <div class="project-tags">
                <span v-if="editMode !== project.id" class="tag date">{{ formatDate(project.createdAt) }}</span>
                <input
                  v-else
                  type="date"
                  class="date-input"
                  :value="formatDateForInput(project.createdAt)"
                  @input="updateDate($event, project)"
                >
                <span v-if="editMode !== project.id" class="tag status">{{ project.status || '规划中' }}</span>
                <select v-else v-model="project.status" class="status-select">
                  <option value="规划中">规划中</option>
                  <option value="进行中">进行中</option>
                  <option value="已完成">已完成</option>
                  <option value="已搁置">已搁置</option>
                </select>
                <span v-if="editMode !== project.id" class="tag tech">{{ project.techStack || '未设置技术栈' }}</span>
                <input v-else v-model="project.techStack" class="tech-input" placeholder="使用技术">
              </div>

              <div class="project-notes">
                <div class="note-section">
                  <label>打算怎么摸</label>
                  <div v-if="editMode !== project.id" class="note-content">{{ project.description || '暂无描述' }}</div>
                  <textarea v-else v-model="project.description" class="description-input" placeholder="打算怎么摸"></textarea>
                </div>
                <div class="note-section">
                  <label>吐槽</label>
                  <div v-if="editMode !== project.id" class="note-content">{{ project.thoughts || '暂无记录' }}</div>
                  <textarea v-else v-model="project.thoughts" class="description-input" placeholder="有什么想吐槽的"></textarea>
                </div>
              </div>

              <div class="project-links">
                <div v-if="editMode !== project.id" class="link-buttons">
                  <a v-if="project.demoLink" :href="project.demoLink" target="_blank" class="link-btn">
                    <span class="material-icons">launch</span>
                    查看Demo
                  </a>
                  <a v-if="project.repoLink" :href="project.repoLink" target="_blank" class="link-btn">
                    <span class="material-icons">code</span>
                    查看仓库
                  </a>
                </div>
                <div v-else class="link-inputs">
                  <div class="link-input-group">
                    <label>Demo链接</label>
                    <input v-model="project.demoLink" class="link-input" placeholder="演示链接（可选）">
                  </div>
                  <div class="link-input-group">
                    <label>仓库链接</label>
                    <input v-model="project.repoLink" class="link-input" placeholder="仓库链接（可选）">
                  </div>
                </div>
              </div>
            </div>
          </div>
        </template>
      </draggable>
    </main>

    <footer class="footer">
      <div class="footer-content">
        <div class="footer-links">
          <a href="https://github.com/sennes2/moyu-journal" target="_blank">
            <span class="material-icons">code</span>
            <span>GitHub</span>
          </a>
          <span class="separator">|</span>
          <a href="https://github.com/sennes2/moyu-journal/blob/main/LICENSE" target="_blank">
            <span class="material-icons">gavel</span>
            <span>MIT License</span>
          </a>
        </div>
        <div class="copyright">
          2024 摸鱼纪
        </div>
      </div>
    </footer>
  </div>
</template>

<script>
import draggable from 'vuedraggable'
import { nanoid } from 'nanoid'
import EmojiPicker from 'vue3-emoji-picker'
import 'vue3-emoji-picker/css'

export default {
  components: {
    draggable,
    EmojiPicker
  },

  data() {
    return {
      projects: [],
      editMode: null,
      isMenuOpen: false,
      showEmojiPicker: false,
      currentProjectId: null,
      drag: false,
      editUsername: false,
      username: localStorage.getItem('moyu-username') || '',
      deleteConfirmId: null,
      showClearConfirmDialog: false,
    }
  },

  computed: {
    sortedProjects: {
      get() {
        return this.projects
      },
      set(value) {
        this.projects = value
        this.saveProjects() // 拖拽排序后保存
      }
    }
  },

  watch: {
    // 深度监听 projects 数组的变化
    projects: {
      handler(newVal) {
        this.saveProjects()
      },
      deep: true
    }
  },

  mounted() {
    this.loadProjects()
    // 点击页面其他地方关闭菜单
    document.addEventListener('click', this.handleClickOutside)
  },

  beforeUnmount() {  // Vue 3 中使用 beforeUnmount 替代 beforeDestroy
    document.removeEventListener('click', this.handleClickOutside)
  },

  methods: {

    handleClickOutside(event) {
      // 确保点击的不是菜单按钮本身或菜单内容
      const menuWrapper = document.querySelector('.menu-wrapper')
      if (menuWrapper && !menuWrapper.contains(event.target)) {
        this.isMenuOpen = false
      }
    },

    toggleEditMode(project) {
      this.editMode = this.editMode === project.id ? null : project.id
    },

    showDeleteConfirm(id) {
      this.deleteConfirmId = id
    },

    cancelDelete() {
      this.deleteConfirmId = null
    },

    confirmDelete(id) {
      this.projects = this.projects.filter(p => p.id !== id)
      this.deleteConfirmId = null
      this.editMode = null
      this.saveProjects()
    },

    openEmojiPicker(projectId) {
      this.currentProjectId = projectId
      this.showEmojiPicker = true
    },

    onEmojiSelect(emoji) {
      const project = this.projects.find(p => p.id === this.currentProjectId)
      if (project) {
        project.emoji_icon = emoji.i
        this.saveProjects()
      }
      this.showEmojiPicker = false
    },

    loadProjects() {
      const stored = localStorage.getItem('moyu-projects')
      if (stored) {
        this.projects = JSON.parse(stored)
      }
    },

    createNew() {
      const newProject = {
        id: nanoid(),
        createdAt: new Date().toISOString(),
        title: '',
        emoji_icon: '🚀', // 默认emoji
        description: '', // 前端显示为"打算怎么摸？"
        techStack: '',
        status: '规划中',
        demoLink: '',
        repoLink: '',
        thoughts: '' // 前端显示为"吐槽"
      }
      this.projects.push(newProject)
      this.saveProjects()
    },

    saveProjects() {
      localStorage.setItem('moyu-projects', JSON.stringify(this.projects))
    },

    async importFromURL() {
      const url = prompt('请输入配置URL：')
      if (!url) return

      try {
        const response = await fetch(url)
        const text = await response.text()
        const newProjects = text.split('\n')
          .filter(Boolean)
          .map(line => JSON.parse(line))
        this.projects = newProjects
        this.saveProjects()
      } catch (error) {
        alert('导入失败：' + error.message)
      }
    },

    importFile(event) {
      const file = event.target.files[0]
      if (!file) return

      const reader = new FileReader()
      reader.onload = (e) => {
        try {
          const text = e.target.result
          const newProjects = text.split('\n')
            .filter(Boolean)
            .map(line => JSON.parse(line))
          this.projects = newProjects
          this.saveProjects()
        } catch (error) {
          alert('导入失败：' + error.message)
        }
      }
      reader.readAsText(file)
      event.target.value = '' // 重置文件输入
    },

    exportData() {
      const jsonl = this.projects
        .map(project => JSON.stringify(project))
        .join('\n')

      const now = new Date()
      const timestamp = now.toISOString()
        .replace(/[:.]/g, '')  // 移除时间中的冒号和点
        .replace('T', '-')     // 用短横线替换T
        .replace('Z', '')      // 移除Z
      
      const projectCount = this.projects.length
      const fileName = `moyu-journal-${timestamp}-${projectCount}projects.jsonl`

      const blob = new Blob([jsonl], { type: 'text/plain' })
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a')
      a.href = url
      a.download = fileName
      a.click()
      URL.revokeObjectURL(url)
    },

    clearAll() {
      this.projects = []
      this.saveProjects()
      this.showClearConfirmDialog = false
      this.isMenuOpen = false  // 只在实际清空数据时关闭菜单
    },
    
    showClearConfirm() {
      this.showClearConfirmDialog = true
    },
    
    cancelClear() {
      this.showClearConfirmDialog = false
    },
    
    confirmClear() {
      this.clearAll()
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString('zh-CN', {
        year: 'numeric',
        month: 'numeric',
        day: 'numeric'
      })
    },
    formatDateForInput(date) {
      return new Date(date).toISOString().split('T')[0]
    },
    updateDate(event, project) {
      project.createdAt = event.target.value + 'T00:00:00.000Z'
      this.saveProjects()
    },
    saveEdit(project) {
      this.editMode = null
      this.saveProjects()
    },
    cancelEdit(project) {
      this.editMode = null
    },
    saveUsername() {
      this.editUsername = false
      localStorage.setItem('moyu-username', this.username)
      if (this.$refs.usernameInput) {
        this.$refs.usernameInput.blur()
      }
    }
  }
}
</script>

<style>
.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  position: relative;
  padding-bottom: 100px;
}

.main-content {
  flex: 1;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
}

.project-list {
  margin-bottom: 2rem;
}

.footer {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  padding: 1rem 0;
  background-color: rgba(248, 249, 250, 0.9);
  backdrop-filter: blur(10px);
  border-top: 1px solid #dadce0;
  box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.05);
  z-index: 100;
  transition: transform 0.3s ease;
}

@media (max-height: 800px) {
  .footer {
    transform: translateY(70%);
  }
  
  .footer:hover {
    transform: translateY(0);
  }
}

.footer-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
  text-align: center;
}

.footer-links {
  margin-bottom: 0.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
}

.footer-links a {
  color: #5f6368;
  text-decoration: none;
  display: flex;
  align-items: center;
  gap: 0.25rem;
  transition: color 0.3s ease;
}

.footer-links a:hover {
  color: #1a73e8;
}

.footer-links .separator {
  color: #dadce0;
}

.copyright {
  color: #5f6368;
  font-size: 0.875rem;
}

/* 工具栏样式 */
.toolbar {
  display: flex;
  justify-content: center;
  margin: 1rem 0 1.5rem;
}

.new-project {
  position: relative;
  background: none;
  border: none;
  font-size: 1.25rem;
  padding: 1.5rem 3rem;
  cursor: pointer;
  transition: all 0.3s ease;
  color: #1a73e8;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

.fish-pond {
  position: relative;
  height: 2rem;
  width: 100%;
}

.fish {
  position: absolute;
  font-size: 1.5rem;
  animation: swimAround 4s infinite ease-in-out;
  transform-origin: center;
}

.bubble {
  position: absolute;
  color: #1a73e8;
  opacity: 0.5;
  font-size: 1rem;
  animation: floatUp 2s infinite ease-in-out;
}

.bubble-1 {
  left: 30%;
  animation-delay: 0s;
}

.bubble-2 {
  left: 50%;
  animation-delay: 0.5s;
}

.bubble-3 {
  left: 70%;
  animation-delay: 1s;
}

@keyframes swimAround {
  0% {
    left: -20%;
    transform: scaleX(1) translateY(0);
  }
  25% {
    transform: scaleX(1) translateY(-5px);
  }
  49% {
    transform: scaleX(1) translateY(0);
  }
  50% {
    left: 120%;
    transform: scaleX(1) translateY(0);
  }
  51% {
    transform: scaleX(-1) translateY(0);
  }
  75% {
    transform: scaleX(-1) translateY(-5px);
  }
  99% {
    transform: scaleX(-1) translateY(0);
  }
  100% {
    left: -20%;
    transform: scaleX(1) translateY(0);
  }
}

@keyframes floatUp {
  0% {
    transform: translateY(20px);
    opacity: 0;
  }
  50% {
    opacity: 0.5;
  }
  100% {
    transform: translateY(-20px);
    opacity: 0;
  }
}

.fish-text {
  position: relative;
  z-index: 2;
  transition: transform 0.3s ease;
  font-weight: 500;
}

.fish-wave {
  color: #1a73e8;
  opacity: 0.3;
  font-size: 1.25rem;
  animation: waveFlow 2s infinite ease-in-out;
}

@keyframes waveFlow {
  0%, 100% {
    transform: translateX(-10px);
  }
  50% {
    transform: translateX(10px);
  }
}

.new-project::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: #e3f2fd;
  border-radius: 30px;
  transform: scale(0.95);
  opacity: 0;
  transition: all 0.3s ease;
  z-index: 1;
}

.new-project:hover::before {
  transform: scale(1);
  opacity: 1;
}

.new-project:hover .fish {
  animation-duration: 2s;
}

.new-project:hover .fish-text {
  transform: translateY(-3px);
}

.new-project:hover .fish-wave {
  opacity: 0.8;
}

/* 项目卡片样式 */
.project-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}

.project-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12);
  overflow: hidden;
  transition: all 0.3s ease;
  cursor: move;
}

.project-card:hover {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  transform: translateY(-2px);
}

.ghost {
  opacity: 0.5;
  background: #c8ebfb;
}

.card-header {
  padding: 16px;
  border-bottom: 1px solid #f1f3f4;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: background-color 0.3s ease;
}

.card-header.edit-mode {
  background-color: #f8f9fa;
}

.project-title {
  display: flex;
  align-items: center;
  gap: 12px;
}

.emoji-icon {
  font-size: 2.5rem;
  line-height: 1;
}

.project-title h3 {
  margin: 0;
  font-size: 1.2rem;
  color: #202124;
}

.card-actions {
  display: flex;
  gap: 0.5rem;
}

.edit-btn, .delete-btn {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s ease;
}

.edit-btn:hover {
  background-color: rgba(0, 0, 0, 0.05);
}

.delete-btn {
  color: #d93025;
}

.delete-btn:hover {
  background-color: #fee7e7;
}

.delete-btn .material-icons {
  font-size: 1.25rem;
}

/* 项目内容样式 */
.project-content {
  padding: 16px;
}

.project-tags {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin-bottom: 16px;
}

.tag {
  padding: 4px 12px;
  border-radius: 16px;
  font-size: 0.9rem;
  white-space: nowrap;
}

.tag.date {
  background-color: #e3f2fd;
  color: #1976d2;
}

.tag.status {
  background-color: #f3e5f5;
  color: #7b1fa2;
}

.tag.tech {
  background-color: #e8f5e9;
  color: #388e3c;
}

.project-notes {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin: 20px 0;
}

.note-section {
  background: #f8f9fa;
  padding: 16px;
  border-radius: 8px;
}

.note-section label {
  color: #5f6368;
  font-weight: 500;
  margin-bottom: 12px;
  display: block;
}

.note-content {
  color: #202124;
  line-height: 1.6;
}

.project-links {
  margin-top: 16px;
  border-top: 1px solid #f1f3f4;
  padding-top: 16px;
}

.link-buttons {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.link-btn {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 6px 12px;
  background: #f8f9fa;
  border: 1px solid #dadce0;
  border-radius: 4px;
  color: #1a73e8;
  text-decoration: none;
  transition: all 0.2s ease;
}

.link-btn:hover {
  background: #f1f3f4;
  border-color: #1a73e8;
}

.link-inputs {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.link-input-group {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.link-input-group label {
  font-size: 0.9rem;
  color: #5f6368;
}

.link-input {
  padding: 8px 12px;
  border: 1px solid #dadce0;
  border-radius: 4px;
  background: white;
  width: 100%;
  transition: border-color 0.2s ease;
}

.link-input:focus {
  outline: none;
  border-color: #1a73e8;
}

.link-input:hover {
  border-color: #5f6368;
}

/* 空状态样式 */
.empty-state {
  text-align: center;
  padding: 40px;
  color: #5f6368;
  font-size: 1.1rem;
}

/* 响应式设计 */
@media (max-width: 640px) {
  .container {
    padding: 16px;
  }

  .project-list {
    grid-template-columns: 1fr;
  }

  .project-notes {
    grid-template-columns: 1fr;
  }

  .project-tags {
    flex-direction: column;
  }

  .toolbar {
    flex-direction: column;
    gap: 16px;
  }

  .sort-controls {
    width: 100%;
    justify-content: center;
  }
}

.emoji-picker-wrapper {
  position: relative;
}

.emoji-trigger {
  font-size: 2.5rem;
  line-height: 1;
  cursor: pointer;
  transition: transform 0.2s ease;
}

.emoji-trigger:hover {
  transform: scale(1.1);
}

.emoji-picker-container {
  position: absolute;
  z-index: 1000;
  top: 100%;
  left: 0;
}

.emoji-picker-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: -1;
}

.title-input {
  padding: 6px 12px;
  border: 1px solid #dadce0;
  border-radius: 4px;
  background: white;
  width: 100%;
}

.status-select {
  padding: 6px 12px;
  border: 1px solid #dadce0;
  border-radius: 4px;
  background: white;
  cursor: pointer;
}

.tech-input {
  padding: 6px 12px;
  border: 1px solid #dadce0;
  border-radius: 4px;
  background: white;
  width: 100%;
}

.description {
  color: #202124;
  line-height: 1.6;
}

.description-input {
  padding: 6px 12px;
  border: 1px solid #dadce0;
  border-radius: 4px;
  background: white;
  width: 100%;
  height: 100px;
  resize: none;
}

/* 自定义 emoji picker 样式 */
:root {
  --ep-color-bg: #ffffff;
  --ep-color-border: #dadce0;
  --ep-color-sbg: #f8f9fa;
  --ep-color-active: #1a73e8;
}

.ep-container {
  border-radius: 8px !important;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06) !important;
}

.ep-categories {
  border-color: var(--ep-color-border) !important;
}

.ep-emojis {
  padding: 8px !important;
}

.ep-emoji {
  border-radius: 4px !important;
}

.ep-emoji:hover {
  background-color: var(--ep-color-sbg) !important;
}

/* 标题和菜单样式 */
header {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 1.5rem 1rem 0.5rem;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  position: relative;
}

.menu-wrapper {
  position: relative;
}

.menu-trigger {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.3s ease;
}

.menu-trigger:hover {
  background-color: rgba(0, 0, 0, 0.05);
}

.menu-dropdown {
  position: absolute;
  top: 100%;
  right: 0;
  background: white;
  border: 1px solid #dadce0;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  padding: 0.5rem;
  z-index: 1000;
  min-width: 150px;
}

.menu-dropdown button {
  display: block;
  width: 100%;
  padding: 0.5rem 1rem;
  text-align: left;
  background: none;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  color: #5f6368;
  transition: all 0.3s ease;
}

.menu-dropdown button:hover {
  background-color: #f8f9fa;
  color: #1a73e8;
}

.menu-dropdown button.danger {
  color: #d93025;
}

.menu-dropdown button.danger:hover {
  background-color: #fce8e8;
}

.menu-dropdown .delete-wrapper {
  position: relative;
  width: 100%;
}

.menu-dropdown .delete-confirm {
  position: absolute;
  right: calc(100% + 8px);
  top: 50%;
  transform: translateY(-50%);
  background: white;
  border: 1px solid rgba(217, 48, 37, 0.2);
  border-radius: 8px;
  padding: 0.5rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  animation: slideIn 0.2s ease;
  z-index: 10;
  white-space: nowrap;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(-50%) translateX(10px);
  }
  to {
    opacity: 1;
    transform: translateY(-50%) translateX(0);
  }
}

.confirm-text {
  color: #d93025;
  font-weight: 500;
  font-size: 0.875rem;
}

.confirm-btn {
  background: none;
  border: none;
  width: 24px;
  height: 24px;
  padding: 0;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.2s ease;
  line-height: 1;
}

.confirm-btn .material-icons {
  font-size: 16px;
  width: 16px;
  height: 16px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.confirm-yes {
  color: #34a853;
}

.confirm-yes:hover {
  background-color: #e6f4ea;
}

.confirm-no {
  color: #d93025;
}

.confirm-no:hover {
  background-color: #fce8e8;
}

.title-section {
  flex: 1;
  text-align: center;
  margin-bottom: 1rem;
}

.main-title {
  font-size: 2.5rem;
  font-weight: bold;
  margin-bottom: 0.25rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

/* 标题样式 */
.username {
  color: #1a73e8;
  transition: all 0.3s ease;
}

.username:hover {
  opacity: 0.8;
}

.username-input {
  font-size: 2.5rem;
  font-weight: bold;
  color: #1a73e8;
  border: none;
  border-bottom: 2px solid #1a73e8;
  background: transparent;
  padding: 0 0.5rem;
  width: auto;
  text-align: center;
}

.username-input:focus {
  outline: none;
}

.title-separator {
  color: #5f6368;
  font-weight: normal;
}

.title-main {
  background: linear-gradient(45deg, #1a73e8, #34a853);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.subtitle {
  font-size: 1rem;
  color: #5f6368;
  font-style: italic;
  margin-top: 0.25rem;
  opacity: 0.8;
}

/* 删除确认框样式 */
.delete-wrapper {
  position: relative;
  display: inline-block;
}

.delete-confirm {
  position: absolute;
  right: 0;
  top: calc(100% + 8px);
  background: white;
  padding: 8px 12px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  display: flex;
  align-items: center;
  gap: 8px;
  white-space: nowrap;
  z-index: 10;
  border: 1px solid rgba(217, 48, 37, 0.2);
}

.delete-confirm::before {
  content: '';
  position: absolute;
  top: -6px;
  right: 12px;
  transform: rotate(45deg);
  width: 12px;
  height: 12px;
  background: white;
  border-left: 1px solid rgba(217, 48, 37, 0.2);
  border-top: 1px solid rgba(217, 48, 37, 0.2);
}

.confirm-text {
  color: #5f6368;
  font-size: 0.875rem;
}

.confirm-btn {
  background: none;
  border: none;
  width: 24px;
  height: 24px;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.2s ease;
}

.confirm-btn .material-icons {
  font-size: 16px;
}

.confirm-btn.confirm-yes {
  background-color: #d93025;
  color: white;
}

.confirm-btn.confirm-yes:hover {
  background-color: #b92318;
}

.confirm-btn.confirm-no {
  color: #5f6368;
}

.confirm-btn.confirm-no:hover {
  background-color: rgba(0, 0, 0, 0.05);
}
</style>
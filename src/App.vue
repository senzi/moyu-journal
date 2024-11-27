<template>
  <div class="container">
    <header>
      <h1>摸鱼纪</h1>
      <div class="menu-wrapper">
        <button class="menu-trigger" @click="isMenuOpen = !isMenuOpen">
          <span class="dots"></span>
        </button>
        <div class="menu-dropdown" v-show="isMenuOpen" @click="isMenuOpen = false">
          <input type="file" accept=".jsonl" @change="importFile" style="display: none" ref="fileInput">
          <button @click="$refs.fileInput.click()">导入文件</button>
          <button @click="importFromURL">从URL导入</button>
          <button @click="exportData">导出数据</button>
          <button @click="clearAll" class="danger">清空数据</button>
        </div>
      </div>
    </header>

    <main>
      <div class="toolbar">
        <button class="new-project" @click="createNew">新建项目</button>
      </div>

      <div v-if="projects.length === 0" class="empty-state">
        还没有任何项目，开始创建或导入吧！
      </div>

      <draggable v-model="projects" v-else class="project-list" ghost-class="ghost" @start="drag = true"
        @end="drag = false">
        <template #item="{ element: project }">
          <div class="project-card">
            <div class="card-header" :class="{ 'edit-mode': editMode === project.id }">
              <div class="project-title">
                <div class="emoji-icon">{{ project.emoji_icon || '🚀' }}</div>
                <h3>{{ project.title || '未命名项目' }}</h3>
              </div>
              <div class="card-actions">
                <button class="edit-btn" @click="toggleEditMode(project)">
                  <span class="material-icons">{{ editMode === project.id ? 'done' : 'edit' }}</span>
                </button>
                <button v-if="editMode === project.id" class="delete-btn" @click="confirmDelete(project.id)">
                  <span class="material-icons">close</span>
                </button>
              </div>
            </div>

            <div class="project-content">
              <div class="project-tags">
                <span class="tag date">{{ formatDate(project.createdAt) }}</span>
                <span class="tag status">{{ project.status || '未设置' }}</span>
                <span class="tag tech">{{ project.techStack || '未设置' }}</span>
              </div>

              <div class="project-notes">
                <div class="note-section">
                  <label>打算怎么摸？</label>
                  <div class="note-content">{{ project.description || '暂无描述' }}</div>
                </div>
                <div class="note-section">
                  <label>吐槽</label>
                  <div class="note-content">{{ project.thoughts || '暂无记录' }}</div>
                </div>
              </div>

              <div class="project-links">
                <a v-if="project.demoLink" :href="project.demoLink" target="_blank" class="link-btn">
                  <span class="material-icons">launch</span>
                  查看Demo
                </a>
                <a v-if="project.repoLink" :href="project.repoLink" target="_blank" class="link-btn">
                  <span class="material-icons">code</span>
                  查看仓库
                </a>
              </div>
            </div>
          </div>
        </template>
      </draggable>
    </main>
  </div>
</template>

<script>
import draggable from 'vuedraggable'

export default {
  components: {
    draggable
  },

  data() {
    return {
      projects: [],
      currentProject: null,
      isMenuOpen: false,
      editMode: null,
      sortBy: 'createdAt',
      sortOrder: 'desc',
      drag: false
    }
  },

  computed: {
    sortedProjects: {
      get() {
        return this.projects.slice().sort((a, b) => {
          const factor = this.sortOrder === 'asc' ? 1 : -1
          const valueA = a[this.sortBy]
          const valueB = b[this.sortBy]
          return valueA < valueB ? -factor : factor
        })
      },
      set(value) {
        this.projects = value
        this.saveProjects()
      }
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

    confirmDelete(id) {
      this.deleteProject(id)
    },

    handleClickOutside(event) {
      const menuWrapper = document.querySelector('.menu-wrapper')
      if (menuWrapper && !menuWrapper.contains(event.target)) {
        this.isMenuOpen = false
      }
    },

    loadProjects() {
      const stored = localStorage.getItem('moyu-projects')
      if (stored) {
        this.projects = JSON.parse(stored)
      }
    },

    createNew() {
      const newProject = {
        id: crypto.randomUUID(),
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

    deleteProject(id) {
      if (confirm('确定要删除这个项目吗？')) {
        this.projects = this.projects.filter(p => p.id !== id)
        this.saveProjects()
      }
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

      const blob = new Blob([jsonl], { type: 'text/plain' })
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a')
      a.href = url
      a.download = 'moyu-journal.jsonl'
      a.click()
      URL.revokeObjectURL(url)
    },

    clearAll() {
      if (confirm('确定要清空所有数据吗？')) {
        this.projects = []
        this.saveProjects()
      }
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString('zh-CN', {
        year: 'numeric',
        month: 'numeric',
        day: 'numeric'
      })
    }
  }
}
</script>

<style>
/* 基础布局 */
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 40px;
}

h1 {
  font-size: 2rem;
  color: #1a73e8;
  margin: 0;
}

/* 菜单相关 */
.menu-wrapper {
  position: relative;
}

.menu-trigger {
  width: 40px;
  height: 40px;
  border: none;
  border-radius: 50%;
  background: #f1f3f4;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.3s ease;
}

.menu-trigger:hover {
  background: #e8eaed;
}

.dots {
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: #5f6368;
  position: relative;
}

.dots::before,
.dots::after {
  content: '';
  position: absolute;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: #5f6368;
}

.dots::before {
  left: 0;
  top: -6px;
}

.dots::after {
  left: 0;
  top: 6px;
}

.menu-dropdown {
  position: absolute;
  right: 0;
  top: 100%;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  padding: 8px;
  min-width: 160px;
  z-index: 1000;
}

.menu-dropdown button {
  display: block;
  width: 100%;
  padding: 8px 16px;
  text-align: left;
  border: none;
  background: none;
  cursor: pointer;
  color: #3c4043;
  border-radius: 4px;
  transition: background-color 0.2s ease;
}

.menu-dropdown button:hover {
  background: #f1f3f4;
}

.menu-dropdown button.danger {
  color: #d93025;
}

.menu-dropdown button.danger:hover {
  background: #fee;
}

/* 工具栏样式 */
.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.new-project {
  padding: 8px 16px;
  background: #1a73e8;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.new-project:hover {
  background: #1557b0;
}

.sort-controls {
  display: flex;
  gap: 8px;
  align-items: center;
}

.sort-controls select {
  padding: 6px 12px;
  border: 1px solid #dadce0;
  border-radius: 4px;
  background: white;
  cursor: pointer;
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
  gap: 8px;
}

.edit-btn,
.delete-btn {
  width: 36px;
  height: 36px;
  border: none;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
}

.edit-btn {
  background-color: #e3f2fd;
  color: #1976d2;
}

.edit-btn:hover {
  background-color: #bbdefb;
}

.delete-btn {
  background-color: #ffebee;
  color: #f44336;
}

.delete-btn:hover {
  background-color: #ffcdd2;
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
  display: flex;
  gap: 12px;
  margin-top: 16px;
}

.link-btn {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 6px 12px;
  border-radius: 6px;
  text-decoration: none;
  color: #1976d2;
  background-color: #e3f2fd;
  transition: background-color 0.3s ease;
}

.link-btn:hover {
  background-color: #bbdefb;
}

.material-icons {
  font-size: 20px;
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
</style>
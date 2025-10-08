<template>
  <div class="full-page-container">
    <!-- Header -->
    <header class="app-header">
      <div class="header-left">
        <span class="logo">
          <img
            src="xx"
            alt="Logo"
            class="logo-img"
          />
        </span>
      </div>

      <nav class="main-nav-centered">
        <a href="#">首页</a>
        <a href="#">找专家</a>
      </nav>

      <div class="header-right">
        <button class="language-btn">EN / 中</button>
        <button class="publish-btn">发布</button>
      </div>
    </header>

    <!-- Page Content -->
    <div class="page-content-wrapper">
      <div class="content-left-sidebar">
        <div class="context-nav">← 返回项目</div>
      </div>

      <div class="content-main-area">
        <!-- Title -->
        <div class="post-field-group title-field">
          <label class="required-label">* 标题</label>
          <div class="input-wrapper">
            <input
              type="text"
              placeholder="提出一个高品质的讨论话题，最好文字清晰，且从改善需求点，直达技术创新，引入注目..."
              v-model="postTitle"
              maxlength="200"
            />
            <span class="char-count">{{ postTitle.length }}/200</span>
          </div>
        </div>

        <!-- Details / Rich Text Editor -->
        <div class="post-field-group details-field">
          <label class="required-label">* 详情</label>
          <div ref="editor" class="rich-text-editor"></div>
        </div>

        <!-- Other Form Fields -->
        <div class="form-grid">
          <!-- Attachment -->
          <div class="row">
            <div class="label">附件</div>
            <div class="control">
              <div class="file-upload-area">
                <div class="file-btn" @click="triggerFileUpload">
                  {{ fileName || '上传附件' }}
                </div>
                <input
                  type="file"
                  ref="fileInput"
                  @change="handleFileChange"
                  style="display: none"
                />
                <div class="small">
                  *请上传认证项目的文档，说明文档等。有效期为5分钟，请尽快上传...
                </div>
                <div class="error" v-if="fileError">{{ fileError }}</div>
              </div>
            </div>
          </div>

          <!-- Project Category - MULTISELECT -->
          <div class="row">
            <div class="label">所属项目 <span class="req">*</span></div>
            <div class="control">
              <div class="multiselect-wrapper" ref="multiselectWrapper">
                <!-- Input field with inline badges -->
                <div class="multiselect-input-container" @click="toggleProject">
                  <div class="inline-badges-wrapper">
                    <div
                      class="inline-badge"
                      v-for="proj in selectedProjects"
                      :key="proj.id"
                    >
                      <span class="inline-badge-text">{{ proj.name }}</span>
                      <span
                        class="inline-badge-remove"
                        @click.stop="removeProject(proj)"
                        >×</span
                      >
                    </div>
                    <input
                      type="text"
                      class="multiselect-search-input"
                      :placeholder="selectedProjects.length === 0 ? '请选择认证项目' : ''"
                      readonly
                      @click.stop="toggleProject"
                    />
                  </div>
                  <span class="dropdown-arrow-icon">▼</span>
                </div>

                <!-- Dropdown options - HORIZONTAL LAYOUT (4 COLUMNS) -->
                <div
                  class="multiselect-dropdown-horizontal"
                  v-show="showProject"
                >
                  <div
                    class="multiselect-option-horizontal"
                    v-for="proj in projects"
                    :key="proj.id"
                    @click.stop="toggleProjectSelection(proj)"
                  >
                    <label class="checkbox-label-horizontal">
                      <input
                        type="checkbox"
                        :checked="isProjectSelected(proj)"
                        @click.stop
                      />
                      <span class="checkbox-custom-horizontal"></span>
                      <span class="option-text-horizontal"
                        >{{ proj.name }}</span
                      >
                    </label>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Experts (COMPACT HORIZONTAL LAYOUT - MULTIPLE PER ROW) -->
          <div class="row">
            <div class="label">所属专家</div>
            <div class="control">
              <div class="experts-simple-box">
                <div
                  v-if="selectedExperts.length === 0"
                  class="no-experts-placeholder"
                >
                  选择项目后，相关专家将自动显示在这里
                </div>
                <div v-else class="experts-grid">
                  <div
                    class="expert-item-compact"
                    v-for="(expert, index) in selectedExperts"
                    :key="expert.id"
                  >
                    <span class="expert-text">
                      <span class="expert-name-inline">{{ expert.name }}</span>
                      <span class="expert-id-inline">{{ expert.id }}</span>
                    </span>
                    <button
                      class="expert-remove-btn-compact"
                      @click="removeExpert(index)"
                      title="移除"
                    >
                      ×
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Custom Tags -->
          <div class="row">
            <div class="label">自定义标签</div>
            <div class="control">
              <div
                class="tag-input"
                contenteditable="true"
                placeholder="输入标签后按回车添加 (支持逗号、冒号、分号分隔)"
                @keydown.enter.prevent="addTagFromDiv($event)"
              ></div>
              <div class="tags">
                <div class="tag" v-for="(t, i) in tags" :key="i">
                  {{ t }} <span class="remove" @click="removeTag(i)">×</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Actions -->
        <div class="actions-centered">
          <button class="action-btn btn-cancel" @click="cancel">取消</button>
          <button class="action-btn btn-draft" @click="saveDraft">保存</button>
          <button class="action-btn btn-approve" @click="approveForm">
            审核
          </button>
          <button class="action-btn btn-submit" @click="submitForm">
            提交
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        postTitle: '',
        fileName: '',
        fileError: '',
        selectedProjects: [],
        showProject: false,

        projects: [
          {
            id: 1,
            name: '地理信息系统 (GIS)',
            experts: [
              { id: 'EXP-8435695', name: 'Ali Ahmed Abdallsadiq' },
              { id: 'EXP-7234561', name: 'Zhang Wei' },
              { id: 'EXP-9123456', name: 'Maria Garcia' }
            ]
          },
          {
            id: 2,
            name: '人工智能与机器学习',
            experts: [
              { id: 'EXP-5678901', name: 'John Smith' },
              { id: 'EXP-3456789', name: 'Li Ming' },
              { id: 'EXP-8901234', name: 'Sarah Johnson' },
              { id: 'EXP-4567890', name: 'Wang Fang' }
            ]
          },
          {
            id: 3,
            name: '云计算架构',
            experts: [
              { id: 'EXP-2345678', name: 'Chen Jian' },
              { id: 'EXP-6789012', name: 'Emily Brown' }
            ]
          },
          {
            id: 4,
            name: '物联网 (IoT)',
            experts: [
              { id: 'EXP-1234567', name: 'Liu Xiao' },
              { id: 'EXP-7890123', name: 'David Wilson' },
              { id: 'EXP-3456781', name: 'Anna Schmidt' }
            ]
          },
          {
            id: 5,
            name: '网络安全',
            experts: [
              { id: 'EXP-9012345', name: 'Zhang San' },
              { id: 'EXP-4567891', name: 'Michael Davis' }
            ]
          },
          {
            id: 6,
            name: '大数据分析',
            experts: [
              { id: 'EXP-5678902', name: 'Wang Wei' },
              { id: 'EXP-2345679', name: 'Jessica Taylor' },
              { id: 'EXP-8901235', name: 'Carlos Rodriguez' }
            ]
          },
          {
            id: 7,
            name: '区块链技术',
            experts: [
              { id: 'EXP-6789013', name: 'Li Lei' },
              { id: 'EXP-1234568', name: 'Sophie Martin' }
            ]
          }
        ],

        selectedExperts: [],
        tags: [],
        content: '',
        isUploadingImage: false,
        quill: null,
        imageUploadApiUrl: 'xx?',
        imageBaseUrl: 'xx'
      }
    },

    mounted() {
      this.$nextTick(() => {
        const quillCss = document.createElement('link')
        quillCss.rel = 'stylesheet'
        quillCss.href = 'https://cdn.jsdelivr.net/npm/quill@2.0.3/dist/quill.snow.min.css'
        document.head.appendChild(quillCss)

        const quillScript = document.createElement('script')
        quillScript.src = 'https://cdn.jsdelivr.net/npm/quill@2.0.3/dist/quill.min.js'
        quillScript.async = true
        quillScript.defer = true
        quillScript.onload = () => {
          this.initializeQuill()
        }
        document.head.appendChild(quillScript)

        document.addEventListener('click', this.closeProjectDropdown)
      })
    },

    beforeDestroy() {
      document.removeEventListener('click', this.closeProjectDropdown)
    },

    methods: {
      initializeQuill() {
        const Link = Quill.import('formats/link');

        Link.sanitize = function(url) {
          let protocol = url.slice(0, url.indexOf(':'));

          if (this.PROTOCOL_WHITELIST.indexOf(protocol) === -1) {
            url = 'https://' + url;
          }

          let anchor = document.createElement('a');
          anchor.href = url;
          protocol = anchor.href.slice(0, anchor.href.indexOf(':'));

          return this.PROTOCOL_WHITELIST.indexOf(protocol) > -1 ? url : this.PROTOCOL_WHITELIST[0] + url;
        };

        const Font = Quill.import('formats/font');
        Font.whitelist = [
          'sans-serif',
          'serif',
          'monospace',
          'microsoft-yahei',
          'arial',
          'times-new-roman'
        ];
        Quill.register(Font, true);

        const toolbarOptions = [
          ['bold', 'italic', 'underline', 'strike'],
          [{ 'header': [1, 2, 3, 4, 5, 6, false] }],
          [{ 'list': 'ordered' }, { 'list': 'bullet' }],
          [{ 'script': 'sub' }, { 'script': 'super' }],
          [{ 'indent': '-1' }, { 'indent': '+1' }],
          [{ 'direction': 'rtl' }],
          [{ 'size': ['small', false, 'large', 'huge'] }],
          [{ 'color': [] }, { 'background': [] }],
          [{ 'font': Font.whitelist }],
          [{ 'align': [] }],
          ['clean'],
          ['link', 'image']
        ];

        this.quill = new Quill(this.$refs.editor, {
          modules: {
            toolbar: {
              container: toolbarOptions,
              handlers: {
                image: this.imageHandler
              }
            }
          },
          theme: 'snow',
          placeholder: 'Compose an epic...'
        });

        this.quill.on('text-change', () => {
          this.content = this.quill.root.innerHTML;
        });
      },

      imageHandler() {
        const input = document.createElement('input')
        input.setAttribute('type', 'file')
        input.setAttribute('accept', 'image/*')
        input.click()

        input.onchange = async () => {
          const file = input.files[0]
          if (file) {
            this.isUploadingImage = true
            try {
              const imageUrl = await this.uploadImageToApi(file)
              const range = this.quill.getSelection(true)
              this.quill.insertEmbed(range.index, 'image', imageUrl)
              this.quill.setSelection(range.index + 1)
            } catch (error) {
              console.error('Image upload failed:', error)
              alert('无法上传图片，请重试')
            } finally {
              this.isUploadingImage = false
            }
          }
        }
      },

      toggleProject() {
        this.showProject = !this.showProject
      },

      isProjectSelected(proj) {
        return this.selectedProjects.some(p => p.id === proj.id)
      },

      toggleProjectSelection(proj) {
        const index = this.selectedProjects.findIndex(p => p.id === proj.id)

        if (index > -1) {
          this.selectedProjects.splice(index, 1)
          this.removeExpertsByProject(proj)
        } else {
          this.selectedProjects.push(proj)
          this.addExpertsByProject(proj)
        }
      },

      removeProject(proj) {
        const index = this.selectedProjects.findIndex(p => p.id === proj.id)
        if (index > -1) {
          this.selectedProjects.splice(index, 1)
          this.removeExpertsByProject(proj)
        }
      },

      addExpertsByProject(proj) {
        proj.experts.forEach(expert => {
          if (!this.selectedExperts.some(e => e.id === expert.id)) {
            this.selectedExperts.push(expert)
          }
        })
      },

      removeExpertsByProject(proj) {
        const expertIdsToRemove = proj.experts.map(e => e.id)
        const otherSelectedProjects = this.selectedProjects.filter(p => p.id !== proj.id)
        const expertIdsInOtherProjects = []

        otherSelectedProjects.forEach(p => {
          p.experts.forEach(e => {
            if (!expertIdsInOtherProjects.includes(e.id)) {
              expertIdsInOtherProjects.push(e.id)
            }
          })
        })

        this.selectedExperts = this.selectedExperts.filter(expert => {
          return !expertIdsToRemove.includes(expert.id) || expertIdsInOtherProjects.includes(expert.id)
        })
      },

      closeProjectDropdown(e) {
        if (
          this.showProject &&
          this.$refs.multiselectWrapper &&
          !this.$refs.multiselectWrapper.contains(e.target)
        ) {
          this.showProject = false
        }
      },

      removeExpert(index) {
        this.selectedExperts.splice(index, 1)
      },

      async uploadImageToApi(file) {
        const formData = new FormData()
        formData.append('image', file)
        formData.append('sourceType', 'fromWord')

        try {
          const response = await fetch(this.imageUploadApiUrl, {
            method: 'POST',
            credentials: 'include',
            body: formData
          })

          if (!response.ok) {
            const errorBody = await response.text()
            throw new Error(`Upload failed with status: ${response.status}. Response: ${errorBody}`)
          }

          const data = await response.json()
          if (!data.url) {
            throw new Error('API response for image upload is missing the "url" property.')
          }

          return this.imageBaseUrl + data.url
        } catch (error) {
          console.error('Image upload API error:', error)
          throw error
        }
      },

      triggerFileUpload() {
        this.$refs.fileInput.click()
      },

      handleFileChange(e) {
        const file = e.target.files[0]
        if (!file) return
        if (file.size > 150 * 1024 * 1024) {
          this.fileError = '文件过大'
          return
        }
        this.fileName = file.name
        this.fileError = ''
      },

      addTagFromDiv(e) {
        const rawValue = e.target.innerText.trim()

        if (!rawValue) {
          e.target.innerText = ''
          return
        }

        const separatedTags = rawValue.split(/[,;:]/)

        separatedTags.forEach(tag => {
          const trimmedTag = tag.trim()
          if (trimmedTag && !this.tags.includes(trimmedTag)) {
            this.tags.push(trimmedTag)
          }
        })

        e.target.innerText = ''
      },

      removeTag(i) {
        this.tags.splice(i, 1)
      },

      submitForm() {
        if (this.selectedProjects.length === 0) {
          alert('请选择所属项目')
          return
        }
        if (!this.postTitle.trim()) {
          alert('请输入标题')
          return
        }

        const formData = {
          postTitle: this.postTitle,
          content: this.content,
          fileName: this.fileName,
          projects: this.selectedProjects.map(p => p.name),
          tags: this.tags,
          experts: this.selectedExperts
        }

        console.log('Form Submitted:', formData)
        alert('提交成功!')
      },

      approveForm() {
        console.log('Approve action')
        alert('审核通过!')
      },

      saveDraft() {
        console.log('Save draft')
        alert('草稿已保存!')
      },

      cancel() {
        if (confirm('确定要取消吗？未保存的更改将丢失。')) {
          localStorage.removeItem('richTextContent')
          location.reload()
        }
      }
    }
  }
</script>

<style scoped>
  /* Font dropdown labels */
  .ql-snow .ql-picker.ql-font .ql-picker-label[data-value='sans-serif']::before,
  .ql-snow .ql-picker.ql-font .ql-picker-item[data-value='sans-serif']::before {
    content: 'Sans Serif';
  }

  .ql-snow .ql-picker.ql-font .ql-picker-label[data-value='serif']::before,
  .ql-snow .ql-picker.ql-font .ql-picker-item[data-value='serif']::before {
    content: 'Serif';
    font-family: 'serif';
  }

  .ql-snow .ql-picker.ql-font .ql-picker-label[data-value='monospace']::before,
  .ql-snow .ql-picker.ql-font .ql-picker-item[data-value='monospace']::before {
    content: 'Monospace';
    font-family: 'monospace';
  }

  .ql-snow .ql-picker.ql-font .ql-picker-label[data-value='microsoft-yahei']::before,
  .ql-snow .ql-picker.ql-font .ql-picker-item[data-value='microsoft-yahei']::before {
    content: 'Microsoft YaHei';
    font-family: 'Microsoft YaHei', sans-serif;
  }

  .ql-snow .ql-picker.ql-font .ql-picker-label[data-value='arial']::before,
  .ql-snow .ql-picker.ql-font .ql-picker-item[data-value='arial']::before {
    content: 'Arial';
    font-family: 'Arial', sans-serif;
  }

  .ql-snow .ql-picker.ql-font .ql-picker-label[data-value='times-new-roman']::before,
  .ql-snow .ql-picker.ql-font .ql-picker-item[data-value='times-new-roman']::before {
    content: 'Times New Roman';
    font-family: 'Times New Roman', serif;
  }

  .ql-font-serif {
    font-family: 'serif';
  }

  .ql-font-monospace {
    font-family: 'monospace';
  }

  .ql-font-microsoft-yahei {
    font-family: 'Microsoft YaHei', sans-serif;
  }

  .ql-font-arial {
    font-family: 'Arial', sans-serif;
  }

  .ql-font-times-new-roman {
    font-family: 'Times New Roman', serif;
  }

  /* --- General & Layout --- */
  .full-page-container {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
    background-color: #f8fafc;
    min-height: 100vh;
  }

  .app-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 24px;
    height: 64px;
    background-color: #fff;
    border-bottom: 1px solid #e2e8f0;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
    position: relative;
  }

  .header-left {
    display: flex;
    align-items: center;
    flex: 1;
  }

  .logo-img {
      width: 140px;
    height: 50px;
  }

  /* CENTERED NAVIGATION */
  .main-nav-centered {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    align-items: center;
    gap: 40px;
  }

  .main-nav-centered a {
    text-decoration: none;
    color: #4a5568;
    font-size: 14px;
    font-weight: 500;
    transition: color 0.2s;
  }

  .main-nav-centered a:hover {
    color: #2b6cb0;
  }

  .header-right {
    display: flex;
    align-items: center;
    gap: 20px;
    flex: 1;
    justify-content: flex-end;
  }

  .language-btn,
  .publish-btn {
    height: 36px;
    padding: 0 16px;
    border-radius: 6px;
    border: 1px solid #cbd5e0;
    background-color: #fff;
    cursor: pointer;
    font-size: 14px;
    font-weight: 600;
  }

  .publish-btn {
    background-color: #d0021b;
    color: #fff;
    border-color: #d0021b;
  }

  .page-content-wrapper {
    display: flex;
    max-width: 1280px;
    margin: 0 auto;
    padding: 24px;
    gap: 24px;
  }

  .content-left-sidebar {
    width: 200px;
    flex-shrink: 0;
  }

  .content-main-area {
    width: 100%;
    background-color: #fff;
    padding: 32px;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  }

  .context-nav {
    font-size: 13px;
    color: #718096;
    margin-bottom: 16px;
    cursor: pointer;
  }

  .context-nav:hover {
    color: #2b6cb0;
  }

  /* --- Form Fields --- */
  .post-field-group {
    margin-bottom: 24px;
  }

  .post-field-group label {
    display: block;
    margin-bottom: 8px;
    font-weight: 600;
    font-size: 14px;
    color: #2d3748;
  }

  .required-label::before {
    content: '';
  }

  .input-wrapper {
    position: relative;
  }

  .input-wrapper input,
  .input-wrapper textarea {
    width: 100%;
    padding: 10px 12px;
    border: 1px solid #cbd5e0;
    border-radius: 6px;
    font-size: 14px;
    color: #2d3748;
    transition: all 0.2s;
  }

  .input-wrapper input:focus,
  .input-wrapper textarea:focus {
    border-color: #4299e1;
    outline: none;
    box-shadow: 0 0 0 1px #4299e1;
  }

  .char-count {
    position: absolute;
    right: 10px;
    bottom: 8px;
    font-size: 12px;
    color: #a0aec0;
  }

  .rich-text-editor {
    border: 1px solid #cbd5e0;
    border-radius: 8px;
    background-color: #fff;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  }

  .form-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 20px;
    margin-bottom: 32px;
    margin-top: 32px;
  }

  .row {
    display: flex;
    align-items: start;
    gap: 16px;
  }

  .label {
    font-weight: 600;
    font-size: 14px;
    color: #2d3748;
    padding-top: 8px;
    min-width: 100px;
    flex-shrink: 0;
  }

  .control {
    flex: 1;
  }

  .file-upload-area {
    width: 100%;
  }

  .file-btn {
    display: inline-block;
    padding: 10px 16px;
    border: 1px solid #cbd5e0;
    border-radius: 6px;
    cursor: pointer;
    background-color: #fff;
    transition: all 0.2s;
    font-size: 14px;
  }

  .file-btn:hover {
    background-color: #f7fafc;
    border-color: #4299e1;
  }

  /* --- MULTISELECT --- */
  .multiselect-wrapper {
    position: relative;
    width: 100%;
  }

  .multiselect-input-container {
    display: flex;
    align-items: center;
    min-height: 42px;
    padding: 4px 8px;
    border: 1px solid #cbd5e0;
    border-radius: 6px;
    background-color: #fff;
    cursor: pointer;
    transition: all 0.2s;
    gap: 6px;
  }

  .multiselect-input-container:hover {
    border-color: #4299e1;
  }

  .inline-badges-wrapper {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 6px;
    flex: 1;
    min-width: 0;
  }

  .inline-badge {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    padding: 4px 10px;
    background-color: #e6f2ff;
    border: 1px solid #4299e1;
    border-radius: 14px;
    font-size: 12px;
    color: #2c5282;
    font-weight: 500;
    white-space: nowrap;
    max-width: 200px;
    transition: all 0.2s;
  }

  .inline-badge:hover {
    background-color: #bee3f8;
  }

  .inline-badge-text {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .inline-badge-remove {
    cursor: pointer;
    font-size: 16px;
    font-weight: 700;
    color: #4299e1;
    line-height: 1;
    padding: 0 2px;
    transition: color 0.2s;
  }

  .inline-badge-remove:hover {
    color: #c53030;
  }

  .multiselect-search-input {
    border: none;
    outline: none;
    flex: 1;
    min-width: 80px;
    font-size: 14px;
    color: #2d3748;
    background: transparent;
    padding: 4px;
  }

  .multiselect-search-input::placeholder {
    color: #a0aec0;
  }

  .dropdown-arrow-icon {
    font-size: 10px;
    color: #a0aec0;
    flex-shrink: 0;
  }

  .multiselect-dropdown-horizontal {
    position: absolute;
    top: calc(100% + 4px);
    left: 0;
    right: 0;
    background-color: #fff;
    border: 1px solid #cbd5e0;
    border-radius: 8px;
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.12);
    z-index: 100;
    padding: 10px;
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 6px;
    max-height: 200px;
    overflow-y: auto;
  }

  .multiselect-option-horizontal {
    display: flex;
    align-items: center;
    transition: background-color 0.15s;
    border-radius: 6px;
  }

  .multiselect-option-horizontal:hover {
    background-color: #f7fafc;
  }

  .checkbox-label-horizontal {
    display: flex;
    align-items: center;
    padding: 6px 8px;
    cursor: pointer;
    width: 100%;
    position: relative;
  }

  .checkbox-label-horizontal input[type='checkbox'] {
    position: absolute;
    opacity: 0;
    cursor: pointer;
  }

  .checkbox-custom-horizontal {
    width: 16px;
    height: 16px;
    border: 2px solid #cbd5e0;
    border-radius: 4px;
    margin-right: 8px;
    position: relative;
    transition: all 0.2s;
    flex-shrink: 0;
    background-color: #fff;
  }

  .checkbox-label-horizontal input[type='checkbox']:checked ~ .checkbox-custom-horizontal {
    background-color: #4299e1;
    border-color: #4299e1;
  }

  .checkbox-label-horizontal input[type='checkbox']:checked ~ .checkbox-custom-horizontal::after {
    content: '✓';
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    color: #fff;
    font-size: 11px;
    font-weight: 700;
  }

  .option-text-horizontal {
    font-size: 12px;
    color: #2d3748;
    flex: 1;
    line-height: 1.3;
  }

  /* --- EXPERTS --- */
  .experts-simple-box {
    width: 100%;
    min-height: 80px;
    padding: 10px;
    border: 1px solid #cbd5e0;
    border-radius: 6px;
    background-color: #fff;
    transition: all 0.2s;
  }

  .experts-simple-box:hover {
    border-color: #a0aec0;
  }

  .no-experts-placeholder {
    color: #a0aec0;
    font-size: 13px;
    text-align: center;
    padding: 24px 16px;
    font-style: italic;
  }

  .experts-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
  }

  .expert-item-compact {
    display: inline-flex;
    align-items: center;
    justify-content: space-between;
    gap: 8px;
    padding: 6px 10px;
    background-color: #f7fafc;
    border: 1px solid #e2e8f0;
    border-radius: 6px;
    transition: all 0.2s;
    max-width: calc(50% - 4px);
    min-width: 200px;
  }

  .expert-item-compact:hover {
    background-color: #edf2f7;
    border-color: #cbd5e0;
  }

  .expert-text {
    display: flex;
    align-items: center;
    gap: 8px;
    flex: 1;
    min-width: 0;
  }

  .expert-name-inline {
    font-size: 13px;
    font-weight: 600;
    color: #2d3748;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .expert-id-inline {
    font-size: 12px;
    color: #718096;
    white-space: nowrap;
    flex-shrink: 0;
  }

  .expert-remove-btn-compact {
    width: 20px;
    height: 20px;
    border-radius: 4px;
    background-color: transparent;
    border: 1px solid #e2e8f0;
    color: #718096;
    font-size: 16px;
    font-weight: 700;
    line-height: 1;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s;
    padding: 0;
    flex-shrink: 0;
  }

  .expert-remove-btn-compact:hover {
    background-color: #fee;
    border-color: #fc8181;
    color: #c53030;
  }

  .small {
    font-size: 12px;
    color: #718096;
    margin-top: 6px;
  }

  .error {
    font-size: 12px;
    color: #c53030;
    margin-top: 4px;
  }

  .tag-input {
    min-height: 40px;
    padding: 8px 12px;
    border: 1px solid #cbd5e0;
    border-radius: 6px;
    background-color: #fff;
    outline: none;
    cursor: text;
  }

  .tag-input:empty:before {
    content: attr(placeholder);
    color: #a0aec0;
  }

  .tag-input:focus {
    border-color: #4299e1;
    box-shadow: 0 0 0 1px #4299e1;
  }

  .tags {
    margin-top: 8px;
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
  }

  .tag {
    padding: 6px 12px;
    background-color: #edf2f7;
    border-radius: 16px;
    font-size: 13px;
    color: #4a5568;
    display: flex;
    align-items: center;
  }

  .tag .remove {
    margin-left: 8px;
    cursor: pointer;
    font-weight: bold;
    color: #a0aec0;
  }

  .tag .remove:hover {
    color: #c53030;
  }

  .req {
    color: #c53030;
  }

  /* --- ACTIONS --- */
  .actions-centered {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 12px;
    margin-top: 32px;
    padding-top: 24px;
    border-top: 1px solid #e2e8f0;
  }

  .action-btn {
    min-width: 100px;
    height: 40px;
    padding: 0 24px;
    border-radius: 6px;
    border: 1px solid;
    cursor: pointer;
    font-weight: 600;
    font-size: 14px;
    transition: all 0.2s;
  }

  .btn-cancel,
  .btn-draft,
  .btn-approve {
    background-color: #fff;
    color: #718096;
    border-color: #cbd5e0;
  }

  .btn-cancel:hover,
  .btn-draft:hover,
  .btn-approve:hover {
    background-color: #f7fafc;
    border-color: #a0aec0;
    color: #4a5568;
  }

  .btn-submit {
    background-color: #d0021b;
    color: #fff;
    border-color: #d0021b;
  }

  .btn-submit:hover {
    background-color: #b00118;
    border-color: #b00118;
  }
</style>

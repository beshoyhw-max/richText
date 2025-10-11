<template>
  <div class="full-page-container">
    <!-- Header -->
    <div class="header">
      <div class="logo">
        <img
          src="xx"
          alt="expert community"
        />
      </div>
      <div class="navigation">
        <a href="#">首页</a>
        <a href="#" class="active">找专家</a>
      </div>
      <div class="user-actions">
        <a href="#" class="lang-switch">EN / CN</a>
        <button class="new-post-btn">发布</button>
      </div>
    </div>

    <!-- Page Content -->
    <div class="page-content-wrapper">
      <div class="content-main-area">
        <div class="context-nav">← 返回项目</div>
        
        <!-- Title Field -->
        <div class="form-row">
          <label class="form-label required-label">标题</label>
          <div class="form-control">
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
        </div>

        <!-- Details / Rich Text Editor -->
        <div class="form-row">
          <label class="form-label required-label">详情</label>
          <div class="form-control">
            <div ref="editor" class="rich-text-editor"></div>
          </div>
        </div>

        <!-- Attachment -->
        <div class="form-row">
          <label class="form-label">附件</label>
          <div class="form-control">
            <div class="file-upload-area">
              <button class="file-btn" @click="triggerFileUpload">
                上传附件
              </button>
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
              <div class="attachment-list" v-if="attachments.length > 0">
                <div
                  class="attachment-item"
                  v-for="(file, index) in attachments"
                  :key="file.docId"
                >
                  <span class="attachment-name">{{ file.docName }}</span>
                  <div class="attachment-actions">
                    <button
                      class="action-btn-link"
                      @click="downloadAttachment(file.docId)"
                    >
                      下载
                    </button>
                    <button
                      class="action-btn-link"
                      @click="deleteAttachment(index)"
                    >
                      删除
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Project Category - MULTISELECT -->
        <div class="form-row">
          <label class="form-label required-label">所属项目</label>
          <div class="form-control">
            <div class="multiselect-wrapper" ref="multiselectWrapper">
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

              <div
                class="multiselect-dropdown-horizontal"
                v-show="showProject"
              >
                <div
                  class="multiselect-option-horizontal"
                  v-for="proj in projects"
                  :key="proj.id"
                >
                  <label class="checkbox-label-horizontal" @click.stop>
                    <input
                      type="checkbox"
                      :checked="isProjectSelected(proj)"
                      @change="toggleProjectSelection(proj)"
                    />
                    <span class="checkbox-custom-horizontal"></span>
                    <span class="option-text-horizontal">{{ proj.name }}</span>
                  </label>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Experts -->
        <div class="form-row">
          <label class="form-label">所属专家</label>
          <div class="form-control">
            <div class="experts-simple-box">
              <div
                v-if="isLoadingExperts"
                class="loading-experts-placeholder"
              >
                正在加载专家信息...
              </div>
              <div
                v-else-if="selectedExperts.length === 0"
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
        <div class="form-row">
          <label class="form-label">自定义标签</label>
          <div class="form-control">
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

        <!-- Actions -->
        <div class="actions-centered">
          <button class="action-btn btn-cancel" @click="cancel">取消</button>
          <button class="action-btn btn-draft" @click="saveDraft">保存</button>
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
        projects: [],
        selectedExperts: [],
        tags: [],
        content: '',
        isUploadingImage: false,
        isLoadingExperts: false,
        quill: null,
        imageUploadApiUrl: 'xx?',
        imageBaseUrl: 'xx',
        attachments: []
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
      
      // Load project categories
      this.loadProjectCategories();
    },

    beforeDestroy() {
      document.removeEventListener('click', this.closeProjectDropdown)
    },

    methods: {
      loadProjectCategories() {
        // Updated categories as requested
        this.projects = [
          { id: 1, name: '普惠安全', category: '普惠安全' },
          { id: 2, name: '普惠教育', category: '普惠教育' },
          { id: 3, name: '普惠连接', category: '普惠连接' },
          { id: 4, name: '普惠政务', category: '普惠政务' },
          { id: 5, name: '普惠能源', category: '普惠能源' },
          { id: 6, name: '云智OS', category: '云智OS' },
          { id: 7, name: '云与算力', category: '云与算力' }
        ];
      },

      /**
       * Parse expert data from API response
       * Extracts name and ID from expertUserCn field (format: "王正兵 00837587")
       */
      parseExpertData(expertItem) {
        const expertUserCn = expertItem.expertUserCn || '';
        
        // Split by space to separate name and ID
        const parts = expertUserCn.trim().split(/\s+/);
        
        let name = '';
        let id = '';
        
        if (parts.length >= 2) {
          // Last part is the ID (number)
          id = parts[parts.length - 1];
          // Everything before the last part is the name
          name = parts.slice(0, parts.length - 1).join(' ');
        } else if (parts.length === 1) {
          // If only one part, treat it as name
          name = parts[0];
        }
        
        return {
          id: id || expertUserCn, // Use full string as fallback
          name: name || expertUserCn, // Use full string as fallback
          originalData: expertItem // Keep original data if needed
        };
      },

      async fetchExpertsByCategory(category) {
        try {
          this.isLoadingExperts = true;
          
          const response = await fetch('xx/findList ', {
            method: 'POST',
            credentials: 'include', // Include credentials (cookies, authorization headers)
            headers: {
              'Content-Type': 'application/json',
              // Add any additional authentication headers if needed
              // 'Authorization': `Bearer ${yourToken}` // Uncomment and add token if needed
            },
            body: JSON.stringify({
              expertCategory: category
            })
          });

          if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
          }

          const data = await response.json();
          
          // Parse the response - adjust based on your actual API response structure
          let expertsArray = [];
          
          // Check different possible response structures
          if (Array.isArray(data)) {
            expertsArray = data;
          } else if (data.data && Array.isArray(data.data)) {
            expertsArray = data.data;
          } else if (data.experts && Array.isArray(data.experts)) {
            expertsArray = data.experts;
          } else if (data.result && Array.isArray(data.result)) {
            expertsArray = data.result;
          }
          
          // Parse each expert to extract name and ID from expertUserCn
          return expertsArray.map(expertItem => this.parseExpertData(expertItem));
          
        } catch (error) {
          console.error('Error fetching experts:', error);
          alert('获取专家信息失败，请重试');
          return [];
        } finally {
          this.isLoadingExperts = false;
        }
      },

      getToken() {
        return new Promise((resolve, reject) => {
          Hae.View.loading();
          this.$service.network
            .post('//xxathorization', {})
            .then((res) => {
              let token = res.data.authorization;
              resolve(token);
            })
            .catch(err => {
              Hae.View.loadingClose();
              reject(err);
            });
        });
      },

      uploadEdmApi(file) {
        return new Promise((resolve, reject) => {
          this.getToken().then((token) => {
            var formData = new FormData();
            formData.append('serverType', 'upload');
            formData.append('origin', 'xx');
            formData.append('updateFile', false);
            formData.append('multipartFile', file);

            this.$service.network
              .put('xx/documents', formData, {
                headers: {
                  ,
                }
              })
              .then((res) => {
                Hae.View.loadingClose();
                resolve(res.data.result);
              })
              .catch(err => {
                Hae.View.loadingClose();
                reject(err);
              });
          });
        });
      },

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
        const input = document.createElement('input');
        input.setAttribute('type', 'file');
        input.setAttribute('accept', 'image/*');
        input.click();

        input.onchange = async () => {
          const file = input.files[0];
          if (file) {
            this.isUploadingImage = true;
            try {
              const result = await this.uploadEdmApi(file);
              const imageUrl = `xx/preview/${result.docId}/source`;
              const range = this.quill.getSelection(true);
              this.quill.insertEmbed(range.index, 'image', imageUrl);
              this.quill.setSelection(range.index + 1);
            } catch (error) {
              console.error('Image upload failed:', error);
              alert('无法上传图片，请重试');
            } finally {
              this.isUploadingImage = false;
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

      async toggleProjectSelection(proj) {
        const index = this.selectedProjects.findIndex(p => p.id === proj.id)

        if (index > -1) {
          // Deselecting project
          this.selectedProjects.splice(index, 1)
          this.removeExpertsByProject(proj)
        } else {
          // Selecting project
          this.selectedProjects.push(proj)
          await this.addExpertsByProject(proj)
        }
      },

      removeProject(proj) {
        const index = this.selectedProjects.findIndex(p => p.id === proj.id)
        if (index > -1) {
          this.selectedProjects.splice(index, 1)
          this.removeExpertsByProject(proj)
        }
      },

      async addExpertsByProject(proj) {
        // Fetch experts from API based on project category
        const experts = await this.fetchExpertsByCategory(proj.category);
        
        // Add fetched experts to selectedExperts
        experts.forEach(expert => {
          // Avoid duplicates by checking if expert ID already exists
          if (!this.selectedExperts.some(e => e.id === expert.id)) {
            this.selectedExperts.push(expert)
          }
        })
      },

      removeExpertsByProject(proj) {
        // When a project is deselected, we need to remove its experts
        // but keep experts that belong to other selected projects
        
        // If no projects are selected, clear all experts
        if (this.selectedProjects.length === 0) {
          this.selectedExperts = []
          return
        }
        
        // For a more sophisticated approach, you'd need to track which experts
        // belong to which projects. For now, we'll reload all experts from
        // remaining selected projects
        this.reloadAllExperts()
      },

      async reloadAllExperts() {
        // Clear current experts
        this.selectedExperts = []
        
        // Fetch experts for all selected projects
        for (const proj of this.selectedProjects) {
          const experts = await this.fetchExpertsByCategory(proj.category)
          experts.forEach(expert => {
            if (!this.selectedExperts.some(e => e.id === expert.id)) {
              this.selectedExperts.push(expert)
            }
          })
        }
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

      triggerFileUpload() {
        this.$refs.fileInput.click()
      },

      async handleFileChange(e) {
        const file = e.target.files[0];
        if (!file) return;

        if (file.size > 150 * 1024 * 1024) {
          this.fileError = '文件过大';
          return;
        }
        this.fileError = '';

        try {
          const result = await this.uploadEdmApi(file);
          this.attachments.push({
            docId: result.docId,
            docName: result.docName
          });
        } catch (error) {
          console.error('Attachment upload failed:', error);
          this.fileError = '附件上传失败，请重试';
        } finally {
          this.$refs.fileInput.value = '';
        }
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

      deleteAttachment(index) {
        this.attachments.splice(index, 1);
      },

      async downloadAttachment(docId) {
        try {
          const token = await this.getToken();
          const paramsObj = {
            'doc_version': 'V1',
            'doc_id': docId,
            'wm_type': '',
            'EDM-Authorization': token
          };
          let queryParam = '';
          for (const key in paramsObj) {
            queryParam += `&${key}=${paramsObj[key]}`;
          }
          const url = `xx/download?dummy=1${queryParam}`;
          window.open(url, '_blank');
          Hae.View.loadingClose();
        } catch (error) {
          console.error('Failed to get token for download:', error);
          alert('无法生成下载链接，请重试');
        }
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
  body {
    background: white;
  }

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

  /* ========================================
     QUILL EDITOR RESPONSIVE STYLES
     ======================================== */

  /* Main editor container */
  .rich-text-editor {
    border: 0.1vw solid #cbd5e0;
    border-radius: 0.5vw;
    background-color: #fff;
    box-shadow: 0 0.1vw 0.3vw rgba(0, 0, 0, 0.05);
    min-height: 30vh;
    display: flex;
    flex-direction: column;
  }

  /* Quill toolbar */
  .rich-text-editor :deep(.ql-toolbar) {
    border: none;
    border-bottom: 0.1vw solid #cbd5e0;
    background-color: #f7fafc;
    padding: 0.8vh 1vw;
    border-radius: 0.5vw 0.5vw 0 0;
    font-size: 0.9vw;
  }

  /* Toolbar buttons */
  .rich-text-editor :deep(.ql-toolbar button) {
    width: 1.8vw !important;
    height: 1.8vw !important;
    padding: 0.3vw;
    margin: 0 0.15vw;
    border-radius: 0.3vw;
    transition: all 0.2s;
  }

  .rich-text-editor :deep(.ql-toolbar button:hover) {
    background-color: #e2e8f0;
  }

  .rich-text-editor :deep(.ql-toolbar button.ql-active) {
    background-color: #4299e1;
    color: white;
  }

  /* Toolbar button SVG icons */
  .rich-text-editor :deep(.ql-toolbar button svg) {
    width: 1vw !important;
    height: 1vw !important;
  }

  /* Toolbar dropdown selects */
  .rich-text-editor :deep(.ql-toolbar select) {
    height: 2vh;
    font-size: 0.8vw;
    padding: 0.2vh 0.5vw;
    border: 0.1vw solid #cbd5e0;
    border-radius: 0.3vw;
    margin: 0 0.3vw;
    min-width: 5vw;
  }

  .rich-text-editor :deep(.ql-toolbar select:focus) {
    outline: none;
    border-color: #4299e1;
  }

  /* Dropdown arrows */
  .rich-text-editor :deep(.ql-toolbar .ql-picker-label) {
    padding: 0.3vh 0.5vw;
    font-size: 0.8vw;
    border: 0.1vw solid #cbd5e0;
    border-radius: 0.3vw;
    min-width: 5vw;
    height: 2.5vh;
    display: flex;
    align-items: center;
  }

  .rich-text-editor :deep(.ql-toolbar .ql-picker-label::before) {
    font-size: 0.8vw;
  }

  .rich-text-editor :deep(.ql-toolbar .ql-picker-label svg) {
    width: 0.8vw;
    height: 0.8vw;
    margin-left: 0.5vw;
  }

  /* Picker options dropdown */
  .rich-text-editor :deep(.ql-toolbar .ql-picker-options) {
    font-size: 0.85vw;
    padding: 0.5vh 0;
    border: 0.1vw solid #cbd5e0;
    border-radius: 0.4vw;
    box-shadow: 0 0.3vw 1vw rgba(0, 0, 0, 0.1);
    max-height: 25vh;
    overflow-y: auto;
  }

  .rich-text-editor :deep(.ql-toolbar .ql-picker-item) {
    padding: 0.5vh 1vw;
    font-size: 0.85vw;
  }

  .rich-text-editor :deep(.ql-toolbar .ql-picker-item:hover) {
    background-color: #f7fafc;
  }

  /* Quill editor content area */
  .rich-text-editor :deep(.ql-container) {
    border: none;
    font-size: 0.95vw;
    min-height: 25vh;
    flex: 1;
    border-radius: 0 0 0.5vw 0.5vw;
  }

  .rich-text-editor :deep(.ql-editor) {
    padding: 1.5vh 1.5vw;
    min-height: 25vh;
    font-size: 0.95vw;
    line-height: 1.6;
    overflow-y: auto;
  }

  .rich-text-editor :deep(.ql-editor.ql-blank::before) {
    font-size: 0.9vw;
    color: #a0aec0;
    font-style: italic;
    left: 1.5vw;
    right: 1.5vw;
  }

  /* Content inside editor - make all text responsive */
  .rich-text-editor :deep(.ql-editor p),
  .rich-text-editor :deep(.ql-editor ol),
  .rich-text-editor :deep(.ql-editor ul),
  .rich-text-editor :deep(.ql-editor pre),
  .rich-text-editor :deep(.ql-editor blockquote),
  .rich-text-editor :deep(.ql-editor h1),
  .rich-text-editor :deep(.ql-editor h2),
  .rich-text-editor :deep(.ql-editor h3),
  .rich-text-editor :deep(.ql-editor h4),
  .rich-text-editor :deep(.ql-editor h5),
  .rich-text-editor :deep(.ql-editor h6) {
    margin: 0;
    padding: 0;
    margin-bottom: 1vh;
  }

  /* Responsive heading sizes */
  .rich-text-editor :deep(.ql-editor h1) {
    font-size: 2vw;
    line-height: 1.3;
  }

  .rich-text-editor :deep(.ql-editor h2) {
    font-size: 1.7vw;
    line-height: 1.3;
  }

  .rich-text-editor :deep(.ql-editor h3) {
    font-size: 1.4vw;
    line-height: 1.4;
  }

  .rich-text-editor :deep(.ql-editor h4) {
    font-size: 1.2vw;
    line-height: 1.4;
  }

  .rich-text-editor :deep(.ql-editor h5) {
    font-size: 1vw;
    line-height: 1.5;
  }

  .rich-text-editor :deep(.ql-editor h6) {
    font-size: 0.9vw;
    line-height: 1.5;
  }

  /* Lists */
  .rich-text-editor :deep(.ql-editor ul),
  .rich-text-editor :deep(.ql-editor ol) {
    padding-left: 2vw;
  }

  .rich-text-editor :deep(.ql-editor li) {
    font-size: 0.95vw;
    margin-bottom: 0.5vh;
    line-height: 1.6;
  }

  /* Blockquote */
  .rich-text-editor :deep(.ql-editor blockquote) {
    border-left: 0.3vw solid #cbd5e0;
    padding-left: 1vw;
    margin-left: 0;
    margin-right: 0;
    font-size: 1vw;
    font-style: italic;
    color: #718096;
  }

  /* Code blocks */
  .rich-text-editor :deep(.ql-editor pre) {
    background-color: #f7fafc;
    border: 0.1vw solid #e2e8f0;
    border-radius: 0.4vw;
    padding: 1vh 1vw;
    font-size: 0.85vw;
    overflow-x: auto;
  }

  .rich-text-editor :deep(.ql-editor code) {
    background-color: #f7fafc;
    padding: 0.2vh 0.4vw;
    border-radius: 0.3vw;
    font-size: 0.85vw;
  }

  /* Links */
  .rich-text-editor :deep(.ql-editor a) {
    color: #4299e1;
    text-decoration: underline;
    font-size: inherit;
  }

  /* Images */
  .rich-text-editor :deep(.ql-editor img) {
    max-width: 100%;
    height: auto;
    border-radius: 0.4vw;
    margin: 1vh 0;
  }

  /* Font sizes - custom responsive sizes */
  .rich-text-editor :deep(.ql-editor .ql-size-small) {
    font-size: 0.7vw;
  }

  .rich-text-editor :deep(.ql-editor .ql-size-large) {
    font-size: 1.2vw;
  }

  .rich-text-editor :deep(.ql-editor .ql-size-huge) {
    font-size: 1.5vw;
  }

  /* Scrollbar styling for editor */
  .rich-text-editor :deep(.ql-editor::-webkit-scrollbar) {
    width: 0.6vw;
  }

  .rich-text-editor :deep(.ql-editor::-webkit-scrollbar-track) {
    background: #f7fafc;
    border-radius: 0.3vw;
  }

  .rich-text-editor :deep(.ql-editor::-webkit-scrollbar-thumb) {
    background: #cbd5e0;
    border-radius: 0.3vw;
  }

  .rich-text-editor :deep(.ql-editor::-webkit-scrollbar-thumb:hover) {
    background: #a0aec0;
  }

  /* Tooltip styling */
  .rich-text-editor :deep(.ql-tooltip) {
    font-size: 0.85vw;
    padding: 0.8vh 1vw;
    border-radius: 0.4vw;
    box-shadow: 0 0.3vw 1vw rgba(0, 0, 0, 0.1);
    border: 0.1vw solid #cbd5e0;
  }

  .rich-text-editor :deep(.ql-tooltip input) {
    font-size: 0.85vw;
    padding: 0.5vh 0.8vw;
    border: 0.1vw solid #cbd5e0;
    border-radius: 0.3vw;
  }

  .rich-text-editor :deep(.ql-tooltip button) {
    font-size: 0.85vw;
    padding: 0.5vh 1vw;
    border-radius: 0.3vw;
  }

  /* Color picker */
  .rich-text-editor :deep(.ql-color-picker .ql-picker-options) {
    width: 12vw;
    padding: 0.5vh 0.5vw;
  }

  .rich-text-editor :deep(.ql-color-picker .ql-picker-item) {
    width: 1.2vw;
    height: 1.2vw;
    border-radius: 0.2vw;
    margin: 0.2vw;
  }

  /* Align buttons */
  .rich-text-editor :deep(.ql-align .ql-picker-item) {
    width: 1.5vw;
    height: 1.5vw;
  }

  /* --- General & Layout --- */
  .full-page-container {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
    background-color: #fff;
    min-height: 100vh;
  }

  .logo {
    font-size: 1.8vw;
    font-weight: bold;
  }

  .header {
    display: flex;
    justify-content: space-between;
    padding: 0.7vw;
    background-color: #fff;
    border-bottom: 0.1vw solid #e0e0e0;
    position: fixed;
    z-index: 999;
    width: 99.2vw;
    height: 5vw;
    margin-top: -0.62vw;
    align-items: center;
  }

  .logo img {
    width: 11.61vw;
    height: 3.85vw;
    margin-left: 1.5vw;
  }

  .navigation {
    width: 8.54vw;
    display: flex;
    margin-top: 0.9vw;
    height: 2.71vw;
  }

  .navigation a {
    margin: 0vw 0.9vw;
    text-decoration: none;
    color: #333;
    font-size: 1vw;
    font-weight: bold;
    padding-bottom: 0.1vw;
  }

  .navigation a.active {
    color: rgba(215, 123, 107, 1);
    border-bottom: 0.2vw solid rgba(215, 123, 107, 1);
  }

  .user-actions {
    display: flex;
    align-items: center;
    margin-right: 4.3vw;
    width: 8.96vw;
    height: 2.08vw;
    gap: 1.67vw;
  }

  .lang-switch {
    text-decoration: none;
    color: #666;
    font-size: 0.9vw;
  }

  .new-post-btn {
    background-color: #E96E72;
    color: white;
    border: none;
    padding: 0.83vw;
    border-radius: 0.42vw;
    cursor: pointer;
    font-weight: 500;
    font-size: 0.83vw;
    transition: all 0.2s;
    width: 4.53vw;
    height: 2.08vw;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    display: flex;
    gap: 0.52vw;
  }

  .new-post-btn:hover {
    background-color: #7e4647;
  }

  .page-content-wrapper {
    display: flex;
    max-width: 80vw;
    margin: 0 auto;
    padding: 7vw 2vw 5vw 2vw;
    gap: 1.5vw;
  }

  .content-main-area {
    width: 100%;
    background-color: #fff;
    padding: 2vw;
  }

  .context-nav {
    font-size: 0.9vw;
    color: #718096;
    margin-bottom: 2vh;
    cursor: pointer;
  }

  .context-nav:hover {
    color: #2b6cb0;
  }

  /* --- TWO-COLUMN FORM LAYOUT --- */
  .form-row {
    display: grid;
    grid-template-columns: 12vw 1fr;
    gap: 2vw;
    margin-bottom: 2.5vh;
    align-items: start;
    min-height: 5vh;
  }

  .form-label {
    font-weight: 600;
    font-size: 0.95vw;
    color: #2d3748;
    padding-top: 1vh;
    text-align: center;
    white-space: nowrap;
  }

  .required-label::before {
    content: '* ';
    color: #c53030;
  }

  .form-control {
    width: 100%;
  }

  /* --- Input Styling --- */
  .input-wrapper {
    position: relative;
    width: 100%;
  }

  .input-wrapper input,
  .input-wrapper textarea {
    width: 100%;
    padding: 1vh 1vw;
    border: 0.1vw solid #cbd5e0;
    border-radius: 0.4vw;
    font-size: 0.9vw;
    color: #2d3748;
    transition: all 0.2s;
    min-height: 4vh;
  }

  .input-wrapper input:focus,
  .input-wrapper textarea:focus {
    border-color: #4299e1;
    outline: none;
    box-shadow: 0 0 0 0.15vw rgba(66, 153, 225, 0.3);
  }

  .char-count {
    position: absolute;
    right: 1vw;
    bottom: 1vh;
    font-size: 0.75vw;
    color: #a0aec0;
  }

  /* --- File Upload --- */
  .file-upload-area {
    width: 100%;
  }

  .file-btn {
    display: inline-block;
    padding: 1vh 1.2vw;
    border: 0.1vw solid #cbd5e0;
    border-radius: 0.4vw;
    cursor: pointer;
    background-color: #fff;
    transition: all 0.2s;
    font-size: 0.9vw;
  }

  .file-btn:hover {
    background-color: #f7fafc;
    border-color: #4299e1;
  }

  .small {
    font-size: 0.75vw;
    color: #718096;
    margin-top: 0.8vh;
  }

  .error {
    font-size: 0.75vw;
    color: #c53030;
    margin-top: 0.5vh;
  }

  .attachment-list {
    margin-top: 1.5vh;
  }

  .attachment-item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 1vh 1vw;
    background-color: #f7fafc;
    border: 0.1vw solid #e2e8f0;
    border-radius: 0.4vw;
    margin-bottom: 1vh;
  }

  .attachment-name {
    font-size: 0.9vw;
    color: #2d3748;
  }

  .attachment-actions {
    display: flex;
    gap: 1vw;
  }

  .action-btn-link {
    background: none;
    border: none;
    color: #4299e1;
    cursor: pointer;
    font-size: 0.85vw;
    text-decoration: underline;
    padding: 0;
  }

  .action-btn-link:hover {
    color: #2b6cb0;
  }

  /* --- MULTISELECT --- */
  .multiselect-wrapper {
    position: relative;
    width: 100%;
  }

  .multiselect-input-container {
    display: flex;
    align-items: center;
    min-height: 4vh;
    padding: 0.5vh 0.8vw;
    border: 0.1vw solid #cbd5e0;
    border-radius: 0.4vw;
    background-color: #fff;
    cursor: pointer;
    transition: all 0.2s;
    gap: 0.5vw;
  }

  .multiselect-input-container:hover {
    border-color: #4299e1;
  }

  .inline-badges-wrapper {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 0.5vw;
    flex: 1;
    min-width: 0;
  }

  .inline-badge {
    display: inline-flex;
    align-items: center;
    gap: 0.5vw;
    padding: 0.4vh 0.8vw;
    background-color: #e6f2ff;
    border: 0.1vw solid #4299e1;
    border-radius: 1vw;
    font-size: 0.8vw;
    color: #2c5282;
    font-weight: 500;
    white-space: nowrap;
    max-width: 15vw;
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
    font-size: 1.1vw;
    font-weight: 700;
    color: #4299e1;
    line-height: 1;
    padding: 0 0.2vw;
    transition: color 0.2s;
  }

  .inline-badge-remove:hover {
    color: #c53030;
  }

  .multiselect-search-input {
    border: none;
    outline: none;
    flex: 1;
    min-width: 6vw;
    font-size: 0.9vw;
    color: #2d3748;
    background: transparent;
    padding: 0.4vh;
  }

  .multiselect-search-input::placeholder {
    color: #a0aec0;
  }

  .dropdown-arrow-icon {
    font-size: 0.7vw;
    color: #a0aec0;
    flex-shrink: 0;
  }

  .multiselect-dropdown-horizontal {
    position: absolute;
    top: calc(100% + 0.5vh);
    left: 0;
    right: 0;
    background-color: #fff;
    border: 0.1vw solid #cbd5e0;
    border-radius: 0.5vw;
    box-shadow: 0 0.3vw 1vw rgba(0, 0, 0, 0.12);
    z-index: 100;
    padding: 1vh 0.8vw;
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 0.5vw;
    max-height: 25vh;
    overflow-y: auto;
  }

  .multiselect-option-horizontal {
    display: flex;
    align-items: center;
    transition: background-color 0.15s;
    border-radius: 0.4vw;
  }

  .multiselect-option-horizontal:hover {
    background-color: #f7fafc;
  }

  .checkbox-label-horizontal {
    display: flex;
    align-items: center;
    padding: 0.6vh 0.6vw;
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
    width: 1.1vw;
    height: 1.1vw;
    border: 0.15vw solid #cbd5e0;
    border-radius: 0.3vw;
    margin-right: 0.6vw;
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
    font-size: 0.75vw;
    font-weight: 700;
  }

  .option-text-horizontal {
    font-size: 0.8vw;
    color: #2d3748;
    flex: 1;
    line-height: 1.3;
  }

  /* --- EXPERTS --- */
  .experts-simple-box {
    width: 100%;
    min-height: 10vh;
    padding: 1vh 1vw;
    border: 0.1vw solid #cbd5e0;
    border-radius: 0.4vw;
    background-color: #fff;
    transition: all 0.2s;
  }

  .experts-simple-box:hover {
    border-color: #a0aec0;
  }

  .loading-experts-placeholder {
    color: #4299e1;
    font-size: 0.85vw;
    text-align: center;
    padding: 3vh 1vw;
    font-style: italic;
  }

  .no-experts-placeholder {
    color: #a0aec0;
    font-size: 0.85vw;
    text-align: center;
    padding: 3vh 1vw;
    font-style: italic;
  }

  .experts-grid {
    display: flex;
    flex-wrap: wrap;
    gap: 0.8vw;
  }

  .expert-item-compact {
    display: inline-flex;
    align-items: center;
    justify-content: space-between;
    gap: 0.6vw;
    padding: 0.7vh 0.8vw;
    background-color: #f7fafc;
    border: 0.1vw solid #e2e8f0;
    border-radius: 0.4vw;
    transition: all 0.2s;
    max-width: calc(50% - 0.4vw);
    min-width: 14vw;
  }

  .expert-item-compact:hover {
    background-color: #edf2f7;
    border-color: #cbd5e0;
  }

  .expert-text {
    display: flex;
    align-items: center;
    gap: 0.6vw;
    flex: 1;
    min-width: 0;
  }

  .expert-name-inline {
    font-size: 0.85vw;
    font-weight: 600;
    color: #2d3748;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .expert-id-inline {
    font-size: 0.75vw;
    color: #718096;
    white-space: nowrap;
    flex-shrink: 0;
  }

  .expert-remove-btn-compact {
    width: 1.5vw;
    height: 1.5vw;
    border-radius: 0.3vw;
    background-color: transparent;
    border: 0.1vw solid #e2e8f0;
    color: #718096;
    font-size: 1.1vw;
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

  /* --- Tags --- */
  .tag-input {
    min-height: 5vh;
    padding: 1vh 1vw;
    border: 0.1vw solid #cbd5e0;
    border-radius: 0.4vw;
    background-color: #fff;
    outline: none;
    cursor: text;
    font-size: 0.9vw;
  }

  .tag-input:empty:before {
    content: attr(placeholder);
    color: #a0aec0;
  }

  .tag-input:focus {
    border-color: #4299e1;
    box-shadow: 0 0 0 0.15vw rgba(66, 153, 225, 0.3);
  }

  .tags {
    margin-top: 1vh;
    display: flex;
    flex-wrap: wrap;
    gap: 0.6vw;
  }

  .tag {
    padding: 0.6vh 1vw;
    background-color: #edf2f7;
    border-radius: 1vw;
    font-size: 0.85vw;
    color: #4a5568;
    display: flex;
    align-items: center;
  }

  .tag .remove {
    margin-left: 0.6vw;
    cursor: pointer;
    font-weight: bold;
    color: #a0aec0;
  }

  .tag .remove:hover {
    color: #c53030;
  }

  /* --- ACTIONS --- */
  .actions-centered {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 1vw;
    margin-top: 4vh;
    padding-top: 3vh;
    border-top: 0.1vw solid #e2e8f0;
  }

  .action-btn {
    min-width: 7vw;
    height: 4vh;
    padding: 0 1.8vw;
    border-radius: 0.4vw;
    border: 0.1vw solid;
    cursor: pointer;
    font-weight: 600;
    font-size: 0.9vw;
    transition: all 0.2s;
  }

  .btn-cancel,
  .btn-draft {
    background-color: #fff;
    color: #718096;
    border-color: #cbd5e0;
  }

  .btn-cancel:hover,
  .btn-draft:hover {
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

  /* ========================================
     RESPONSIVE BREAKPOINTS
     ======================================== */

  /* Tablet and smaller desktops */
  @media (max-width: 1024px) {
    .rich-text-editor :deep(.ql-toolbar) {
      padding: 1vh 1.5vw;
    }

    .rich-text-editor :deep(.ql-toolbar button) {
      width: 2.5vw !important;
      height: 2.5vw !important;
    }

    .rich-text-editor :deep(.ql-toolbar button svg) {
      width: 1.5vw !important;
      height: 1.5vw !important;
    }

    .rich-text-editor :deep(.ql-toolbar select),
    .rich-text-editor :deep(.ql-toolbar .ql-picker-label) {
      font-size: 1vw;
      min-width: 7vw;
    }

    .rich-text-editor :deep(.ql-editor) {
      font-size: 1.2vw;
    }
  }

  /* Mobile tablets */
  @media (max-width: 768px) {
    .form-row {
      grid-template-columns: 1fr;
      gap: 1vh;
    }

    .form-label {
      text-align: left;
      padding-top: 0;
    }

    .multiselect-dropdown-horizontal {
      grid-template-columns: repeat(2, 1fr);
    }

    .rich-text-editor :deep(.ql-toolbar) {
      padding: 1.5vh 2vw;
    }

    .rich-text-editor :deep(.ql-toolbar button) {
      width: 4vw !important;
      height: 4vw !important;
      margin: 0 0.3vw;
    }

    .rich-text-editor :deep(.ql-toolbar button svg) {
      width: 2.5vw !important;
      height: 2.5vw !important;
    }

    .rich-text-editor :deep(.ql-toolbar select),
    .rich-text-editor :deep(.ql-toolbar .ql-picker-label) {
      font-size: 1.5vw;
      min-width: 10vw;
      height: 3.5vh;
    }

    .rich-text-editor :deep(.ql-editor) {
      font-size: 1.8vw;
      padding: 2vh 2.5vw;
    }

    .rich-text-editor :deep(.ql-editor h1) {
      font-size: 3vw;
    }

    .rich-text-editor :deep(.ql-editor h2) {
      font-size: 2.5vw;
    }

    .rich-text-editor :deep(.ql-editor h3) {
      font-size: 2.2vw;
    }
  }

  /* Small mobile phones */
  @media (max-width: 480px) {
    .rich-text-editor {
      min-height: 35vh;
    }

    .rich-text-editor :deep(.ql-toolbar) {
      padding: 2vh 3vw;
    }

    .rich-text-editor :deep(.ql-toolbar button) {
      width: 6vw !important;
      height: 6vw !important;
      margin: 0 0.5vw;
    }

    .rich-text-editor :deep(.ql-toolbar button svg) {
      width: 3.5vw !important;
      height: 3.5vw !important;
    }

    .rich-text-editor :deep(.ql-toolbar select),
    .rich-text-editor :deep(.ql-toolbar .ql-picker-label) {
      font-size: 2.5vw;
      min-width: 15vw;
      height: 4vh;
    }

    .rich-text-editor :deep(.ql-editor) {
      font-size: 2.5vw;
      padding: 2.5vh 3vw;
    }

    .rich-text-editor :deep(.ql-editor h1) {
      font-size: 4.5vw;
    }

    .rich-text-editor :deep(.ql-editor h2) {
      font-size: 4vw;
    }

    .rich-text-editor :deep(.ql-editor h3) {
      font-size: 3.5vw;
    }
  }
</style>


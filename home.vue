
<template>
  <div class="full-page-container">
    <!-- Header -->
    <header class="app-header">
      <div class="header-left">
        <span class="logo">
          <img
            src="https://f/b94a41833dbdc8f942daacfef75877d2"
            alt="Logo"
            class="logo-img"
          />
        </span>
        <nav class="main-nav">
          <a href="#">Home</a>
          <a href="#">Expert Board</a>
        </nav>
      </div>
      <div class="header-right">
        <button class="search-btn"><i class="icon-search"></i> Search</button>
        <button class="publish-btn">Publish</button>
      </div>
    </header>

    <!-- Page Content -->
    <div class="page-content-wrapper">
      <div class="content-left-sidebar">
        <div class="context-nav">Discussion / Create Post</div>
      </div>

      <div class="content-main-area">
        <!-- Title -->
        <div class="post-field-group title-field">
          <label class="required-label">Title</label>
          <div class="input-wrapper">
            <input
              type="text"
              placeholder="Enter a catchy title"
              v-model="postTitle"
              maxlength="200"
            />
            <span class="char-count">{{ postTitle.length }}/200</span>
          </div>
        </div>

        <!-- Summary -->
        <div class="post-field-group summary-field">
          <label>Summary</label>
          <div class="input-wrapper">
            <textarea
              placeholder="Enter a brief summary"
              v-model="postSummary"
              maxlength="500"
            ></textarea>
            <span class="char-count">{{ postSummary.length }}/500</span>
          </div>
        </div>

        <!-- Rich Text Editor -->
        <div class="rich-text-editor">
          <div class="toolbar">
            <div class="toolbar-group">
              <button
                @click="format('bold')"
                :class="{ active: isFormatActive('bold') }"
                title="Bold (Ctrl+B)"
              >
                <strong>B</strong>
              </button>
              <button
                @click="format('italic')"
                :class="{ active: isFormatActive('italic') }"
                title="Italic (Ctrl+I)"
              >
                <em>I</em>
              </button>
              <button
                @click="format('underline')"
                :class="{ active: isFormatActive('underline') }"
                title="Underline (Ctrl+U)"
              >
                <u>U</u>
              </button>
              <button
                @click="format('strikeThrough')"
                :class="{ active: isFormatActive('strikeThrough') }"
                title="Strikethrough"
              >
                <s>S</s>
              </button>
            </div>
            <div class="toolbar-separator"></div>
            <div class="toolbar-group">
              <select @change="setFontSize($event)" class="font-size-select">
                <option value="">Size</option>
                <option value="1">8pt</option>
                <option value="2">10pt</option>
                <option value="3">12pt</option>
                <option value="4">14pt</option>
                <option value="5">18pt</option>
                <option value="6">24pt</option>
                <option value="7">36pt</option>
              </select>
            </div>
            <div class="toolbar-separator"></div>
            <div class="toolbar-group">
              <button
                @click="format('justifyLeft')"
                :class="{ active: isFormatActive('justifyLeft') }"
                title="Align Left"
              >
                ⬅
              </button>
              <button
                @click="format('justifyCenter')"
                :class="{ active: isFormatActive('justifyCenter') }"
                title="Align Center"
              >
                ⬌
              </button>
              <button
                @click="format('justifyRight')"
                :class="{ active: isFormatActive('justifyRight') }"
                title="Align Right"
              >
                ➡
              </button>
              <button
                @click="format('justifyFull')"
                :class="{ active: isFormatActive('justifyFull') }"
                title="Justify"
              >
                ☰
              </button>
            </div>
            <div class="toolbar-separator"></div>
            <div class="toolbar-group">
              <button
                @click="format('insertUnorderedList')"
                :class="{ active: isFormatActive('insertUnorderedList') }"
                title="Bullet List"
              >
                •
              </button>
              <button
                @click="format('insertOrderedList')"
                :class="{ active: isFormatActive('insertOrderedList') }"
                title="Numbered List"
              >
                1.
              </button>
              <button @click="format('outdent')" title="Decrease Indent">
                ⇤
              </button>
              <button @click="format('indent')" title="Increase Indent">
                ⇥
              </button>
            </div>
            <div class="toolbar-separator"></div>
            <div class="toolbar-group">
              <input
                type="color"
                @change="setTextColor($event)"
                title="Text Color"
                class="color-picker"
              />
              <input
                type="color"
                @change="setBackgroundColor($event)"
                title="Background Color"
                class="color-picker"
                value="#ffffff"
              />
            </div>
            <div class="toolbar-separator"></div>
            <div class="toolbar-group">
              <button @click="insertLink" title="Insert Link">🔗</button>
              <button @click="insertImage" title="Insert Image">🖼</button>
              <button
                @click="format('formatBlock','<blockquote>')"
                title="Quote"
              >
                "
              </button>
              <button @click="format('formatBlock','<pre>')" title="Code">
                &lt;/&gt;
              </button>
            </div>
            <div class="toolbar-separator"></div>
            <div class="toolbar-group">
              <button @click="format('undo')" title="Undo (Ctrl+Z)">↶</button>
              <button @click="format('redo')" title="Redo (Ctrl+Y)">↷</button>
            </div>
            <div class="toolbar-separator"></div>
            <div class="toolbar-group">
              <button @click="clearAllFormatting" title="Clear Formatting">
                ✖
              </button>
            </div>
          </div>
          <div
            ref="editor"
            class="editor-content"
            :class="{ empty: isEmpty }"
            contenteditable="true"
            @input="handleInput"
            @keydown="handleKeydown"
            @paste="handlePaste"
            @mouseup="updateToolbarState"
            @keyup="updateToolbarState"
            v-html="content"
            :data-placeholder="placeholder"
            spellcheck="true"
            aria-label="Rich text editor"
          ></div>
          <div class="status-bar">
            <span>Words: {{ wordCount }}</span>
            <span>Characters: {{ charCount }}</span>
            <span>{{ savedStatus }}</span>
          </div>
        </div>

        <!-- Other Form Fields -->
        <div class="form-grid">
          <div class="row">
            <div class="label">Attachment</div>
            <div class="control">
              <div class="file-btn" @click="triggerFileUpload">
                {{ fileName || 'Click to upload file' }}
              </div>
              <input
                type="file"
                ref="fileInput"
                @change="handleFileChange"
                style="display:none"
              />
              <div class="small">Max file size: 150MB</div>
              <div class="error" v-if="fileError">{{ fileError }}</div>
            </div>
          </div>
          <div class="row">
            <div class="label">Domain <span class="req">*</span></div>
            <div class="control">
              <div class="select-div" @click="toggleDomain">
                {{ domain || 'Select a domain' }}
                <div class="options" v-show="showDomain">
                  <div
                    class="option"
                    v-for="d in domains"
                    :key="d"
                    @click.stop="selectDomain(d)"
                  >
                    {{ d }}
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="row">
            <div class="label">Custom Tags</div>
            <div class="control">
              <div
                class="tag-input"
                contenteditable="true"
                @keydown.enter.prevent="addTagFromDiv($event)"
              ></div>
              <div class="tags">
                <div class="tag" v-for="(t,i) in tags" :key="i">
                  {{ t }} <span class="remove" @click="removeTag(i)">×</span>
                </div>
              </div>
            </div>
          </div>
          <div class="row">
            <div class="label">Co-submitters</div>
            <div class="control">
              <div
                class="tag-input"
                contenteditable="true"
                @keydown.enter.prevent="addCoFromDiv($event)"
              ></div>
              <div class="tags">
                <div class="tag" v-for="(c,i) in coSubmitters" :key="i">
                  {{ c }} <span class="remove" @click="removeCo(i)">×</span>
                </div>
              </div>
            </div>
          </div>
          <div class="row">
            <div class="label">Other</div>
            <div class="control inline">
              <div
                class="toggle"
                @click="hasPatent=!hasPatent"
                :class="{active: hasPatent}"
              >
                Has Patent {{ hasPatent ? '✅' : '❌' }}
              </div>
              <div
                class="toggle"
                @click="hasDemo=!hasDemo"
                style="margin-left:10px"
                :class="{active: hasDemo}"
              >
                Has Demo {{ hasDemo ? '✅' : '❌' }}
              </div>
            </div>
          </div>
          <div class="row">
            <div class="label">Visibility <span class="req">*</span></div>
            <div class="control inline">
              <div
                class="toggle-radio"
                :class="{active:scope==='public'}"
                @click="scope='public'"
              >
                Public
              </div>
              <div
                class="toggle-radio"
                :class="{active:scope==='private'}"
                @click="scope='private'"
              >
                Private
              </div>
            </div>
          </div>
        </div>

        <!-- Actions -->
        <div class="actions">
          <div class="btn primary" @click="submitForm">Submit</div>
          <div class="btn" @click="saveDraft">Save Draft</div>
          <div class="btn" @click="cancel">Cancel</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        // Form data
        postTitle: '',
        postSummary: '',
        fileName: '',
        fileError: '',
        domain: '',
        showDomain: false,
        domains: ['AI', 'Cloud', 'IoT', 'Security'],
        tags: [],
        hasPatent: false,
        hasDemo: false,
        coSubmitters: [],
        scope: 'public',

        // Rich text editor
        content: '',
        isEmpty: true,
        placeholder: 'Start writing your content here...',
        savedStatus: 'Ready',
        autoSaveTimer: null,
        wordCount: 0,
        charCount: 0,
        shortcutHandler: null,
        editorObserver: null, // For the permanent text direction fix
      };
    },
    mounted() {
      this.$nextTick(() => {
        this.loadSavedContent();
        if (this.content) this.$refs.editor.innerHTML = this.content;
        this.updateToolbarState();
        this.startAutoSave();
        this.addShortcuts();
        document.addEventListener('click', this.closeDomainDropdown);

        // --- PERMANENT TEXT DIRECTION FIX ---
        this.setupDirectionObserver();
      });
    },
    beforeDestroy() {
      if (this.autoSaveTimer) clearInterval(this.autoSaveTimer);
      if (this.shortcutHandler) document.removeEventListener('keydown', this.shortcutHandler);
      document.removeEventListener('click', this.closeDomainDropdown);
      // Disconnect the observer when the component is destroyed
      if (this.editorObserver) this.editorObserver.disconnect();
    },

    methods: {
      // --- DEFINITIVE TEXT DIRECTION FIX ---
      setupDirectionObserver() {
        const editorNode = this.$refs.editor;
        if (!editorNode) return;

        const forceLtr = (node) => {
          if (node.nodeType === 1) { // Ensure it's an element
              if (node.style.direction !== 'ltr') node.style.direction = 'ltr';
              if (node.style.textAlign !== 'left' && node.style.textAlign !== 'center' && node.style.textAlign !== 'right' && node.style.textAlign !== 'justify') node.style.textAlign = 'left';
              if (node.getAttribute('dir') !== 'ltr') node.setAttribute('dir', 'ltr');
          }
        };

        forceLtr(editorNode); // Force it on the main editor once initially

        this.editorObserver = new MutationObserver((mutationsList) => {
          this.editorObserver.disconnect(); // Stop observing to prevent infinite loops

          for (const mutation of mutationsList) {
            if (mutation.type === 'attributes') {
              forceLtr(mutation.target);
            }
            mutation.addedNodes.forEach(forceLtr);
          }

          this.editorObserver.observe(editorNode, {
            attributes: true,
            childList: true,
            subtree: true,
            attributeFilter: ['style', 'dir'],
          });
        });

        this.editorObserver.observe(editorNode, {
          attributes: true,
          childList: true,
          subtree: true,
          attributeFilter: ['style', 'dir'],
        });
      },

      // --- CLIENT-SIDE IMAGE RESIZING & COMPRESSION ---
      async resizeAndCompressImage(file, maxWidth = 1024, quality = 0.8) {
        return new Promise((resolve, reject) => {
          const reader = new FileReader();
          reader.readAsDataURL(file);
          reader.onload = (event) => {
            const img = new Image();
            img.src = event.target.result;
            img.onload = () => {
              const canvas = document.createElement('canvas');
              let { width, height } = img;

              if (width > maxWidth) {
                height = (height * maxWidth) / width;
                width = maxWidth;
              }

              canvas.width = width;
              canvas.height = height;
              const ctx = canvas.getContext('2d');
              ctx.drawImage(img, 0, 0, width, height);

              const compressedDataUrl = canvas.toDataURL('image/jpeg', quality);
              resolve(compressedDataUrl);
            };
            img.onerror = (error) => reject(error);
          };
          reader.onerror = (error) => reject(error);
        });
      },

      // --- UPDATED IMAGE INSERTION LOGIC ---
      async insertImage() {
        const input = document.createElement('input');
        input.type = 'file';
        input.accept = 'image/*';
        input.onchange = async (e) => {
          const file = e.target.files[0];
          if (!file) return;

          try {
            const resizedImageBase64 = await this.resizeAndCompressImage(file);
            this.insertHTML(
              `<img src="${resizedImageBase64}" class="resizable-image" alt="${file.name}" />`
            );
          } catch (error) {
            console.error("Image processing failed:", error);
            alert("Could not process image. Please try a different file.");
          }
        };
        input.click();
      },

      // --- UPDATED CLEAR FORMATTING ---
      clearAllFormatting() {
        if (confirm("Are you sure you want to clear all content and formatting?")) {
          this.$refs.editor.innerHTML = '';
          this.content = '';
          this.syncFromEditor();
          this.$refs.editor.focus();
        }
      },

      // --- PASTE AS PLAIN TEXT ---
      handlePaste(event) {
        event.preventDefault();
        const text = (event.clipboardData || window.clipboardData).getData('text/plain');
        document.execCommand('insertText', false, text);
      },

      // Standard Editor Methods
      format(cmd, value = null) {
        document.execCommand(cmd, false, value);
        this.syncFromEditor();
        this.$refs.editor.focus();
      },
      isFormatActive(cmd) {
          if (typeof document.queryCommandState !== 'undefined') {
              try { return document.queryCommandState(cmd); } catch(e) { return false; }
          }
          return false;
      },
      setFontSize(event) {
        if (event.target.value) this.format('fontSize', event.target.value);
        event.target.value = '';
      },
      setTextColor(event) { this.format('foreColor', event.target.value); },
      setBackgroundColor(event) { this.format('hiliteColor', event.target.value); },
      insertLink() {
        const url = prompt('Enter URL:');
        if (url) this.format('createLink', url);
      },
      insertHTML(html) {
        document.execCommand('insertHTML', false, html);
        this.syncFromEditor();
      },
      handleInput() { this.syncFromEditor(); },
      handleKeydown(event) {
        if (event.key === 'Tab') {
          event.preventDefault();
          this.format(event.shiftKey ? 'outdent' : 'indent');
        }
      },
      updateToolbarState() {
        const editor = this.$refs.editor;
        if (!editor) return;
        const text = editor.innerText || '';
        this.content = editor.innerHTML;
        this.charCount = text.length;
        this.wordCount = text.trim() ? text.trim().split(/\s+/).length : 0;
        this.isEmpty = !text.trim() && !editor.querySelector('img');
        this.savedStatus = 'Modified';
        this.$forceUpdate();
      },
      syncFromEditor() { this.$nextTick(() => this.updateToolbarState()); },
      startAutoSave() { this.autoSaveTimer = setInterval(() => this.saveContent(), 30000); },
      saveContent() {
        localStorage.setItem('richTextContent', this.content);
        this.savedStatus = 'Saved';
        setTimeout(() => { if (this.savedStatus === 'Saved') this.savedStatus = 'Ready'; }, 2000);
      },
      loadSavedContent() {
        const saved = localStorage.getItem('richTextContent');
        if (saved) {
          this.content = saved;
          this.syncFromEditor();
        }
      },
      addShortcuts() {
        this.shortcutHandler = (e) => {
          if ((e.ctrlKey || e.metaKey) && this.$refs.editor === document.activeElement) {
            switch (e.key.toLowerCase()) {
              case 'b': e.preventDefault(); this.format('bold'); break;
              case 'i': e.preventDefault(); this.format('italic'); break;
              case 'u': e.preventDefault(); this.format('underline'); break;
              case 's': e.preventDefault(); this.saveContent(); break;
            }
          }
        };
        document.addEventListener('keydown', this.shortcutHandler);
      },

      // Other Form Methods
      triggerFileUpload() { this.$refs.fileInput.click(); },
      handleFileChange(e) {
        const file = e.target.files[0];
        if (!file) return;
        if (file.size > 150 * 1024 * 1024) { this.fileError = 'File too large'; return; }
        this.fileName = file.name; this.fileError = '';
      },
      toggleDomain() { this.showDomain = !this.showDomain; },
      selectDomain(d) { this.domain = d; this.showDomain = false; },
      closeDomainDropdown(e) {
        if (this.showDomain && this.$el.querySelector('.select-div') && !this.$el.querySelector('.select-div').contains(e.target)) {
          this.showDomain = false;
        }
      },
      addTagFromDiv(e) {
        const v = e.target.innerText.trim();
        if (v && !this.tags.includes(v)) this.tags.push(v);
        e.target.innerText = '';
      },
      removeTag(i) { this.tags.splice(i, 1); },
      addCoFromDiv(e) {
        const v = e.target.innerText.trim();
        if (v && !this.coSubmitters.includes(v)) this.coSubmitters.push(v);
        e.target.innerText = '';
      },
      removeCo(i) { this.coSubmitters.splice(i, 1); },
      submitForm() {
        if (!this.domain) { alert('Please select a domain'); return; }
        const formData = {
          postTitle: this.postTitle,
          postSummary: this.postSummary,
          content: this.content,
          fileName: this.fileName,
          domain: this.domain,
          tags: this.tags,
          hasPatent: this.hasPatent,
          hasDemo: this.hasDemo,
          coSubmitters: this.coSubmitters,
          scope: this.scope
        };
        console.log('Form Submitted:', formData);
        alert('Form submitted successfully!');
      },
      saveDraft() { this.saveContent(); alert('Draft saved!'); },
      cancel() {
        if (confirm('Are you sure? Unsaved changes will be lost.')) {
          localStorage.removeItem('richTextContent');
          location.reload();
        }
      }
    }
  };
</script>

<style scoped>
  /* --- THE MOST AGGRESSIVE FIX --- */
  .editor-content,
  .editor-content * {
    /* This creates a new directional context, isolating it from parents */
    unicode-bidi: embed !important;
    /* This forces the direction with maximum power */
    direction: ltr !important;
  }
  /* Ensure paragraph alignment defaults to left unless specified otherwise */
  .editor-content {
    text-align: left !important;
  }
  .editor-content * {
     /* Allows toolbar alignment buttons to override the default */
    text-align: inherit !important;
  }

  /* --- General & Layout --- */
  .full-page-container {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
    background-color: #f8fafc;
    min-height: 100vh;
  }
  .app-header {
    display: flex; justify-content: space-between; align-items: center;
    padding: 0 24px; height: 64px; background-color: #fff;
    border-bottom: 1px solid #e2e8f0; box-shadow: 0 1px 3px rgba(0,0,0,0.05);
  }
  .header-left, .header-right { display: flex; align-items: center; gap: 20px; }
  .logo-img { width: 32px; height: 32px; }
  .main-nav a { text-decoration: none; color: #4a5568; font-size: 14px; font-weight: 500; }
  .main-nav a:hover { color: #2b6cb0; }
  .search-btn, .publish-btn {
    height: 36px; padding: 0 16px; border-radius: 6px; border: 1px solid #cbd5e0;
    background-color: #fff; cursor: pointer; font-size: 14px; font-weight: 600;
  }
  .publish-btn { background-color: #d0021B; color: #fff; border-color: #d0021B; }
  .page-content-wrapper { display: flex; max-width: 1280px; margin: 0 auto; padding: 24px; gap: 24px; }
  .content-left-sidebar { width: 200px; flex-shrink: 0; }
  .content-main-area {
    width: 100%; background-color: #fff; padding: 32px; border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0,0,0,0.08);
  }
  .context-nav { font-size: 13px; color: #718096; margin-bottom: 16px; }

  /* --- Form Fields --- */
  .post-field-group { margin-bottom: 24px; }
  .post-field-group label {
    display: block; margin-bottom: 8px; font-weight: 600; font-size: 14px; color: #2d3748;
  }
  .input-wrapper { position: relative; }
  .input-wrapper input, .input-wrapper textarea {
    width: 100%; padding: 10px 12px; border: 1px solid #cbd5e0; border-radius: 6px;
    font-size: 14px; color: #2d3748; transition: all 0.2s;
  }
  .input-wrapper input:focus, .input-wrapper textarea:focus {
    border-color: #4299e1; outline: none; box-shadow: 0 0 0 1px #4299e1;
  }
  .input-wrapper textarea { min-height: 120px; resize: vertical; }
  .char-count { position: absolute; right: 10px; bottom: 8px; font-size: 12px; color: #a0aec0; }

  /* --- Rich Text Editor --- */
  .rich-text-editor {
    border: 1px solid #cbd5e0; border-radius: 8px; background-color: #fff;
    box-shadow: 0 2px 4px rgba(0,0,0,0.05); margin-bottom: 24px;
  }
  .toolbar {
    display: flex; flex-wrap: wrap; align-items: center; padding: 12px;
    background-color: #f7fafc; border-bottom: 1px solid #e2e8f0; gap: 6px;
    border-radius: 8px 8px 0 0;
  }
  .toolbar-group { display: flex; gap: 4px; }
  .toolbar-separator { width: 1px; height: 24px; background-color: #e2e8f0; margin: 0 8px; }
  .toolbar button {
    width: 34px; height: 34px; border: 1px solid transparent; background: none; cursor: pointer;
    border-radius: 6px; font-size: 16px; display: flex; align-items: center;
    justify-content: center; color: #4a5568; transition: all 0.2s;
  }
  .toolbar button:hover { background-color: #e2e8f0; }
  .toolbar button.active { background-color: #dbeafe; color: #2563eb; }
  .font-size-select, .color-picker {
    height: 34px; border: 1px solid #cbd5e0; border-radius: 6px; padding: 0 4px;
    background-color: white; cursor: pointer;
  }
  .color-picker { width: 34px; padding: 4px; }
  .editor-content {
    min-height: 400px; max-height: 600px; overflow-y: auto; padding: 20px; outline: none;
    font-size: 16px; line-height: 1.7; color: #1a202c; position: relative;
  }
  .editor-content.empty:before {
    content: attr(data-placeholder); color: #a0aec0;
    position: absolute; pointer-events: none; font-style: italic;
  }
  .editor-content :deep(img.resizable-image) {
    max-width: 100%; display: block; margin: 16px 0; border-radius: 6px;
    border: 1px solid #e2e8f0; resize: both; overflow: hidden;
  }
  .status-bar {
    display: flex; justify-content: space-between; padding: 12px 20px;
    background-color: #f7fafc; border-top: 1px solid #e2e8f0; font-size: 12px;
    color: #718096; border-radius: 0 0 8px 8px;
  }

  /* --- Other Form Fields --- */
  .form-grid { display: grid; grid-template-columns: 1fr; gap: 20px; margin-bottom: 32px; }
  .row { display: grid; grid-template-columns: 140px 1fr; gap: 16px; align-items: start; }
  .label { font-weight: 600; font-size: 14px; color: #2d3748; padding-top: 8px; }
  .control.inline { display: flex; align-items: center; gap: 12px; }
  .file-btn, .select-div, .toggle, .toggle-radio {
    padding: 10px 12px; border: 1px solid #cbd5e0; border-radius: 6px; cursor: pointer;
    background-color: #fff; transition: all 0.2s;
  }
  .select-div { position: relative; min-width: 200px; }
  .options {
    position: absolute; top: 100%; left: 0; width: 100%; margin-top: 4px;
    border: 1px solid #cbd5e0; background-color: #fff; z-index: 10;
    border-radius: 6px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); overflow: hidden;
  }
  .option { padding: 10px 12px; cursor: pointer; }
  .option:hover { background-color: #f7fafc; }
  .small { font-size: 12px; color: #718096; margin-top: 4px; }
  .error { font-size: 12px; color: #c53030; margin-top: 4px; }
  .tag-input {
    min-height: 40px; padding: 8px 12px; border: 1px solid #cbd5e0; border-radius: 6px;
    background-color: #fff; outline: none; cursor: text;
  }
  .tag-input:focus-within { border-color: #4299e1; box-shadow: 0 0 0 1px #4299e1; }
  .tags { margin-top: 8px; display: flex; flex-wrap: wrap; gap: 8px; }
  .tag {
    padding: 6px 12px; background-color: #edf2f7; border-radius: 16px;
    font-size: 13px; color: #4a5568; display: flex; align-items: center;
  }
  .tag .remove { margin-left: 8px; cursor: pointer; font-weight: bold; color: #a0aec0; }
  .tag .remove:hover { color: #c53030; }
  .toggle-radio.active, .toggle.active {
    background-color: #2b6cb0; color: #fff; border-color: #2b6cb0;
  }
  .req { color: #c53030; }

  /* --- Actions --- */
  .actions { display: flex; justify-content: flex-end; gap: 12px; margin-top: 24px; }
  .btn {
    display: inline-block; padding: 12px 24px; border-radius: 6px;
    border: 1px solid #cbd5e0; cursor: pointer; background-color: #fff;
    font-weight: 600; font-size: 14px; transition: all 0.2s;
  }
  .btn.primary { background-color: #2b6cb0; color: #fff; border-color: #2b6cb0; }
  .btn:hover { border-color: #a0aec0; }
  .btn.primary:hover { background-color: #2c5282; }
</style>
```

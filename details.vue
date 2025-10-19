<template>
    <div class="full-page-container">
      <!-- Loading Overlay -->
      <div v-show="isLoading" class="loading-overlay">
        <div class="loading-spinner"></div>
      </div>
  <div class="groundcolor" style="background-color:#fff !important">
      <!-- Header -->
      <div class="header">
        <div class="logo">
          <img
            src="/8e7e942f2952a6c9a3863"
            alt="expert community"
          />
        </div>
        <div class="navigation">
          <a href="#">首页</a>
          <a href="#" class="active">找专家</a>
        </div>
        <div class="user-actions">
          <a href="#" class="lang-switch">EN / CN</a>
        </div>
      </div>
  
      <!-- Page Content -->
      <div class="page-content-wrapper">
         <div class="side-actions">
      <button class="action-btn" @click="toggleLike" :disabled="isLiking">
        <img :src="isLiked ? '613f81804540de6061844ff62b7' : '1613f81804540de6061844ff62b7'" 
             alt="like" 
             class="action-icon"
             :class="{ liked: isLiked }" />
        <span>{{ post.likes }}</span>
      </button>
      <button class="action-btn" @click="scrollToComments">
        <img src="70e7aa613877f0cb8bcb7aac9036" 
             alt="comment" 
             class="action-icon" />
        <span>{{ totalComments }}</span>
      </button>
      <button class="action-btn" @click="sharePost">
        <img src="1c9d4fe9fb64be42607b2d3b611ca" 
             alt="share" 
             class="action-icon" />
      </button>
    </div>
        
        <div class="content-main-area">
          <div class="post-header">
            <a href="#" class="back-link" @click.prevent="goBack">← 返回</a>
            <h1 class="post-title">{{ post.title }}</h1>
            <div class="post-tags">
              <span class="tag" v-for="tag in post.tags" :key="tag">{{ tag }}</span>
            </div>
          </div>
          
          <div class="post-content" v-html="post.content"></div>
          
          <div class="author-info">
            <img :src="post.author.pic" alt="Author" class="author-avatar" />
            <div class="author-details">
              <span class="author-name">{{ post.author.name }}</span>
              <span class="author-id" v-show="post.author.id">({{ post.author.id }})</span>
            </div>
            <div class="post-time">{{ post.createTime }}</div>
          </div>
  
          <!-- TWO Rating Sections - Only show if postType is 1 -->
          <div class="rating-sections-wrapper" v-show="postType === 1">
            <div class="rating-section">
              <div class="rating-label">评价:</div>
              <div class="rating-stars">
                <span
                  v-for="star in 5"
                  :key="star"
                  class="star"
                  :class="{ filled: star <= currentRating }"
                  @click="ratePost(star)"
                >★</span>
              </div>
              <div class="rating-count">({{ currentRating }})</div>
            </div>
            
            <div class="rating-section">
              <div class="rating-label">质量:</div>
              <div class="rating-stars">
                <span
                  v-for="star in 5"
                  :key="'q-' + star"
                  class="star"
                  :class="{ filled: star <= qualityRating }"
                  @click="rateQuality(star)"
                >★</span>
              </div>
              <div class="rating-count">({{ qualityRating }})</div>
            </div>
          </div>
  
          <div class="comments-section" ref="commentsSection">
            <h3 class="comments-title">评论 ({{ totalComments }})</h3>
            
            <!-- SIMPLE FLAT DISPLAY OF ALL COMMENTS -->
            <div class="comment-list">
              <div v-for="comment in paginatedComments" :key="comment.commentId">
                <div class="comment-item">
                  <img :src="comment.userPhoto" alt="user" class="comment-avatar" />
                  <div class="comment-content">
                    <div class="comment-header">
                      <span class="comment-author">{{ getCleanName(comment.userFullName) }}</span>
                      <span class="comment-id" v-show="comment.userNumber">({{ comment.userNumber }})</span>
                      <span v-show="comment.commentParentId !== '0'" class="reply-arrow">回复</span>
                      <span v-show="comment.commentParentId !== '0' && comment.parentAuthorName" class="comment-author">{{ getCleanName(comment.parentAuthorName) }}</span>
                      <span v-show="comment.commentParentId !== '0' && comment.parentAuthorId" class="comment-id">({{ comment.parentAuthorId }})</span>
                      <span class="comment-time">{{ comment.createdDt }}</span>
                    </div>
                    <div class="comment-body" v-html="comment.commentDetails"></div>
                    <div class="comment-actions">
                      <a href="#" @click.prevent="openReplyModal(comment)" class="reply-btn">回复</a>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          
          <div class="pagination-container" v-show="totalPages > 1">
            <span class="page-arrow" @click="previousPage" :class="{ disabled: currentPage===1 }">‹</span>
            <span
              v-for="page in displayedPages"
              :key="page"
              class="page-number"
              :class="{ active: currentPage===page, ellipsis: page==='...' }"
              @click="goToPage(page)"
            >{{ page }}</span>
            <span class="page-arrow" @click="nextPage" :class="{ disabled: currentPage===totalPages }">›</span>
          </div>
  
          <div class="new-comment-section">
            <h4 class="new-comment-title">发表评论</h4>
            <div ref="newCommentEditor" class="new-comment-editor"></div>
            <div class="new-comment-actions">
              <div class="custom-checkbox-container" @click="isAnonymousComment=!isAnonymousComment">
                <div class="custom-checkbox" :class="{ checked:isAnonymousComment }">
                  <span v-show="isAnonymousComment">✓</span>
                </div>
                <label>匿名评论</label>
              </div>
              <button @click="submitNewComment" :disabled="isSubmittingComment" class="submit-btn">
                {{ isSubmittingComment?'提交中...':'发表评论' }}
              </button>
            </div>
          </div>
        </div>
      </div>
       </div>
      <!-- Reply Modal - FIXED POSITION -->
      <div v-show="isReplyModalOpen" class="modal-overlay" @click.self="closeReplyModal">
        <div class="modal-content">
          <div class="modal-header">
            <h3 class="modal-title">
              回复 {{ replyingToComment ? getCleanName(replyingToComment.userFullName) : '' }} 
              <span v-show="replyingToComment && replyingToComment.userNumber">({{ replyingToComment ? replyingToComment.userNumber : '' }})</span>
            </h3>
            <button @click="closeReplyModal" class="modal-close-btn">✕</button>
          </div>
          <div class="modal-body">
            <div ref="replyEditor" class="reply-editor"></div>
          </div>
          <div class="modal-footer">
            <button @click="closeReplyModal" class="modal-btn btn-cancel">取消</button>
            <button @click="submitReply" :disabled="isSubmittingReply" class="modal-btn btn-submit">
              {{ isSubmittingReply?'提交中...':'提交回复' }}
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
        postId: null,
        postType: null,
        userLikeId: null,
        post: {
          title: '',
          tags: [],
          content: '',
          author: { name: '', id: '', pic: '' },
          likes: 0,
          views: 0,
          createTime: ''
        },
        comments: [],
        isReplyModalOpen: false,
        replyingToComment: null,
        replyQuill: null,
        replyContent: '',
        newCommentQuill: null,
        newCommentContent: '',
        isAnonymousComment: false,
        currentPage: 1,
        commentsPerPage: 7,
        totalPages: 1,
        isLiked: false,
        isLiking: false,
        isLoading: false,
        isSubmittingComment: false,
        isSubmittingReply: false,
        currentRating: 0,
        qualityRating: 0,
        currentUserInfo: { userId: '', userName: '', userDept: '', empNo: '' }
      }
    },
    computed: {
      totalComments() {
        return this.comments.length
      },
      allComments() {
        const result = []
        const topLevel = this.comments.filter(c => c.commentParentId === '0' || !c.commentParentId)
        
        const addWithChildren = (comment, level) => {
          comment.level = level
          result.push(comment)
          
          const children = this.comments.filter(c => c.commentParentId === comment.commentId.toString())
          children.forEach(child => {
            child.parentAuthorName = comment.userFullName
            child.parentAuthorId = comment.userNumber
            addWithChildren(child, level + 1)
          })
        }
        
        topLevel.forEach(comment => addWithChildren(comment, 0))
        return result
      },
      paginatedComments() {
        const start = (this.currentPage - 1) * this.commentsPerPage
        const end = start + this.commentsPerPage
        return this.allComments.slice(start, end)
      },
      displayedPages() {
        const pages = []
        for (let i = 1; i <= this.totalPages; i++) {
          pages.push(i)
        }
        return pages
      }
    },
    watch: {
      allComments() {
        this.totalPages = Math.ceil(this.allComments.length / this.commentsPerPage)
      }
    },
    async mounted() {
      this.postId = this.getPostIdFromUrl()
      const postType = $vm.$route.query.postType
      const hostname = window.location.hostname
      
      if (!hostname.includes('star.com')) {
        this.validateType(postType)
      } else {
        if (postType) {
          this.postType = Number(postType)
        }
      }
      
      if (this.postId) {
        await this.fetchCurrentUserInfo()
        await this.fetchPostData()
        await this.checkUserLike()
        await this.incrementViews()
        await this.loadComments()
      } else {
        alert('未找到帖子ID')
      }
      this.loadQuill()
    },
    beforeDestroy() {
      if (this.newCommentQuill) {
        this.newCommentQuill = null
      }
      if (this.replyQuill) {
        this.replyQuill = null
      }
    },
    methods: {
      validateType(postType) {
        const numericPostType = Number(postType)
        const validTypes = [1, 2, 3]
  
        if (validTypes.includes(numericPostType)) {
          this.postType = numericPostType
          console.log(`Post type set to: ${this.postType}`)
        } else {
          alert('Invalid post type! Redirecting...')
          window.location.href = '/error-page'
        }
      },
  
      getCleanName(fullName) {
        if (!fullName) return ''
        const lastSpaceIndex = fullName.lastIndexOf(' ')
        if (lastSpaceIndex !== -1) {
          const afterSpace = fullName.substring(lastSpaceIndex + 1).trim()
          if (/^\d+$/.test(afterSpace)) {
            return fullName.substring(0, lastSpaceIndex).trim()
          }
        }
        return fullName
      },
  
      getIndentLevel(comment) {
        return '0vw'
      },
      
      async fetchCurrentUserInfo() {
        try {
          const userInfo = $vm.$service.base.getUserInfoSync()
          console.log('User Info from sync:', userInfo)
  
          const userDataResponse = await fetch(
            '//personalservletdata',
            {
              method: 'POST',
              credentials: 'include',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({ empNo: userInfo.employeeNumber })
            }
          )
  
          if (!userDataResponse.ok) {
            throw new Error('Failed to fetch user data')
          }
  
          const userData = await userDataResponse.json()
          console.log('User Data Response:', userData)
  
          this.currentUserInfo = {
            userId: userInfo.userId || '',
            userName: userInfo.displayName || '',
            userDept: userData.name || '',
            empNo: userInfo.employeeNumber || ''
          }
  
          console.log('Current User Info Set:', this.currentUserInfo)
        } catch (error) {
          console.error('Error fetching current user info:', error)
          this.currentUserInfo = { userId: '', userName: '', userDept: '', empNo: '' }
        }
      },
  
      getPostIdFromUrl() {
        const hash = window.location.hash
        const queryIndex = hash.indexOf('?')
        return queryIndex === -1 ? null : new URLSearchParams(hash.substring(queryIndex)).get('postId')
      },
  
      async incrementViews() {
        try {
          const response = await fetch(
            '///posts/update',
            {
              method: 'POST',
              credentials: 'include',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({ 
                postId: this.postId, 
                views: this.post.views + 2
              })
            }
          )
          if (response.ok) {
            this.post.views += 2
          }
        } catch (error) {
          console.error('Error incrementing views:', error)
        }
      },
  
      async fetchPostData() {
        console.log('Fetching post data for postId:', this.postId)
        this.isLoading = true
        try {
          const response = await fetch(
            '//posts/findOne',
            {
              method: 'POST',
              credentials: 'include',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({ postId: this.postId })
            }
          )
  
          if (!response.ok) {
            throw new Error('HTTP error! status: ' + response.status)
          }
  
          const data = await response.json()
          console.log('Received post data:', data)
  
          if (!data || !data.postTitle) {
            throw new Error('Invalid post data received')
          }
  
          let authorName = ''
          let authorNumber = ''
          let authorPhoto = 'https://via.placeholder.com/40'
          
          if (data.creatorUserCn && typeof data.creatorUserCn === 'string') {
            const rawString = data.creatorUserCn.trim()
            const lastSpaceIndex = rawString.lastIndexOf(' ')
            
            if (lastSpaceIndex !== -1) {
              authorName = rawString.substring(0, lastSpaceIndex).trim()
              authorNumber = rawString.substring(lastSpaceIndex + 1).trim()
            } else {
              authorName = rawString
            }
          }
  
          if (data.createdBy) {
            authorPhoto = 'xxce/' + data.createdBy + '/120'
          }
  
          this.post = {
            title: data.postTitle,
            tags: JSON.parse(data.postTags || '[]'),
            content: data.postDetails,
            author: {
              name: authorName || '匿名用户',
              id: authorNumber,
              pic: authorPhoto
            },
            likes: data.likes || 0,
            views: data.views || 0,
            createTime: data.createdDt || ''
          }
  
          console.log('Post loaded successfully:', this.post.title)
        } catch (error) {
          console.error('Error fetching post data:', error)
          alert('加载帖子失败: ' + error.message)
        } finally {
          this.isLoading = false
        }
      },
  
      async checkUserLike() {
        try {
          const creatorUserCn = this.currentUserInfo.userName + ' ' + this.currentUserInfo.empNo
          
          console.log('Checking user like for:', creatorUserCn, 'postId:', this.postId)
          
          const response = await fetch(
            '/postLikes/findOne',
            {
              method: 'POST',
              credentials: 'include',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({
                creatorUserCn: creatorUserCn,
                postId: this.postId
              })
            }
          )
  
          if (response.ok) {
            const data = await response.json()
            console.log('Like check response:', data)
            
            // Save the like ID and set liked status
            if (data && (data.postLikesId || data.likeId || data.id)) {
              this.userLikeId = data.postLikesId || data.likeId || data.id
              this.isLiked = true
              console.log('User already liked - ID:', this.userLikeId)
            } else {
              this.userLikeId = null
              this.isLiked = false
              console.log('User has not liked')
            }
          } else {
            this.userLikeId = null
            this.isLiked = false
            console.log('User has not liked this post')
          }
        } catch (error) {
          console.error('Error checking user like:', error)
          this.userLikeId = null
          this.isLiked = false
        }
      },
  
      async toggleLike() {
        if (this.isLiking) return
  
        this.isLiking = true
        const previousLikes = this.post.likes
        const previousLiked = this.isLiked
        const previousLikeId = this.userLikeId
  
        // Toggle UI optimistically
        this.isLiked = !this.isLiked
        this.post.likes += this.isLiked ? 1 : -1
  
        try {
          const creatorUserCn = this.currentUserInfo.userName + ' ' + this.currentUserInfo.empNo
          
          console.log('Toggle like - isLiked:', this.isLiked, 'likeId:', this.userLikeId)
  
          if (this.isLiked) {
            // ADD LIKE
            console.log('Adding like...')
            const insertResponse = await fetch(
              '//insert',
              {
                method: 'POST',
                credentials: 'include',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                  creatorUserCn: creatorUserCn,
                  postId: this.postId
                })
              }
            )
  
            if (!insertResponse.ok) {
              const errorText = await insertResponse.text()
              console.error('Insert failed:', errorText)
              throw new Error('Failed to add like')
            }
  
            const insertData = await insertResponse.json()
            console.log('Like inserted successfully:', insertData)
            
            // Save the new like ID
            this.userLikeId = insertData.postLikesId || insertData.likeId || insertData.id
            console.log('New like ID saved:', this.userLikeId)
  
          } else {
            // REMOVE LIKE - Use postLikesId
            console.log('Removing like with ID:', this.userLikeId)
            
            if (!this.userLikeId) {
              throw new Error('No like ID found')
            }
            
            const deleteResponse = await fetch(
              'postLikes/delete',
              {
                method: 'POST',
                credentials: 'include',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                  postLikesId: this.userLikeId
                })
              }
            )
  
            if (!deleteResponse.ok) {
              const errorText = await deleteResponse.text()
              console.error('Delete failed:', errorText)
              throw new Error('Failed to remove like')
            }
  
            const deleteData = await deleteResponse.json()
            console.log('Like deleted successfully:', deleteData)
            
            // Clear the like ID
            this.userLikeId = null
          }
  
          // Update the post likes count
          console.log('Updating post likes count to:', this.post.likes)
          const updateResponse = await fetch(
            'posts/update',
            {
              method: 'POST',
              credentials: 'include',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({ 
                postId: this.postId, 
                likes: this.post.likes 
              })
            }
          )
  
          if (!updateResponse.ok) {
            throw new Error('Failed to update likes count')
          }
  
          console.log('Likes count updated successfully')
  
        } catch (error) {
          console.error('Error toggling like:', error)
          // Revert UI on error
          this.isLiked = previousLiked
          this.post.likes = previousLikes
          this.userLikeId = previousLikeId
          alert('操作失败，请重试: ' + error.message)
        } finally {
          this.isLiking = false
        }
      },
  
      async loadComments() {
        try {
          this.isLoading = true
          const response = await fetch(
            '/postComments/findList',
            {
              method: 'POST',
              credentials: 'include',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({ postId: this.postId })
            }
          )
  
          if (!response.ok) {
            throw new Error('Failed to load comments')
          }
  
          const data = await response.json()
          console.log('Comments loaded RAW:', data)
  
          const parseUserInfo = (comment) => {
            let userFullName = ''
            let userNumber = ''
            let userPhoto = 'https://via.placeholder.com/40'
  
            if (comment.creatorUserCn && typeof comment.creatorUserCn === 'string') {
              const rawString = comment.creatorUserCn.trim()
              const lastSpaceIndex = rawString.lastIndexOf(' ')
              
              if (lastSpaceIndex !== -1) {
                userFullName = rawString.substring(0, lastSpaceIndex).trim()
                userNumber = rawString.substring(lastSpaceIndex + 1).trim()
              } else {
                userFullName = rawString
              }
            }
  
            if (comment.createdBy) {
              userPhoto = '/' + comment.createdBy + '/120'
            }
  
            return { userFullName, userNumber, userPhoto }
          }
  
          if (Array.isArray(data)) {
            this.comments = data.map(comment => {
              const { userFullName, userNumber, userPhoto } = parseUserInfo(comment)
              
              return {
                ...comment,
                commentParentId: comment.commentParentId != null ? comment.commentParentId.toString() : '0',
                userFullName: userFullName,
                userNumber: userNumber,
                userPhoto: userPhoto
              }
            })
          } else if (data && Array.isArray(data.data)) {
            this.comments = data.data.map(comment => {
              const { userFullName, userNumber, userPhoto } = parseUserInfo(comment)
              
              return {
                ...comment,
                commentParentId: comment.commentParentId != null ? comment.commentParentId.toString() : '0',
                userFullName: userFullName,
                userNumber: userNumber,
                userPhoto: userPhoto
              }
            })
          } else {
            this.comments = []
          }
  
          console.log('Comments processed:', this.comments)
          console.log('Total comments:', this.comments.length)
        } catch (error) {
          console.error('Error loading comments:', error)
          this.comments = []
        } finally {
          this.isLoading = false
        }
      },
  
      ratePost(rating) {
        this.currentRating = rating
        console.log('Post rated:', rating)
      },
  
      rateQuality(rating) {
        this.qualityRating = rating
        console.log('Quality rated:', rating)
      },
  
      sharePost() {
        const url = window.location.href
        if (navigator.share) {
          navigator.share({ title: this.post.title, url: url })
            .catch(error => console.log('Error sharing:', error))
        } else {
          navigator.clipboard.writeText(url).then(
            () => alert('链接已复制到剪贴板!'),
            err => console.error('Could not copy text: ', err)
          )
        }
      },
  
      scrollToComments() {
        this.$refs.commentsSection.scrollIntoView({ behavior: 'smooth' })
      },
  
      goBack() {
        window.history.back()
      },
  
      goToPage(page) {
        if (page !== '...') this.currentPage = page
      },
  
      previousPage() {
        if (this.currentPage > 1) this.currentPage--
      },
  
      nextPage() {
        if (this.currentPage < this.totalPages) this.currentPage++
      },
  
      async submitReply() {
        if (!this.replyContent.trim() || this.replyContent.trim() === '<p><br></p>') {
          alert('回复内容不能为空')
          return
        }
  
        this.isSubmittingReply = true
  
        try {
          const response = await fetch(
            '/postComments/insert',
            {
              method: 'POST',
              credentials: 'include',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({
                commentParentId: this.replyingToComment.commentId.toString(),
                postId: this.postId,
                creatorDept: this.currentUserInfo.userDept,
                commentDetails: this.replyContent,
                creatorUserCn: this.currentUserInfo.userName + ' ' + this.currentUserInfo.empNo,
                isAnonymous: '0'
              })
            }
          )
  
          if (!response.ok) {
            throw new Error('Failed to submit reply')
          }
  
          const data = await response.json()
          console.log('Reply submitted successfully:', data)
  
          this.closeReplyModal()
          await this.loadComments()
          
          alert('回复成功!')
        } catch (error) {
          console.error('Error submitting reply:', error)
          alert('回复失败，请重试: ' + error.message)
        } finally {
          this.isSubmittingReply = false
        }
      },
  
      async submitNewComment() {
        if (!this.newCommentContent.trim() || this.newCommentContent.trim() === '<p><br></p>') {
          alert('评论内容不能为空')
          return
        }
  
        this.isSubmittingComment = true
  
        try {
          const userNameToSubmit = this.isAnonymousComment 
            ? '匿名用户' 
            : this.currentUserInfo.userName + ' ' + this.currentUserInfo.empNo
  
          const response = await fetch(
            '/postComments/insert',
            {
              method: 'POST',
              credentials: 'include',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({
                commentParentId: '0',
                postId: this.postId,
                creatorDept: this.currentUserInfo.userDept,
                commentDetails: this.newCommentContent,
                creatorUserCn: userNameToSubmit,
                isAnonymous: this.isAnonymousComment ? '1' : '0'
              })
            }
          )
  
          if (!response.ok) {
            throw new Error('Failed to submit comment')
          }
  
          const data = await response.json()
          console.log('Comment submitted successfully:', data)
  
          this.newCommentQuill.setContents([])
          this.newCommentContent = ''
          
          await this.loadComments()
          
          alert('评论成功!')
        } catch (error) {
          console.error('Error submitting comment:', error)
          alert('评论失败，请重试: ' + error.message)
        } finally {
          this.isSubmittingComment = false
        }
      },
  
      openReplyModal(comment) {
        this.replyingToComment = comment
        this.isReplyModalOpen = true
        this.$nextTick(() => {
          this.initializeReplyEditor()
        })
      },
  
      closeReplyModal() {
        this.isReplyModalOpen = false
        this.replyingToComment = null
        this.replyContent = ''
        if (this.replyQuill) {
          this.replyQuill = null
        }
      },
  
      loadQuill() {
        if (window.Quill) {
          this.initializeNewCommentEditor()
          return
        }
  
        const quillScript = document.createElement('script')
        quillScript.src = 'https://cdn.jsdelivr.net/npm/quill@2.0.3/dist/quill.min.js'
        quillScript.async = true
        quillScript.onload = () => {
          console.log('Quill loaded')
          this.initializeNewCommentEditor()
        }
        document.head.appendChild(quillScript)
  
        const quillCss = document.createElement('link')
        quillCss.rel = 'stylesheet'
        quillCss.href = 'https://cdn.jsdelivr.net/npm/quill@2.0.3/dist/quill.snow.min.css'
        document.head.appendChild(quillCss)
      },
  
      getToken() {
        return new Promise((resolve, reject) => {
          fetch(
            '//getedmauthorization',
            {
              method: 'POST',
              credentials: 'include',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify({})
            }
          )
            .then(res => res.json())
            .then(data => resolve(data.authorization))
            .catch(err => reject(err))
        })
      },
  
      uploadEdmApi(file) {
        return new Promise((resolve, reject) => {
          this.getToken().then(token => {
            const formData = new FormData()
            formData.append('serverType', 'upload')
            formData.append('origin', '')
            formData.append('updateFile', false)
            formData.append('multipartFile', file)
  
            fetch(
              '/documents',
              {
                method: 'PUT',
                credentials: 'include',
                headers: {
                 
                },
                body: formData
              }
            )
              .then(res => res.json())
              .then(data => resolve(data.result))
              .catch(err => reject(err))
          })
        })
      },
  
      initializeNewCommentEditor() {
        if (!window.Quill || this.newCommentQuill) return
  
        this.$nextTick(() => {
          const Link = Quill.import('formats/link')
          Link.sanitize = function(url) {
            let protocol = url.slice(0, url.indexOf(':'))
            if (this.PROTOCOL_WHITELIST.indexOf(protocol) === -1) {
              url = 'https://' + url
            }
            const anchor = document.createElement('a')
            anchor.href = url
            protocol = anchor.href.slice(0, anchor.href.indexOf(':'))
            return this.PROTOCOL_WHITELIST.indexOf(protocol) > -1 ? url : this.PROTOCOL_WHITELIST[0] + url
          }
  
          const Font = Quill.import('formats/font')
          Font.whitelist = ['sans-serif', 'serif', 'monospace', 'microsoft-yahei', 'arial', 'times-new-roman']
          Quill.register(Font, true)
  
          const toolbarOptions = [
            ['bold', 'italic', 'underline', 'strike'],
            [{ header: [1, 2, 3, 4, 5, 6, false] }],
            [{ list: 'ordered' }, { list: 'bullet' }],
            [{ script: 'sub' }, { script: 'super' }],
            [{ indent: '-1' }, { indent: '+1' }],
            [{ direction: 'rtl' }],
            [{ size: ['small', false, 'large', 'huge'] }],
            [{ color: [] }, { background: [] }],
            [{ font: Font.whitelist }],
            [{ align: [] }],
            ['clean'],
            ['link', 'image']
          ]
  
          this.newCommentQuill = new Quill(this.$refs.newCommentEditor, {
            modules: {
              toolbar: {
                container: toolbarOptions,
                handlers: {
                  image: () => this.imageHandler('newComment')
                }
              }
            },
            theme: 'snow',
            placeholder: '留下你的精彩评论...'
          })
  
          this.newCommentQuill.on('text-change', () => {
            this.newCommentContent = this.newCommentQuill.root.innerHTML
          })
        })
      },
  
      initializeReplyEditor() {
        if (!window.Quill) {
          console.error('Quill not loaded yet!')
          return
        }
        
        if (this.replyQuill) {
          const container = this.$refs.replyEditor
          if (container) {
            container.innerHTML = ''
          }
          this.replyQuill = null
        }
  
        this.$nextTick(() => {
          const Link = Quill.import('formats/link')
          Link.sanitize = function(url) {
            let protocol = url.slice(0, url.indexOf(':'))
            if (this.PROTOCOL_WHITELIST.indexOf(protocol) === -1) {
              url = 'https://' + url
            }
            const anchor = document.createElement('a')
            anchor.href = url
            protocol = anchor.href.slice(0, anchor.href.indexOf(':'))
            return this.PROTOCOL_WHITELIST.indexOf(protocol) > -1 ? url : this.PROTOCOL_WHITELIST[0] + url
          }
  
          const Font = Quill.import('formats/font')
          Font.whitelist = ['sans-serif', 'serif', 'monospace', 'microsoft-yahei', 'arial', 'times-new-roman']
          Quill.register(Font, true)
  
          const toolbarOptions = [
            ['bold', 'italic', 'underline', 'strike'],
            [{ header: [1, 2, 3, 4, 5, 6, false] }],
            [{ list: 'ordered' }, { list: 'bullet' }],
            [{ script: 'sub' }, { script: 'super' }],
            [{ indent: '-1' }, { indent: '+1' }],
            [{ direction: 'rtl' }],
            [{ size: ['small', false, 'large', 'huge'] }],
            [{ color: [] }, { background: [] }],
            [{ font: Font.whitelist }],
            [{ align: [] }],
            ['clean'],
            ['link', 'image']
          ]
  
          this.replyQuill = new Quill(this.$refs.replyEditor, {
            modules: {
              toolbar: {
                container: toolbarOptions,
                handlers: {
                  image: () => this.imageHandler('reply')
                }
              }
            },
            theme: 'snow',
            placeholder: '写下你的回复...'
          })
  
          this.replyQuill.on('text-change', () => {
            this.replyContent = this.replyQuill.root.innerHTML
          })
        })
      },
  
      imageHandler(editorType) {
        const input = document.createElement('input')
        input.setAttribute('type', 'file')
        input.setAttribute('accept', 'image/*')
        input.click()
  
        input.onchange = () => {
          const file = input.files[0]
          if (file) {
            this.uploadEdmApi(file).then(result => {
              const imageUrl = '/open/preview/' + result.docId + '/source'
              const quill = editorType === 'reply' ? this.replyQuill : this.newCommentQuill
              const range = quill.getSelection(true)
              quill.insertEmbed(range.index, 'image', imageUrl)
              quill.setSelection(range.index + 1)
            }).catch(error => {
              console.error('Image upload failed:', error)
              alert('无法上传图片，请重试')
            })
          }
        }
      }
    }
  }
  </script>
  
  
  
  
  
  
  
  
  <style scoped>
  
  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }
  
  .action-icon {
    width: 1.4vw;
    height: 1.4vw;
    margin-bottom: 0.2vw;
    transition: all 0.3s;
  }
  
  .action-icon.liked {
    filter: brightness(1.3);
    transform: scale(1.1);
  }
  
  /* Remove these old emoji styles */
  .icon-like::before,
  .icon-comment::before,
  .icon-share::before {
    display: none;
  }
  
  .comment-body img,
  .post-content img {
    max-width: 100% !important;
    height: auto !important;
    display: block;
    margin: 0.8vw 0;
    border-radius: 0.4vw;
  }
  
  .loading-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background-color: rgba(0, 0, 0, 0.75);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 99999;
  }
  
  .loading-spinner {
    width: 4vw;
    height: 4vw;
    border: 0.4vw solid #f3f3f3;
    border-top: 0.4vw solid #d0021b;
    border-radius: 50%;
    animation: spin 1s linear infinite;
  }
  
  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }
  
  .full-page-container {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, 'Microsoft YaHei', sans-serif;
    background-color: #ffffff;
    min-height: 100vh;
  }
  
  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1vw 3vw;
    background-color: #fff;
    border-bottom: 0.05vw solid #e0e0e0;
    top: 0;
    left: 0;
    right: 0;
    z-index: 1000;
    height: 4vw;
    box-shadow: 0 0.1vw 0.2vw rgba(0, 0, 0, 0.05);
  }
  
  .logo img {
    height: 2.5vw;
    width: auto;
  }
  
  .navigation {
    display: flex;
    gap: 2vw;
  }
  
  .navigation a {
    text-decoration: none;
    color: #333;
    font-size: 0.95vw;
    font-weight: 500;
    padding-bottom: 0.3vw;
    transition: all 0.2s;
  }
  
  .navigation a:hover {
    color: #d77b6b;
  }
  
  .navigation a.active {
    color: #d77b6b;
    border-bottom: 0.15vw solid #d77b6b;
  }
  
  .lang-switch {
    text-decoration: none;
    color: #666;
    font-size: 0.85vw;
    transition: color 0.2s;
  }
  
  .lang-switch:hover {
    color: #333;
  }
  
  /* FIXED WIDTH CENTERED LAYOUT */
  .page-content-wrapper {
    display: flex;
    justify-content: center;
    max-width: 1400px;
    width: 100%;
    margin: 2vw auto;
    gap: 7vw;
    padding: 0 2vw;
   // margin-left: 6vw;
  }
  
  .side-actions {
    display: flex;
    flex-direction: column;
    gap: 1.2vw;
    width: 60px;
    flex-shrink: 0;
    position: sticky;
    top: 12vw;
    height: fit-content;
  }
  
  .side-actions .action-btn {
    background-color: #fff;
    border: 0.08vw solid #ddd;
    border-radius: 2.6vw;
    width: 3vw;
    height: 3vw;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    font-size: 0.7vw;
    color: #606770;
    transition: all 0.3s;
    box-shadow: 0 0.1vw 0.3vw rgba(0, 0, 0, 0.08);
  }
  
  .side-actions .action-btn:hover:not(:disabled) {
    background-color: #f0f2f5;
    border-color: #999;
    transform: translateY(-0.2vw);
    box-shadow: 0 0.3vw 0.6vw rgba(0, 0, 0, 0.15);
  }
  
  .side-actions .action-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
  
  /* FIXED WIDTH MAIN CONTENT */
  .content-main-area {
    flex: 1;
    max-width: 900px;
    width: 100%;
    background-color: #fff;
    padding: 0.5vw;
    overflow-wrap: break-word;
    word-wrap: break-word;
    word-break: break-word;
  }
  
  .back-link {
    color: #65676b;
    text-decoration: none;
    font-size: 0.85vw;
    margin-bottom: 1vw;
    display: inline-block;
    transition: color 0.2s;
  }
  
  .back-link:hover {
    color: #1877f2;
  }
  
  /* TEXT CONTROL - MAX 100 CHARS PER LINE */
  .post-title {
    font-size: 1.8vw;
    font-weight: 700;
    color: #1c1e21;
    margin: 0.8vw 0;
    line-height: 1.3;
    max-width: 100%;
    overflow-wrap: break-word;
    word-wrap: break-word;
    word-break: break-word;
    hyphens: auto;
  }
  
  .post-tags {
    display: flex;
    gap: 0.6vw;
    flex-wrap: wrap;
    margin: 1vw 0;
  }
  
  .tag {
    background-color: #e7f3ff;
    color: #1877f2;
    padding: 0.4vw 1vw;
    border-radius: 1.2vw;
    font-size: 0.75vw;
    font-weight: 500;
  }
  
  .post-content {
    font-size: 0.95vw;
    line-height: 1.8;
    color: #1c1e21;
    margin: 1.5vw 0;
    max-width: 100%;
    overflow-wrap: break-word;
    word-wrap: break-word;
    word-break: break-word;
    hyphens: auto;
  }
  
  /* Long words/URLs will break */
  .post-content p,
  .comment-body p {
    max-width: 100%;
    overflow-wrap: break-word;
    word-wrap: break-word;
    word-break: break-word;
  }
  
  .author-info {
    display: flex;
    align-items: center;
    gap: 1vw;
    padding: 1.5vw 0;
    border-top: 0.05vw solid #e4e6eb;
    border-bottom: 0.05vw solid #e4e6eb;
    margin: 1.5vw 0;
  }
  
  .author-avatar {
    width: 2.8vw;
    height: 2.8vw;
    border-radius: 50%;
  }
  
  .author-details {
    display: flex;
    gap: 0.4vw;
    align-items: center;
    overflow-wrap: break-word;
  }
  
  .author-name {
    font-weight: 600;
    font-size: 0.9vw;
    color: #1c1e21;
    overflow-wrap: break-word;
  }
  
  .author-id {
    font-size: 0.75vw;
    color: #65676b;
  }
  
  .post-time {
    margin-left: auto;
    font-size: 0.75vw;
    color: #65676b;
  }
  
  .rating-sections-wrapper {
    display: flex;
    justify-content: space-between;
    gap: 2vw;
    padding: 1.2vw 0;
    margin: 1vw 0;
    border-bottom: 0.05vw solid #e4e6eb;
  }
  
  .rating-section {
    display: flex;
    align-items: center;
    gap: 1vw;
  }
  
  .rating-label {
    font-size: 0.95vw;
    font-weight: 600;
    color: #1c1e21;
  }
  
  .rating-stars {
    display: flex;
    gap: 0.3vw;
  }
  
  .rating-stars .star {
    font-size: 1.8vw;
    color: #ddd;
    cursor: pointer;
    transition: all 0.2s;
    user-select: none;
  }
  
  .rating-stars .star:hover,
  .rating-stars .star.filled {
    color: #ffa500;
    transform: scale(1.15);
  }
  
  .rating-count {
    font-size: 0.85vw;
    color: #65676b;
    margin-left: 0.3vw;
  }
  
  .comments-section {
    margin-top: 2vw;
  }
  
  .comments-title {
    font-size: 1.2vw;
    font-weight: 600;
    color: #1c1e21;
    margin-bottom: 1.5vw;
  }
  
  .comment-list {
    display: flex;
    flex-direction: column;
    gap: 0.3vw;
  }
  
  .comment-item {
    display: flex;
    gap: 1vw;
    padding: 1.2vw;
    background-color: #fff;
    border-bottom: 0.05vw solid #e4e6eb;
    transition: all 0.2s;
  }
  
  .comment-item:hover {
    background-color: #f9f9f9;
  }
  
  .comment-avatar {
    width: 2.5vw;
    height: 2.5vw;
    border-radius: 50%;
    flex-shrink: 0;
  }
  
  .comment-content {
    flex: 1;
    overflow: hidden;
    max-width: 100%;
  }
  
  .comment-header {
    display: flex;
    align-items: center;
    gap: 0.4vw;
    margin-bottom: 0.6vw;
    flex-wrap: wrap;
  }
  
  .comment-author {
    font-weight: 600;
    font-size: 0.85vw;
    color: #1c1e21;
  }
  
  .comment-id {
    font-size: 0.75vw;
    color: #65676b;
  }
  
  .reply-arrow {
    font-size: 0.75vw;
    color: #999;
    margin: 0 0.2vw;
  }
  
  .comment-time {
    font-size: 0.7vw;
    color: #999;
    margin-left: auto;
  }
  
  .comment-body {
    font-size: 0.9vw;
    line-height: 1.6;
    color: #1c1e21;
    margin-bottom: 0.6vw;
    max-width: 100%;
    overflow-wrap: break-word;
    word-wrap: break-word;
    word-break: break-word;
  }
  
  .comment-actions {
    display: flex;
    gap: 1vw;
  }
  
  .reply-btn {
    color: #1877f2;
    text-decoration: none;
    font-size: 0.75vw;
    font-weight: 600;
    transition: color 0.2s;
  }
  
  .reply-btn:hover {
    color: #166fe5;
    text-decoration: underline;
  }
  
  .pagination-container {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 0.5vw;
    margin: 2vw 0;
  }
  
  .page-arrow,
  .page-number {
    padding: 0.5vw 0.9vw;
    border: 0.05vw solid #ddd;
    border-radius: 0.4vw;
    cursor: pointer;
    font-size: 0.85vw;
    background-color: #fff;
    transition: all 0.2s;
  }
  
  .page-arrow:hover:not(.disabled),
  .page-number:hover:not(.ellipsis) {
    background-color: #f0f2f5;
    border-color: #999;
  }
  
  .page-number.active {
    background-color: #1877f2;
    color: #fff;
    border-color: #1877f2;
  }
  
  .page-arrow.disabled,
  .page-number.ellipsis {
    opacity: 0.5;
    cursor: not-allowed;
    border: none;
  }
  
  .new-comment-section {
    margin-top: 2vw;
    padding-top: 2vw;
    border-top: 0.05vw solid #e4e6eb;
  }
  
  .new-comment-title {
    font-size: 1vw;
    font-weight: 600;
    color: #1c1e21;
    margin-bottom: 1vw;
  }
  
  .new-comment-editor {
    border: 0.08vw solid #ccd0d5;
    border-radius: 0.5vw;
    background-color: #fff;
  }
  
  .new-comment-editor :deep(.ql-toolbar) {
    border: none;
    border-bottom: 0.08vw solid #ccd0d5;
    background-color: #f7f8fa;
    padding: 0.8vw;
  }
  
  .new-comment-editor :deep(.ql-container) {
    border: none;
    font-size: 0.9vw;
  }
  
  .new-comment-editor :deep(.ql-editor) {
    padding: 1.2vw;
    min-height: 10vw;
    max-height: 10vw;
    font-size: 0.9vw;
    line-height: 1.6;
    overflow-y: auto;
  }
  .ql-editor{
    min-height: 7vw;
  }
  .new-comment-editor :deep(.ql-editor img) {
    max-width: 100% !important;
    height: auto !important;
  }
  
  .new-comment-actions {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 1vw;
  }
  
  .custom-checkbox-container {
    display: flex;
    align-items: center;
    gap: 0.5vw;
    cursor: pointer;
  }
  
  .custom-checkbox {
    width: 1.2vw;
    height: 1.2vw;
    border: 0.08vw solid #ccd0d5;
    border-radius: 0.2vw;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s;
  }
  
  .custom-checkbox.checked {
    background-color: #1877f2;
    border-color: #1877f2;
  }
  
  .custom-checkbox span {
    color: #fff;
    font-size: 0.9vw;
    font-weight: bold;
  }
  
  .custom-checkbox-container label {
    font-size: 0.85vw;
    color: #65676b;
    cursor: pointer;
  }
  
  .submit-btn {
    background-color: #1877f2;
    color: #fff;
    border: none;
    padding: 0.7vw 1.8vw;
    border-radius: 0.4vw;
    font-size: 0.9vw;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.2s;
  }
  
  .submit-btn:hover:not(:disabled) {
    background-color: #166fe5;
  }
  
  .submit-btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
  
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: -webkit-fill-available;
    background-color: rgba(0, 0, 0, 0.75);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 10000;
    overflow-y: auto;
  }
  
  .modal-content {
    background-color: #fff;
    width: 70vw;
    max-width: 70vw;
    max-height: 85vh;
    border-radius: 0.8vw;
    box-shadow: 0 1vw 3vw rgba(0, 0, 0, 0.5);
    display: flex;
    flex-direction: column;
    position: relative;
    margin: auto;
  }
  
  .modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1.5vw 2vw;
    border-bottom: 0.08vw solid #e4e6eb;
    background-color: #f7f8fa;
  }
  
  .modal-title {
    font-size: 1.2vw;
    font-weight: 600;
    color: #1c1e21;
    margin: 0;
  }
  
  .modal-close-btn {
    background: none;
    border: none;
    font-size: 1.8vw;
    color: #65676b;
    cursor: pointer;
    width: 2.5vw;
    height: 2.5vw;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 50%;
    transition: all 0.2s;
  }
  
  .modal-close-btn:hover {
    background-color: #e4e6eb;
    color: #1c1e21;
  }
  
  .modal-body {
    flex: 1;
    overflow-y: auto;
    padding: 0;
  }
  
  .reply-editor {
    height: 100%;
  }
  
  .reply-editor :deep(.ql-toolbar) {
    border: none;
    border-bottom: 0.08vw solid #ccd0d5;
    background-color: #f7f8fa;
    padding: 1vw 2vw;
  }
  
  .reply-editor :deep(.ql-container) {
    border: none;
    font-size: 0.95vw;
  }
  
  .reply-editor :deep(.ql-editor) {
    padding: 2vw;
    min-height: 20vw;
    max-height: 20vw;
    font-size: 0.95vw;
    line-height: 1.6;
    overflow-y: auto;
  }
  
  .reply-editor :deep(.ql-editor img) {
    max-width: 100% !important;
    height: auto !important;
  }
  
  .modal-footer {
    display: flex;
    justify-content: flex-end;
    gap: 1vw;
    padding: 1.5vw 2vw;
    border-top: 0.08vw solid #e4e6eb;
    background-color: #f7f8fa;
  }
  
  .modal-btn {
    padding: 0.7vw 1.8vw;
    border-radius: 0.4vw;
    border: none;
    font-size: 0.9vw;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.2s;
  }
  
  .modal-btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
  
  .btn-cancel {
    background-color: #e4e6eb;
    color: #1c1e21;
  }
  
  .btn-cancel:hover:not(:disabled) {
    background-color: #d8dadf;
  }
  
  .btn-submit {
    background-color: #1877f2;
    color: #fff;
  }
  
  .btn-submit:hover:not(:disabled) {
    background-color: #166fe5;
  }
  
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
  
  .ql-font-serif { font-family: 'serif'; }
  .ql-font-monospace { font-family: 'monospace'; }
  .ql-font-microsoft-yahei { font-family: 'Microsoft YaHei', sans-serif; }
  .ql-font-arial { font-family: 'Arial', sans-serif; }
  .ql-font-times-new-roman { font-family: 'Times New Roman', serif; }
  </style>
  
  
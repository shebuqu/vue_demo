<template>
  <div class="col-md-9 left-col pull-right">
    <div class="panel article-body content-body">
      <h1 class="text-center">{{ title }}</h1>
      <div class="article-meta text-center">
        <i class="fa fa-clock-o"></i>
        <abbr>{{ date | moment('from') }}</abbr>
      </div>
      <div class="entry-content">
        <div class="content-body entry-content panel-body ">
          <div class="markdown-body" v-html="content"></div>

          <div v-if="auth && uid === 1" class="panel-footer operate">
            <div class="actions">
              <a @click="deleteArticle" class="admin" href="javascript:;"><i class="fa fa-trash-o"></i></a>
              <a @click="editArticle" class="admin" href="javascript:;"><i class="fa fa-pencil-square-o"></i></a>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="votes-container panel panel-default padding-md">
      <div class="panel-body vote-box text-center">
        <div class="btn-group">
          <a @click="like" href="javascript:;" class="vote btn btn-primary popover-with-html" :class="likeClass">
            <i class="fa fa-thumbs-up"></i> {{ likeClass ? '已赞' : '点赞' }}
          </a>
        </div>
        <div class="voted-users">
          <div class="user-lists">
            <span v-for="likeUser in likeUsers">
              <!-- 点赞用户是当前用户时，加上类 animated 和 swing 以显示一个特别的动画  -->
              <img :src="user && user.avatar" class="img-thumbnail avatar avatar-middle" :class="{ 'animated swing' : likeUser.uid === 1 }">
            </span>
          </div>
          <div v-if="!likeUsers.length" class="vote-hint">成为第一个点赞的人吧 😄</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import SimpleMDE from 'simplemde'
import hljs from 'highlight.js'
import emoji from 'node-emoji'
import { mapState } from 'vuex'

export default {
  name: 'Content',
  data() {
    return {
      title: '', // 文章标题
      content: '', // 文章内容
      date: '', // 文章创建时间
      uid: 1, // 用户 ID
      likeUsers: [], // 点赞用户列表
      likeClass: '', // 点赞样式
    }
  },
  computed: {
    ...mapState([
      'auth',
      'user'
    ])
  },
  created() {
    const articleId = this.$route.params.articleId
    const article = this.$store.getters.getArticleById(articleId)

    if (article) {
      let { uid, title, content, date, likeUsers } = article

      this.uid = uid
      this.title = title
      this.content = SimpleMDE.prototype.markdown(emoji.emojify(content, name => name))
      this.date = date

      this.likeUsers = likeUsers || []
      this.likeClass = this.likeUsers.some(likeUser => likeUser.uid === 1) ? 'active' : ''

      this.$nextTick(() => {
        this.$el.querySelectorAll('pre code').forEach((el) => {
          hljs.highlightBlock(el)
        })
      })
    }

    this.articleId = articleId
  },
  methods: {
    editArticle() {
      this.$router.push({ name: 'Edit', params: { articleId: this.articleId } })
    },
    deleteArticle() {
      this.$swal({
        text: '你确定要删除此内容吗?',
        confirmButtonText: '删除'
      }).then((res) => {
        if (res.value) {
          this.$store.dispatch('post', { articleId: this.articleId })
        }
      })
    },
    like(e) {
      // 未登录时，提示登录
      if (!this.auth) {
        this.$swal({
          text: '需要登录以后才能执行此操作。',
          confirmButtonText: '前往登录'
        }).then((res) => {
          if (res.value) {
            this.$router.push('/auth/login')
          }
        })
      } else {
        const target = e.currentTarget
        // 点赞按钮是否含有 active 类，我们用它来判断是否已赞
        const active = target.classList.contains('active')
        const articleId = this.articleId

        if (active) {
          // 清除已赞样式
          this.likeClass = ''
          // 分发 like 事件取消赞，更新实例的 likeUsers 为返回的值
          this.$store.dispatch('like', { articleId }).then((likeUsers) => {
            this.likeUsers = likeUsers
          })
        } else {
          // 添加已赞样式
          this.likeClass = 'active animated rubberBand'
          // 分发 like 事件，传入 isAdd 参数点赞，更新实例的 likeUsers 为返回的值
          this.$store.dispatch('like', { articleId, isAdd: true }).then((likeUsers) => {
            this.likeUsers = likeUsers
          })
        }
      }
    }
  }
}
</script>

<style scoped>

</style>
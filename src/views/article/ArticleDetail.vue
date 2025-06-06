<template>
  <div class="cotainer" ref="body">
    <div class="filter"></div>
    <TopNavBar />
    <figure class="banner" v-if="article">
      <MyElimage :img="article.banner" :zip="4" />
    </figure>
    <main class="main">
      <article class="article">
        <ThemeConfig />
        <div class="main-body" v-if="article">
          <transition name="onlyopacity">
            <ToolBars
              v-if="isToolsShow"
              :article="article"
              @like-article="likedArticle"
              @collect-article="collectArticle"
              @toComment="toComment"
            />
          </transition>
          <!-- 标题和摘要 -->
          <div class="article-container" ref="articleContainer">
            <ArticleSummary :article="article" />
            <!-- 主体部分 -->
            <ArticleBody :html="article.body.html" />
            <!-- 结束部分 显示查看数 点赞数 以及评论数 -->
            <ElDivider />
          </div>
          <BottomItem
            :article="article"
            @like-article="likedArticle"
            @collect-article="collectArticle"
          />
          <!-- 评论区 -->
          <div class="comments" ref="comment">
            <div class="edit-part">
              <div class="title">
                <p>{{ totalComment }}条评论</p>
              </div>
              <ElDivider />
              <div class="main-content">
                <div class="avatar">
                  <MyElimage :img="user?.avatar" :zip="2" />
                </div>
                <div class="edit-area">
                  <V3Emoji
                    :textArea="true"
                    :customSize="customSize"
                    :recent="true"
                    :keep="true"
                    :optionsName="optionsName"
                    :disableGroup="disableGroup"
                    v-model="commentParams.content"
                    :customTheme="customTheme"
                  />
                </div>
              </div>
              <div class="button">
                <ElButtonGroup>
                  <ElButton
                    @click="publishComment"
                    class="buttonself"
                    :disabled="user?.id === ''"
                    type="success"
                  >
                    发布评论
                  </ElButton>
                </ElButtonGroup>
              </div>
            </div>
            <AdkEmpty
              v-if="commentList.length === 0"
              desc="还没有评论哦~快来发送第一条吧"
            ></AdkEmpty>
            <div class="comment-list">
              <transition-group name="list">
                <CommentItem
                  class="list-item"
                  v-for="(commentitem, i) in commentList"
                  :commentInfo="commentitem"
                  :floor="(pageparams.page - 1) * pageparams.pagesize + i + 1"
                  :key="commentitem.id"
                  :authorId="article.authorVo.id"
                  :articleId="article.id"
                  @published="publishSecond"
                />
              </transition-group>
            </div>
          </div>
          <!-- 评论分页 -->
          <MyPagination
            :pageParams="pageparams"
            :total="totalComment"
            @changePage="changePage"
            class="page"
          />
        </div>
        <AdkEmpty desc="努力加载中" v-else />
      </article>
    </main>
    <FooterItem />
  </div>
</template>

<script setup lang="ts">
// 定义组件名字 不然include 和keepalive 无法生效
import V3Emoji from 'vue3-emoji'
import 'vue3-emoji/dist/style.css'
import { useArticle } from '@/hooks/Article'
import ArticleSummary from './components/ArticleSummary.vue'
import ArticleBody from './components/ArticleBody.vue'
import BottomItem from './components/BottomItem.vue'
import ToolBars from './components/ToolBars.vue'
import { setConfig } from '@/theme/theme'
import { useHead } from '@vueuse/head'
import { useEmoji } from '@/hooks/useEmoji'
const {
  publishSecond,
  publishComment,
  article,
  commentList,
  user,
  likedArticle,
  collectArticle,
  totalComment,
  pageparams,
  commentParams,
  changePage
} = useArticle()
const { optionsName, disableGroup, customSize, customTheme } = useEmoji()
const comment = ref<HTMLElement>()
const articleContainer = ref<HTMLElement>()
const isToolsShow = ref(false)
const toComment = () => {
  let height = comment.value.offsetTop
  document.documentElement.scrollTo({ top: height, behavior: 'smooth' })
}
useHead({
  // Can be static or computed
  title: computed(
    () =>
      `${
        article.value?.articleName ? article.value.articleName : '文章详情'
      } - ADKBlog-我的个人小站`
  ),
  meta: [
    {
      name: `description`,
      content: computed(() => (article.value?.summary ? article.value.summary : '描述'))
    },
    {
      name: `author`,
      content: computed(
        () =>
          `文章作者:${
            article.value?.authorVo?.nickname ? article.value.authorVo.nickname : '作者名字'
          }`
      )
    }
  ]
})
// 刷新的时候工具栏出现的bug
onMounted(() => {
  document.addEventListener('scroll', () => {
    if (articleContainer.value) {
      if (window.scrollY > articleContainer.value.offsetTop) {
        isToolsShow.value = true
      } else {
        isToolsShow.value = false
      }
    }
  })
  setConfig()
})
</script>

<style scoped lang="less">
@import url(@/assets/styles/Layout/Layout.less);
@import url(./styles/Article.less);
@import url('@/assets/styles/MyAnimate.less');
:deep(.emoji-textarea) {
  textarea {
    .border-normal();
    background-color: @bgColor;
  }
  .pollup {
    background-color: @bgColor;
    .shadow();
    .tab-container {
      background-color: @bgColor;
    }
  }
}
</style>

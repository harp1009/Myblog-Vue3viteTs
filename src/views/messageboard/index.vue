<template>
  <div class="message" ref="body">
    <div class="up-info">
      <div class="logo">
        <div style="width: 60px">
          <MyElimage :img="MessageLogo" />
        </div>
        <p class="title">留言板</p>
      </div>
      <p class="desc">输入你的昵称 联系方式 可以给我的网站留言哦~</p>
    </div>
    <div class="edit-part">
      <div class="liuyan-info">
        <div class="name">
          <span>您的昵称：</span>
          <ElInput size="large" v-model="messageParams.authorName" show-word-limit maxlength="16">
          </ElInput>
        </div>
        <div class="name">
          <span>您的联系方式：</span>
          <ElInput
            class="contact"
            size="large"
            v-model="messageParams.contact"
            show-word-limit
            maxlength="64"
            placeholder="example@adkdream.top"
          >
          </ElInput>
        </div>
      </div>
      <ElDivider />
      <div class="main-content">
        <div class="msg-avatar">
          <UploadAvatar
            :avatar="messageParams.avatar ? messageParams.avatar : DefaultAvatar"
            @imglink="changeAvatarParams"
          />
        </div>
        <div class="edit-area">
          <V3Emoji
            :textArea="true"
            :customSize="customSize"
            :recent="true"
            :keep="true"
            :optionsName="optionsName"
            :disableGroup="disableGroup"
            v-model="messageParams.content"
            :customTheme="customTheme"
          />
        </div>
      </div>
      <div class="button">
        <ElButton @click="publishMessage" class="buttonself" type="success">发布留言</ElButton>
      </div>
    </div>

    <!-- 留言列表 -->
    <div class="paixu">
      <h2>留言列表</h2>
      <p :class="{ active: orderRole === 2 }" @click="order(2)">默认</p>
      <p :class="{ active: orderRole === 1 }" @click="order(1)">倒序</p>
    </div>

    <div class="message-part" v-if="messageList">
      <CommentItem
        v-for="(item, i) in toCommentItem"
        :commentInfo="item"
        :key="item.id"
        :reply="false"
        :level="false"
        :floor="(pageparams.page - 1) * pageparams.pagesize + i + 1"
      />
    </div>
    <AdkEmpty v-else desc="暂时没有留言哦~"></AdkEmpty>

    <MyPagination
      :pageParams="pageparams"
      @changePage="changePage"
      :total="total"
      class="pagination"
    />
  </div>
</template>

<script lang="ts">
export default {
  name: 'MessageBoard'
}
</script>

<script lang="ts" setup>
import V3Emoji from 'vue3-emoji'
import 'vue3-emoji/dist/style.css'
import MessageLogo from '@/assets/img/liuyan-logo.png'
import { useChangeParams, useMessageApi, useMessageBoardParams } from '@/hooks/useMessageboard'
import DefaultAvatar from '@/assets/img/logo.png'
import { useEmoji } from '@/hooks/useEmoji'
const { messageParams, pageparams, messageList, total } = useMessageBoardParams()

const { orderRole, publishMessage, order, changePage, body } = useMessageApi(
  messageParams,
  pageparams,
  messageList,
  total
)
const { changeAvatarParams, toCommentItem } = useChangeParams(messageParams, messageList)
const { optionsName, disableGroup, customSize, customTheme } = useEmoji()
</script>

<style lang="less" scoped>
@import url(./styles/MessageBoard.less);
@import url(@/assets/styles/MyAnimate.less);
</style>

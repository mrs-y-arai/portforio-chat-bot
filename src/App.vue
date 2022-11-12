<template>
  <div class="chat-wrap">
    <!-- 画面上部のチャット部分 ---leftクラスが付くと、左側にコメントが表示され、---rightクラスが付くと、右側にコメントが表示される。 -->
    <ul ref="chatList" class="chat-list">
      <li class="chat-item"
          v-for="(message, i) in messages" :key="i"
          v-bind:class="{ '---left': message.position === 'left', '---right': message.position === 'right', }">
        <div class="chat-item__icon-wrap"></div>
        <div v-html="message.text" class="chat-box__bubble"></div>
      </li>
    </ul>
    <div class="btn-wrap">
      <ul class="btn-list">
        <!-- ユーザーからのコメント -->
        <li
        class="btn-item"
        v-for="(userComment, i) in userComments" :key="i"
        >
          <button
          class="btn"
          @click="userCommentSubmit"
          >
          <span>{{userComment.id}}</span>.
          {{userComment.text}}
          </button>
        </li>

        <!-- ユーザーコメント後のアクション -->
        <li
        class="btn-item"
        v-if="userAction"
        >
          <button
          class="btn"
          @click="userActionMethod()"
          >
          {{userAction}}
          </button>
          <form v-if=" userCommentId === 1 || userCommentId === 2 " 
                class="btn-item__form" action="https://smz97hiro.xsrv.jp/contact/" 
                method="get" rel="noopener noreferrer" target="_blank">
            <input type="text" name="type" :value="userCommentId">
            <input ref="submitType" type="submit" value="送信">
          </form>
        </li>
        <li
        class="btn-item"
        v-if="userAction"
        >
          <!-- 画面下の選択肢を初期状態に戻す。 -->
          <button
          class="btn"
          @click="reset"
          >
          最初に戻る
          </button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      messages: [
        {
          text: "初めまして！ご用件は何でしょうか？",
          position: 'left',
        },        
      ],
      userComments: [
        {
          id: 1,
          text: '採用を検討しているのでお話をしたい',
        },
        {
          id: 2,
          text: '仕事の依頼をしたい',
        },
        {
          id: 3,
          text: '経歴を知りたい',
        },
        {
          id: 4,
          text: '制作実績を見たい',
        },
      ],
      // デフォルトのユーザーコメント退避用リスト
      evacuationUserComments: [],
      userCommentId: '',
      reply: [
        '採用のご検討、ありがとうございます！<br>採用のお問い合わせは、以下ボタンからお願い致します。',
        'お仕事のご依頼、ありがとうございます！お仕事のご依頼は、以下ボタンからお願い致します。',
        '経歴にご興味いただき、ありがとうございます！経歴の詳細は、以下ボタンからお願い致します。',
        '制作実績にご興味いただき、ありがとうございます！制作実績の詳細は、以下ボタンからお願い致します。',
      ],
      userActionArray: [
        '採用のお問い合わせはこちら',
        'お仕事のご依頼はこちら',
        '経歴ページはこちら',
        '制作実績ページはこちら'
      ],
      userAction: '',
    }
  },
  methods: {
    // コメント表示後画面スクロール
    screenScroll() {
      const chatScrollHeight = this.$refs.chatList.scrollHeight
      console.log(chatScrollHeight)
      this.$refs.chatList.scrollTop = chatScrollHeight
    },
    // ユーザーのコメント反映
    userCommentPush(e) {
      return new Promise((resolve) => {
        this.messages.push({
          text: e.target.innerHTML,
          position: 'right'
        })
        resolve()
      })
    },
    // ユーザーへの返信
    ownerReply(id) {
      return new Promise((resolve) => {        
        id = id - 1
        this.messages.push({
          text: this.reply[id],
          position: 'left'
        })
        resolve()
      })
    },
    // ユーザーからのコメント・ユーザーのコメントへの返信
    async userCommentSubmit(e) {
      // コメントを反映
      await this.userCommentPush(e)
      .then(() => {
        // コメント反映後、画面スクロール
        console.log('コメント反映成功')
        this.screenScroll()
      })
      .catch((error) => {
        console.log('コメント反映失敗', error)
      })

      // ユーザーコメントIDを、数値に変換
      this.userCommentId = await Number(e.target.children[0].textContent)

      // ユーザーからのコメントに対して返信をする
      setTimeout( async() => {
        await this.ownerReply(this.userCommentId)
        .then(() => {
          // コメント反映後、画面スクロール
          console.log('返信成功')

          // 次のアクション表示処理
          // ユーザーコメントIDによってコメント表示が変わる
          if( this.userCommentId === 1 || this.userCommentId === 2 || 
              this.userCommentId === 3 || this.userCommentId === 4 ){
            const id = this.userCommentId - 1
            this.evacuationUserComments = this.userComments.splice(0)
            this.userAction = this.userActionArray[id]
          }else{
            console.log('ユーザーコメントIDが不正です。')
            return false
          }
          this.screenScroll()
        })
        .catch((error) => {
          console.log('返信失敗', error)
        })
        this.screenScroll()
      }, 500)
    },
    userActionMethod() {
      if( this.userCommentId === 1 || this.userCommentId === 2 ){            
        this.$refs.submitType.click()
      }else if( this.userCommentId === 3 ){
        window.open('https://smz97hiro.xsrv.jp/about/','_blank','noreferrer','noopener')
      }else if( this.userCommentId === 4 ){
        window.open('https://smz97hiro.xsrv.jp/works/','_blank','noreferrer','noopener')
      }
    },
    // 最初の選択肢に戻る
    reset() {
      this.userAction = ''
      this.userCommentId = 0
      this.userComments = this.evacuationUserComments.splice(0)
    },
  },
}
</script>

<style>
body {
  margin: 0;
}
ul {
  margin: 0;
  padding: 0;
}
li {
  list-style: none;
}
a {
  color: #4169e1;
  transition-duration: 0.5s;
  font-weight: 700;
}
a:hover {
  opacity: 0.8;
}
button {
  border: none;
  background: transparent;
}
#app {
  font-family: 'Spinnaker', sans-serif;  
  text-align: center;
  color: #2c3e50;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.chat-wrap {
  border: 3px solid #e0b314;
  margin: 0 auto;
  width: 470px;
  height: 680px;
  border-radius: 50px;
  padding: 1.3%;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
  align-items: center;
  overflow: hidden;
}
@media screen and (max-width: 600px) {
  .chat-wrap  {
    width: 90vw;
    height: 95vh;
  }
}

.chat-list {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 450px;
  height: 410px;
  overflow-y: scroll;
  border-radius: 50px;
  box-sizing: border-box;
  padding: 1.5%;
  margin-bottom: 20px;
}
@media screen and (max-width: 600px) {
  .chat-list {
    width: 100%;
    height: 70%;
    margin-bottom: 10px;
  }
}
.chat-item {
  display: flex;
  align-items: center;
  margin-bottom: 30px;
  width: 100%;
  height: auto;
}
@media screen and (max-width: 600px) {
  .chat-item {
    margin-bottom: 15px;
  }
}
.chat-item.---right {
  flex-direction: row-reverse;
}
.chat-item:last-of-type {
  margin-bottom: 0;
}

.chat-item__icon-wrap {
  background-color: #e0b314;
  width: 60px;
  height: 60px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 50%;
}
@media screen and (max-width: 600px) {
  .chat-item__icon-wrap {
    width: 50px;
    height: 50px;
  }
}

.chat-item.---left .chat-item__icon-wrap {
  margin-right: 10px;
  background-image: url('@/assets/profile-img.jpg');
  background-size: cover;
}
@media screen and (max-width: 600px) {
  .chat-item.---left .chat-item__icon-wrap {
    margin-right: 5px;
  }
}

.chat-item.---right .chat-item__icon-wrap {
  margin-left: 10px;
  background-image: url('@/assets/person-icon.png');
  background-size: 78%;
  background-position: center;
  background-repeat: no-repeat;
}
@media screen and (max-width: 600px) {
  .chat-item.---right .chat-item__icon-wrap {
    margin-left: 5px;
  }
}

.chat-box__bubble {
  background-color: #e0b314;
  color: #fff;
  padding: 3%;
  text-align: start;
  border-radius: 10px;
  width: auto;
  max-width: 300px;
}
@media screen and (max-width: 600px) {
  .chat-box__bubble {
    max-width: 65%;
  }
}

/* 選択肢 */
.btn-wrap {
  width: 100%;
  height: 176px;
  box-sizing: border-box;
}
@media screen and (max-width: 600px) {
  .btn-wrap {
    height: 190px;
  }
}
.btn-list {
  width: 100%;
  display: flex;
  height: 100%;
  flex-direction: column;
  align-items: center;
}

.btn-item {
  width: 100%;
  height: 40px;
  margin-bottom: 4px;
}
@media screen and (max-width: 600px) {
  .btn-item {
    height: 41.5px;
    margin-bottom: 6px;
  }
}
.btn-item:last-of-type {
  margin: 0;
}
.btn {
  background: transparent;
  border: 2px solid #e0b314;
  box-sizing: border-box;
  color: #e0b314;
  font-size: 1rem;
  font-weight: 700;
  height: 100%;
  width: 90%;
  cursor: pointer;
  border-radius: 17px;
  z-index: 1;
  position: relative;
  overflow: hidden;
  transition-duration: 0.5s;
}
@media screen and (max-width: 600px) {
  .btn {
    font-size: 0.8rem;
  }
}

.btn::before {
  content: '';
  height: 38px;
  width: 400px;
  display: block;
  background-color: #e0b314;
  padding: 1px 6px;
  position: absolute;
  top: 0;
  left: 0;
  z-index: -1;
  transform: translateX(-450px);
  transition-duration: 0.5s;
}
@media screen and (max-width: 600px) {
  .btn::before {
    display: none;
  }
}
.btn:hover {
  color: #fff;
}
@media screen and (max-width: 600px) {
  .btn:hover {
    color: #e0b314;
  }
}
.btn:hover::before {
  transform: translateX(0);
}
.btn-item__form {
  display: none;
}
</style>

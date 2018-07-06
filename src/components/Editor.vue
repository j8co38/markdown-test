<template>
  <div class="editor">
    <h1>エディター画面</h1>
    <span>{{ user.displayName }}</span>
    <button class="logoutBtn" @click="logout">ログアウト</button>
    <div class="memoSection">
      <div class="memoListWrapper">
        <button class="addMemoBtn" @click="addMemo">メモの追加</button>
        <div
          class="memoList"
          v-for="(memo, index) in memos"
          :key="`memo-${index}`"
          @click="selectMemo(index)"
          :data-selected="index == selectedIndex"
        >
          <p class="memoTitle">{{ displayTitle(memo.markdown) }}</p>
        </div>
        <button class="deleteMemoBtn" v-if="memos.length" @click="deleteMemo">選択中のメモの削除</button>
      </div>
      <div class="textAreaWrapper">
        <div class="inputArea">
          <textarea class="markdown" v-model="memos[selectedIndex].markdown"></textarea>
          <div class="preview" v-html="preview()"></div>
        </div>
        <button class="saveMemosBtn" @click="saveMemos">{{ (!isSaving) ? "メモの保存" : "保存中..." }}</button>
      </div>
    </div>
  </div>
</template>

<script>
  import marked from 'marked';
  export default {
    name: 'editor',
    props: [
      'user'
    ],
    data () {
      return {
        memos:[{
          markdown: ''
        }],
        selectedIndex: 0,
        isSaving: false
      }
    },
    created: function() {
      firebase
        .database()
        .ref('memos/' + this.user.uid)
        .once('value')
        .then(result => {
          if (result.val()) {
            this.memos = result.val();
          }
        })
    },
    methods: {
      logout: function() {
        firebase.auth().signOut();
      },
      addMemo: function() {
        this.memos.push({
          markdown: '無題のメモ',
        })
      },
      selectMemo: function(index) {
        this.selectedIndex = index;
      },
      preview: function() {
        return marked(this.memos[this.selectedIndex].markdown);
      },
      displayTitle: function(text) {
        return text.split(/\n/)[0];
      },
      deleteMemo: function() {
        this.memos.splice(this.selectedIndex, 1);
        if (this.selectedIndex > 0) {
          this.selectedIndex--;
        }
        firebase
          .database()
          .ref('memos/' + this.user.uid)
          .set(this.memos);
      },
      saveMemos: function() {
        if (this.isSaving) {
          return;
        }
        this.isSaving = true;
        firebase
          .database()
          .ref('memos/' + this.user.uid)
          .set(this.memos)
          .then(res => {
            this.isSaving = false;
          });
      }
    }
  }
</script>
<style lang="scss" scoped>
  @import '../scss/variable';


  .logoutBtn {
    font-size: 12px;
    margin-left: 8px;
    padding: 4px 8px;
    color: white;
    background-color: rgba(0, 10, 10, 0.4);
  }

  .editor {
    width: 90%;
    max-width: 90%;
    margin: 0 auto;
  }

  .memoSection {
    display: flex;
    margin-top: 40px;
  }

  .memoListWrapper {
    display: flex;
    flex-direction: column;
    width: 280px;
    min-width: 1;
  }

  .memoList {
    box-sizing: border-box;
    padding: 16px 10px;
    cursor: pointer;
    transition: all .2s;
    text-align: left;
    border-bottom: 1px solid rgba(0, 10, 10, 0.4);

    &:hover {
      background-color: rgba(0, 10, 10, 0.1);
    }

    &[data-selected='true'] {
      color: #2d2d2d;
      background-color: $color-green;
    }
  }

  .memoTitle {
    overflow: hidden;
    height: 1.5em;
    margin: 0;
    white-space: nowrap;
  }

  .addMemoBtn {
    font-size: 16px;
    font-weight: 600;
    width: 100%;
    margin-bottom: 20px;
    padding: 10px 0;
    transition: all .2s;
    color: white;
    background-color: $color-pink;
    box-shadow: 0 8px 8px rgba(0, 10, 10, 0.2);

    &:hover {
      transform: translateY(2px);
      background-color: darken($color-pink, 4%);
      box-shadow: none;
    }
  }

  .textAreaWrapper {
    display: flex;
    flex: 4;
    flex-direction: column;
    align-items: center;
    margin-left: 2%;

    .inputArea {
      flex: 1;
      display: flex;
      width: 100%;
    }
  }

  .markdown {
    flex: 1;
    font-size: 16px;
    line-height: 1.5;
    width: 49%;
    min-width: 1;
    height: 600px;
    padding: 20px;
    color: #2d2d2d;
    border-color: rgba(0, 10, 10, 0.2);
    border-radius: 8px;
  }

  .preview {
    flex: 1;
    width: 49%;
    min-width: 1;
    margin-left: 2%;
    padding: 20px 32px;
    text-align: left;
    word-wrap: break-word;
    border-radius: 8px;
    background-color: rgba(0, 10, 10, 0.4);
  }

  .deleteMemoBtn {
    font-size: 14px;
    margin: 20px 0 0 auto;
    padding: 10px 16px;
    color: white;
    background-color: rgba(255, 255, 255, 0.08);
  }

  .saveMemosBtn {
    font-size: 16px;
    font-weight: 600;
    width: 300px;
    margin-top: 30px;
    padding: 10px;
    color: #2d2d2d;
    background-color: $color-blue;
  }
</style>

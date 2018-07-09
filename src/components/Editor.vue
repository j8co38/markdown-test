<template>
  <div class="editor">
    <h1>エディター画面</h1>
    <span><i class="fas fa-user-circle"></i>{{ user.displayName }}</span>
    <button class="logoutBtn" @click="logout"><i class="fas fa-sign-out-alt"></i>ログアウト</button>
    <div class="memoSection">
      <div class="memoListWrapper">
        <button class="addMemoBtn" @click="addMemo"><i class="fas fa-pencil-alt"></i>メモの追加</button>
        <div class="memoList">
          <div
            class="memoListItem"
            v-for="(memo, index) in memos"
            :key="`memo-${index}`"
            @click="selectMemo(index)"
            :data-selected="index == selectedIndex"
          >
            <p class="memoTitleWrap"><span class="memoTitle">{{ displayTitle(memo.markdown) }}</span><i class="fas fa-angle-right"></i></p>
          </div>
        </div>
        <button class="deleteMemoBtn" v-if="memos.length > 1" @click="deleteMemo"><i class="fas fa-trash-alt"></i>選択中のメモの削除</button>
      </div>
      <div class="textAreaWrapper">
        <div class="inputArea">
          <textarea class="markdown" v-model="memos[selectedIndex].markdown"></textarea>
          <div class="preview" v-html="preview()"></div>
        </div>
        <button class="saveMemosBtn" @click="saveMemos"><i v-if="!isSaving" class="fas fa-save"></i>{{ (!isSaving) ? "メモの保存" : "保存中..." }}</button>
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

  button {
    transition: all .2s;
  }

  .fas {
    margin-right: 4px;
  }

  .logoutBtn {
    font-size: 12px;
    margin-left: 8px;
    padding: 4px 10px;
    color: white;
    background-color: rgba($color-black, 0.64);

    &:hover {
      background-color: rgba($color-black, 0.72);
    }
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
    overflow-y: auto;
    height: 580px;
  }

  .memoListItem {
    box-sizing: border-box;
    padding: 16px 10px;
    cursor: pointer;
    transition: all .2s;
    text-align: left;
    border-bottom: 1px solid rgba($color-black, 0.16);

    &:hover {
      background-color: rgba($color-black, 0.04);
    }

    &[data-selected='true'] {
      color: #2d2d2d;
      background-color: $color-green;
    }
  }

  .memoTitleWrap {
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 1.5em;
    margin: 0;

    .memoTitle {
      overflow: hidden;
      margin: 0;
      white-space: nowrap;
      text-overflow: ellipsis;
    }

    .fas {
      margin: 0 0 0 8px;
    }
  }

  .addMemoBtn {
    font-size: 16px;
    font-weight: 600;
    width: 100%;
    margin-bottom: 20px;
    padding: 10px 0;
    color: white;
    background-color: $color-pink;
    box-shadow: 0 4px 8px rgba($color-black, 0.1);

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
    border: 1px solid rgba($color-black, 0.16);
    background-color: rgba(245, 245, 242, 1);
  }

  .preview {
    flex: 1;
    width: 49%;
    min-width: 1;
    height: 600px;
    padding: 20px 32px;
    text-align: left;
    word-wrap: break-word;
    border: 1px solid rgba($color-black, 0.16);
    border-left: none;
    background-color: white;
  }

  .deleteMemoBtn {
    font-size: 14px;
    margin: 20px 0 0 auto;
    padding: 10px 16px;
    color: rgba($color-black, 0.64);
    border: 1px solid rgba($color-black, 0.16);

    &:hover {
      background-color: rgba($color-black, 0.08);
    }
  }

  .saveMemosBtn {
    font-size: 16px;
    font-weight: 600;
    width: 300px;
    margin-top: 30px;
    padding: 10px;
    color: #2d2d2d;
    background-color: $color-blue;

    &:hover {
      background-color: darken($color-blue, 4%);
    }
  }
</style>

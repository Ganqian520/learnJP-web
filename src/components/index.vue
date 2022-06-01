<script setup>
import { ref, onMounted, toRaw } from "vue";
import { post } from "../api/index";
import anchor from "./anchor.vue";
import popEdit from "./popEdit.vue";

const collections = ref([]); //所有集合名
const sentences = ref([]); //每个集合下的所有句子
const words = ref([]); //句子包含的单词
const sentence = ref({}); //当前展示的句子对象
const indexCollection = ref(0); //集合序号
const indexSentence = ref(0); //句子序号
const isShowPop = ref(false); //弹窗开关
const isEditSentence = ref(false); //句子编辑开关
const isEditWord = ref(false); //单词编辑开关
const elSentence = ref({}); //句子三项元素dom
const elJM = ref({}); //单词dom集合,假名,用对象代替数组
const elCN = ref({}); //中文

let indexWaitAdd = -1; //待添加句子序号
let isAddSentence = false; //记录应该添加句子还是修改句子

const getCollectionName = async () => {
  //获取集合名
  let res= await post({ action: "getCollectionName" });
  if (res.data) collections.value = res.data;
};
const getSentences = async () => {
  //获取句子
  let res = await post({
    action: "getCollection",
    collection: collections.value[indexCollection.value].collection,
  });
  if (res.data) sentences.value = res.data[0].sentences;
};
const getWords = async () => {
  //获取单词
  words.value = [];
  if (!sentence.value.jp) return;
  let words_ = sentence.value.jp.split("　");
  let res = await post({ action: "getWords", list: words_ });
  words.value = words_.map((item, index) => {
    let obj = { jp: item, cn: "", jm: "" };
    let value = res.data.find((item_) => {
      if (item_.jp === item) return item_;
    }, item);
    if (value) {
      obj.cn = value.cn;
      obj.jm = value.jm;
    }
    return obj;
  });
};
const addSentenceTemp = () => {
  //临时添加句子
  if (indexWaitAdd == -1) {
    sentences.value.push({});
    indexWaitAdd = sentences.value.length - 1;
  }
};
const changeSentence = (index) => {
  //切换句子
  indexSentence.value = index;
  isAddSentence = index == indexWaitAdd;
  sentence.value = sentences.value[index] || {};
  getWords();
};
const okSentence = async () => {
  //句子处确定
  sentence.value.jp = elSentence.value.jp.innerHTML;
  sentence.value.cn = elSentence.value.cn.innerHTML;
  sentence.value.source = elSentence.value.source.innerHTML;
  if (isAddSentence) {
    //添加
    let res = await post({
      action: "addSentence",
      collection: collections.value[indexCollection.value].collection,
      sentence: sentence.value,
    });
    if (res.updated == 1) {
      indexWaitAdd = -1;
      isAddSentence = false;
      isEditSentence.value = false;
      alert("添加成功");
    }
  } else {
    //修改
    let res = await post({
      action: "updateSentence",
      index: indexSentence.value,
      collection: collections.value[indexCollection.value].collection,
      sentence: sentence.value,
    });
    if (res.updated == 1) {
      isEditSentence.value = false;
      alert("修改成功");
    }
  }
};
const addWords = async () => {
  //添加单词
  let list = [];
  words.value.forEach((item, index) => {
    let isHave = item.cn == "" ? false : true;
    item.jm = elJM.value[`${index}`].innerHTML;
    item.cn = elCN.value[`${index}`].innerHTML;
    elCN.value[`${index}`].innerHTML != "" && !isHave && list.push(toRaw(item));
  });
  let res = await post({ action: "addWords", list });
  if (res.inserted > 0) {
    isEditWord.value = false;
    alert(`成功添加${res.inserted}条`);
  }
};
const updateWord = async (index) => {
  //修改单词
  words.value[index].jm = elJM.value[`${index}`].innerHTML;
  words.value[index].cn = elCN.value[`${index}`].innerHTML;
  let word = toRaw(words.value[index]);
  let res = await post({ action: "updateWord", word, jp: word.jp });
  if (res.updated == 1) {
    isEditWord.value = false;
    alert("修改单词成功！");
  }
};
const addCollection = async (input) => {
  //添加句子集合
  let res = await post({
    action: "addCollection",
    collection: input,
    sentences: [],
  });
  if (res.id) {
    isShowPop.value = false;
    alert("添加成功");
  }
};
const changeCollection = async (index) => {
  //切换集合
  indexCollection.value = index;
  await getSentences();
  changeSentence(0);
};

onMounted(async () => {
  await getCollectionName();
  await getSentences();
  sentence.value = sentences.value[indexSentence.value];
  getWords();
});
</script>

<template>
  <div class="index">
    <div class="sentences scroll border">
      <anchor from="collection" @edit="() => (isShowPop = true)" />
      <div
        class="item-sentence hover"
        v-for="(item, index) in collections"
        :key="index"
        @click="changeCollection(index)"
        :style="{
          backgroundColor: index == indexCollection ? 'rgba(0,0,0,0.2)' : '',
        }"
      >
        {{ item.collection }}
      </div>
    </div>

    <div class="indexs scroll border">
      <div
        class="item-index hover"
        v-for="(item, index) in sentences"
        :key="index"
        @click="changeSentence(index)"
        :style="{
          backgroundColor: index == indexSentence ? 'rgba(0,0,0,0.2)' : '',
        }"
      >
        <div class="index-in">{{ index + 1 }}</div>
        <div class="index-cn">{{ item.cn }}</div>
      </div>
      <anchor from="addSentence" @edit="addSentenceTemp" />
    </div>

    <div class="sentence-words">
      <div class="sentence border">
        <anchor
          from="sentence"
          @edit="() => (isEditSentence = !isEditSentence)"
          @confirm="okSentence"
        />
        <div
          class="st-jp"
          :contenteditable="isEditSentence"
          :ref="(el) => (elSentence.jp = el)"
        >
          {{ sentence.jp }}
        </div>
        <div
          class="st-cn"
          :contenteditable="isEditSentence"
          :ref="(el) => (elSentence.cn = el)"
        >
          {{ sentence?.cn }}
        </div>
        <div
          class="st-source"
          :contenteditable="isEditSentence"
          :ref="(el) => (elSentence.source = el)"
        >
          {{ sentence.source }}
        </div>
      </div>

      <div class="bink-w-s"></div>

      <div class="words scroll border">
        <anchor
          from="wordAll"
          style="left: 0; top: 0; z-index: 1"
          @edit="() => (isEditWord = !isEditWord)"
          @confirm="addWords"
        />
        <div
          class="item-word hover"
          v-for="(item, index) in words"
          :key="index"
        >
          <anchor
            from="wordSingle"
            @edit="() => (isEditWord = !isEditWord)"
            @confirm="updateWord(index)"
          />
          <div class="index-word">{{ index + 1 }}</div>
          <div class="jp-jm">
            <div
              class="jm-word"
              :contenteditable="isEditWord"
              :ref="(el) => (elJM[`${index}`] = el)"
            >
              {{ item.jm }}
            </div>
            <div class="jp-word">{{ item.jp }}</div>
          </div>
          <div
            class="cn-word"
            :contenteditable="isEditWord"
            :ref="(el) => (elCN[`${index}`] = el)"
          >
            {{ item.cn }}
          </div>
        </div>
      </div>
    </div>
  </div>
  <pop-edit
    v-if="isShowPop"
    @ClosePop="() => (isShowPop = false)"
    @addCollection="addCollection"
  />
</template>

<style lang='scss' scoped>
.scroll::-webkit-scrollbar {
  width: 10px;
}
.scroll::-webkit-scrollbar-thumb {
  background-color: rgba(0, 0, 0, 0.2);
}
.hover:hover {
  background-color: rgba(0, 0, 0, 0.1);
}
.border {
  box-shadow: 0 0 10px 0 rgba(10, 10, 83, 1);
  box-sizing: border-box;
  border-radius: 1vh;
}
.index {
  color: white;
  background-color: #24478C;
  box-sizing: border-box;
  width: 100vw;
  height: 100vh;
  padding: 10vh;
  padding-left: 5vw;
  padding-right: 5vw;
  display: flex;
  justify-content: space-between;
  position: relative;
  .sentences {
    height: 100%;
    width: 11vw;
    overflow: auto;
    position: relative;
    .item-sentence {
      height: 8vh;
      line-height: 8vh;
      text-overflow: ellipsis;
      white-space: nowrap;
      overflow: hidden;
      padding-left: 1vh;
      padding-right: 1vh;
    }
  }
  .indexs {
    height: 100%;
    width: 16vw;
    position: relative;
    overflow: auto;
    box-sizing: border-box;
    .item-index {
      height: 5vh;
      line-height: 5vh;
      text-overflow: ellipsis;
      white-space: nowrap;
      overflow: hidden;
      display: flex;
      .index-in {
        line-height: 5vh;
        width: 3vh;
        text-align: center;
      }
      .index-cn {
        line-height: 5vh;
        flex: 1;
        text-overflow: ellipsis;
        white-space: nowrap;
        overflow: hidden;
      }
    }
  }
  .sentence-words {
    width: 60vw;
    height: 100%;
    display: flex;
    flex-direction: column;
    .sentence {
      width: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      position: relative;
      .st-jp {
        height: 10vh;
        overflow: auto;
        width: 60vw;
        box-sizing: border-box;
        padding: 1vh;
        font-weight: 900;
      }
      .st-cn {
        height: 5vh;
        line-height: 5vh;
        width: 100%;
        box-sizing: border-box;
        padding-left: 1vh;
      }
      .st-source {
        height: 5vh;
        width: 100%;
        line-height: 5vh;
        box-sizing: border-box;
        padding-left: 1vh;
      }
    }
    .bink-w-s {
      width: 100%;
      height: 2vh;
    }
    .words {
      width: 100%;
      flex: 1;
      display: flex;
      flex-direction: column;
      overflow: auto;
      position: relative;
      .item-word {
        flex-shrink: 0;
        width: 100%;
        height: 7vh;
        display: flex;
        position: relative;
        .index-word {
          width: 7vh;
          line-height: 7vh;
          text-align: center;
        }
        .jp-jm {
          width: 30vh;
          margin-left: 5vh;
          display: flex;
          flex-direction: column;
          box-sizing: border-box;
          padding: 1vh;
          .jm-word {
            height: 2vh;
            line-height: 2vh;
            font-size: 0.8rem;
          }
          .jp-word {
            height: 3vh;
            line-height: 3vh;
            font-weight: 900;
          }
        }
        .cn-word {
          flex: 1;
          line-height: 7vh;
          white-space: nowrap;
          overflow-x: auto;
          overflow-y: hidden;
        }
        .cn-word::-webkit-scrollbar {
          height: 10px;
        }
        .cn-word::-webkit-scrollbar-thumb {
          background-color: rgba(0, 0, 0, 0.2);
        }
      }
    }
  }
}
</style>
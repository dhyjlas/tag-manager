<template>
  <el-affix>
    <el-card>
      <el-input v-model="tagStr" clearable @clear="clear" @input="input" style="margin-bottom: 2px;"></el-input>
      <el-button @click="copyTextToClipboard(tagStr)">复制至剪贴板</el-button>
      <el-button @click="clear">清空</el-button>
    </el-card>
  </el-affix>
  <div v-for="data in dataList" :key="data" style="margin: 20px;">
    <div style="margin-bottom: 2px">{{data.name}}:</div>
    <el-popover v-for="entity in data.list" :key="entity" width="100" placement="top-start" trigger="hover"
      :show-after="500">
      <template #reference>
        <el-checkbox v-model="entity.check" :label="entity.label" @change="change" style="margin-bottom: 1px;" border>
          {{entity.value}}
        </el-checkbox>
      </template>
      <div style="margin-bottom: 2px;">
        {{entity.value}} [ {{entity.label}} ]
      </div>
      <div>
        <el-button @click="setWeight(entity, 0)">0</el-button>
        <el-button @click="setWeight(entity, 1)">1</el-button>
        <el-button @click="setWeight(entity, 2)">2</el-button>
        <el-button @click="setWeight(entity, 3)">3</el-button>
      </div>
    </el-popover>
  </div>
</template>

<script>
  import {
    Search
  } from '@element-plus/icons-vue'
  export default {
    data() {
      return {
        search: "",
        tagStr: "",
        dataList: []
      };
    },
    mounted() {
      TAG_LIST.forEach(tag => {
        const data = {
          name: tag.type,
          list: []
        }
        tag.tags.forEach(e => {
          const s = e.split("|");
          data.list.push({
            label: s[0],
            value: s[1]
          })
        })
        this.dataList.push(data);
      })
    },
    methods: {
      change(e, o) {
        const value = o.target._value;
        let tagList = [];
        if (this.tagStr != null && this.tagStr.length > 0) {
          tagList = this.tagStr.split(",").filter((x) => this.deWeight(x) !== value);
        }
        if (e === true) {
          tagList.push(value);
        }
        this.tagStr = tagList.join(",");
      },
      clear() {
        this.dataList.forEach(data => {
          data.list.forEach(entity => {
            entity.check = false;
          })
        })
        this.tagStr = "";
      },
      input(e) {
        let tagList = [];
        if (e != null && e.length > 0) {
          tagList = e.split(",");
        }
        this.dataList.forEach(data => {
          data.list.forEach(entity => {
            entity.check = false;
            tagList.forEach(e => {
              if (this.deWeight(e) === entity.label) {
                entity.check = true;
              }
            })
          })
        })
      },
      setWeight(entity, weight) {
        let tagList = [];
        if (this.tagStr != null && this.tagStr.length > 0) {
          tagList = this.tagStr.split(",");
        }
        let flag = true;
        for (let i in tagList) {
          if (this.deWeight(tagList[i]) === entity.label) {
            tagList[i] = this.num("{", weight) + entity.label + this.num("}", weight);
            flag = false;
          }
        }
        if (flag) {
          tagList.push(this.num("{", weight) + entity.label + this.num("}", weight));
        }
        entity.check = true;
        this.tagStr = tagList.join(",");
      },
      num(data, n) {
        let s = "";
        for (let i = 0; i < n; i++) {
          s += data;
        }
        return s;
      },
      deWeight(e) {
        e = e.trim();
        if (e.substr(0, 1) === "{" && e.substr(-1) === "}") {
          return this.deWeight(e.substr(1, e.length - 2));
        }
        return e;
      },
      fallbackCopyTextToClipboard() {
        // 1.创建一个可选中元素
        let textArea = document.createElement("textarea");
        textArea.value = window.location.href;
        // 2.使用定位，阻止页面滚动
        textArea.style.top = "0";
        textArea.style.left = "0";
        textArea.style.position = "fixed";
        document.body.appendChild(textArea);
        textArea.focus();
        textArea.select();
        try {
          var successful = document.execCommand('copy');
          var msg = successful ? 'successful' : 'unsuccessful';
        } catch (err) {
          console.error('Fallback: Oops, unable to copy', err);
        }
        // 3.移除元素
        document.body.removeChild(textArea);
      },
      copyTextToClipboard(text) {
        if (!navigator.clipboard) {
          this.fallbackCopyTextToClipboard(text);
          return;
        }
        navigator.clipboard.writeText(text).then(function() {
        }, function(err) {
          console.error('Async: Could not copy text: ', err);
        });
      }
    },
  };
</script>
<style scoped>
  :deep(.el-checkbox.is-bordered) {
    border-radius: 0;
  }

  :deep(.el-checkbox) {
    margin-right: 2px;
  }

  :deep(.el-input__wrapper) {
    border-radius: 0;
  }

  .el-button {
    border-radius: 0;
  }

  .el-button+.el-button {
    margin-left: 2px;
  }
  .el-card {
      border-radius: 0;
  }
</style>

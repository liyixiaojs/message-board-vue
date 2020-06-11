<template>
  <div class="message-board">
    <div class="list">
      <template v-for="(item, index) in list">
        <div v-if="item.name !== userName" class="item" :key="index">
          <div class="name">{{item.name}}</div>
          <div class="time">{{item.time | compFilter('yyyy-MM-dd hh:mm:ss')}}</div>
          <div class="content">
            <div class="message" v-html="item.content">
              <!-- {{item.content}} -->
            </div>
          </div>
        </div>
        <div v-else-if="item.name === userName" class="item owner" :key="index">
          <div class="time">{{item.time | compFilter('yyyy-MM-dd hh:mm:ss')}}</div>
          <div class="name">{{item.name}}</div>
          <div class="content">
            <div class="message" v-html="item.content">
              <!-- {{item.content}} -->
            </div>
          </div>
        </div>
      </template>
    </div>
    <div class="textarea" ref="textarea" contenteditable="true" @paste="paste($event)">
    </div>
    <div class="buttons">
      <div class="user">
        <select v-model="userName">
           <option>user 1</option>
           <option>user 2</option>
        </select>
      </div>
      <input type="file" style="display: none;" ref="file" accept="image/png, image/jpeg" @change="changeFile()" />
      <button type="button" style="margin-right: 20px;" @click="selectFile()">
        插入图片
      </button>
      <button type="button" @click="submitMessage()">
        提交
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Index',
  filters: {
       compFilter: function(value, format) {
         let date = new Date(value)
        let o = {
            "M+": date.getMonth() + 1,
            "d+": date.getDate(),
            "h+": date.getHours(),
            "m+": date.getMinutes(),
            "s+": date.getSeconds(),
        }
        if(/(y+)/.test(format)){
            format = format.replace(RegExp.$1, (date.getFullYear() + "").substr(4-RegExp.$1.length));
            for(let k in o) {
                if(new RegExp(`(${k})`).test(format)){
                    format = format.replace(RegExp.$1, (RegExp.$1.length == 1)?(o[k]):(("00" + o[k]).substr((""+o[k]).length)))
                }
            }
            return format;
        }
    }
  },
  props: {
  },
  data() {
    return {
      files: [],
      userName: 'user 1',
      list: []
    }
  },
  mounted() {
    this.queryList()
  },
  methods: {
    queryList() {
      this.$http.post('/api/queryList').then(function(res){
        this.list = res.body.data || []
      })
    },
    selectFile() {
      this.$refs.file.click()
    },
    changeFile() {
      let file = this.$refs.file.files[0]
      this.readFileAsDataURL(file).then(dataurl => {
        this.$refs.textarea.innerHTML += `<img src="${dataurl}" index="${this.files.length}" />`
        this.files.push(file)
      })
      this.$refs.file.value = ""
    },
    uploadFile() {
      return new Promise((resolve) => {
        let param = new FormData()
        this.files.forEach(file => {
          param.append('image', file)
        })
        this.files = []
        this.$http.post('/api/upload', param)
          .then(function(res){
              resolve(res.data)
          })
          .catch(function(error){
              console.log(error)
          })
      })
    },
    submitMessage() {
      this.$refs.textarea.querySelector(`img[index="0"]`)
      this.uploadFile().then(result => {
        if (typeof result.data === 'object') {
          for (let i in result.data) {
            let img = this.$refs.textarea.querySelector(`img[index="${i}"]`)
            if (img) {
              img.src = result.data[i]
              img.removeAttribute('index')
            }
          }
        }
        this.save()
      })
    },
    save() {
      let param = {
        content: this.$refs.textarea.innerHTML,
        name: this.userName
      }
      this.$http.post('/api/save', param).then(() => {
        this.queryList()
        this.$refs.textarea.innerHTML = ""
      })
    },
    paste(event) {
      if(event.clipboardData) {
        event.preventDefault()
        for(var i = 0; i < event.clipboardData.items.length; i++) {
          if (event.clipboardData.items[i].type.indexOf('image') !== -1) {
            let file = event.clipboardData.items[i].getAsFile()
            this.readFileAsDataURL(file).then(dataurl => {
              this.$refs.textarea.innerHTML += `<img src="${dataurl}" index="${this.files.length}" />`
              this.files.push(file)
            })
          } else if (!event.clipboardData.items[i + 1] || event.clipboardData.items[i + 1].type.indexOf('image') === -1) {
            event.clipboardData.items[i].getAsString(function (s){
              this.$refs.textarea.innerHTML += s
            });
          }
        }
        setTimeout(() => {
          var textbox = this.$refs.textarea
          var sel = window.getSelection()
          var range = document.createRange()
          range.selectNodeContents(textbox)
          range.collapse(false)
          sel.removeAllRanges()
          sel.addRange(range)
        }, 10)
      }
    },
    readFileAsDataURL(file) {
      return new Promise((resolve) => {
        var reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = (event) => {
          resolve(event.target.result)
        }
      })
    }
  }
}
</script>

<style scoped lang="stylus">
.message-board
  width 50%
  height 50%
  margin 0 auto
  padding-top 30px
  .list
    max-height 70%
    overflow-y auto
    // background-color #fff
  .textarea
    margin-top 30px
    height 30%
    background-color #fff
    padding 10px
    /deep/ img
      max-width 50%
      max-height 50%
    &:focus
      outline none
  .buttons
    margin-top 20px
    text-align right
    height 30px
    line-height 30px
    .user
      float left
      select
        padding 3px 8px
  .item
    margin-bottom 20px
    &.owner
      text-align right
      .name
        margin-left 10px
        margin-right 0
    .name
      display inline-block
      margin-right 10px
      font-weight bold
    .time
      display inline-block
      color #666
      font-size 12px
    .content
      .message
        display inline-block
        background #fff
        padding 10px
        border-radius 2px
        max-width 80%
        /deep/ img
          max-width 100%

@media only screen and (max-device-width:481px)
  .message-board
    width 90%
</style>
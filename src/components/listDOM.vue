<template>
  <div class="list" id="list" ref="list">
    <div
      class="list-item"
      v-for="(item, index) in arrDom"
      :key="index"
      ref="itemDom"
    >
      <slot name="list" :item="item"></slot>
    </div>
    <slot name="footer"></slot>
    <!-- <Observer @intersect="intersected"/> -->
  </div>
</template>

<script>
import ListScroll from '../common/js/listSroll'
export default {
  name: 'ListDOM',
  data() {
    return {
      itemsArr: null,
      arrDom: [],
      arrDomHeight: [],
      arrDomHeightPrefix: [],
      listScrollIns: null,
      container: null,
      judgeToTop: 0,
    }
  },
  emits: ['request'],
  props: {
    resArr: {
      type: Array,
      default: [],
      required: true,
    },
    visualDomCount: {
      type: Number,
      default: 5,
      required: true,
    },
    domHeight: {
      type: Number,
      default: 150,
    },
    listHeight: {
      type: Number,
    },
    scrollInstance: {
      required: true,
    },
    scrollAnima: {
      type: Boolean,
      default: true,
    },
  },
  inject: ['request'],
  mounted() {
    const scrollDom = this.scrollInstance()
    this.arrDom = this.resArr
    this.container = document.getElementById('list')
    const lis = document.querySelectorAll('#m-listContainer li')
    const renderPage = async (firstIndex) => {
      let end = 0
      let count = 0
      while (
        firstIndex + this.visualDomCount * 5 > this.resArr.length &&
        count < 5
      ) {
        // this.$emit('request')
        await this.request()
        count++
        if (this.judgeToTop === 1) {
          this.judgeToTop = 0
          break
        }
      }
      // if()
      const UpdateDOMList = () => {
        if (firstIndex + this.visualDomCount * 5 > this.resArr.length) {
          end = this.resArr.length
          this.arrDom = this.resArr.slice(firstIndex)
          // console.log(this.resArr, 'firstIndex', firstIndex)
        } else {
          end = firstIndex + this.visualDomCount * 5
          this.arrDom = this.resArr.slice(
            firstIndex,
            firstIndex + this.visualDomCount * 5
          )
        }
        this.$nextTick(() => {
          for (let i = firstIndex; i < end; i++) {
            this.arrDomHeight[i] =
              this.$refs.itemDom[i - firstIndex].clientHeight
          }
        })
      }
      await UpdateDOMList()
    }
    // 定义一个箭头函数，然后
    // 现在的问题是我不知道什么时候更新，哎呀，这个函数跟渲染逻辑之间割裂了跟难受呀
    const getDomHeight = () => {
      return this.arrDomHeight
    }
    this.$nextTick(() => {
      const Node = document.querySelectorAll('.list-item')
      Node[0].classList.add('_first')
      Node[Node.length - 1].classList.add('_last')
      renderPage(0)

      const renderFunction = async (firstIndex) => {
        await renderPage(firstIndex)
      }
      const getDomHeightFunction = () => {
        return getDomHeight()
      }

      this.listScrollIns = new ListScroll({
        container: this.container,
        listSize: this.visualDomCount * 5,
        itemHeight: this.domHeight,
        // resArr: this.itemsArr || [],
        renderFunction,
        // 动态的获取DOM的高度
        getDomHeightFunction,
        scrollDom: scrollDom,
        scrollAnima: this.scrollAnima,
        // _setScoll: this._setScoll,
      })

      this.listScrollIns.startObserver()
    })
    this.MonitorScroll()
  },
  methods: {
    scrollToTop: function () {
      let scrollList = this.scrollInstance()
      // scrollList.scrollTop = 0
      this.listScrollIns.scrollToTop()
      // this.resArr = this.resArr.slice(0, 20)
      this.judgeToTop = 1
      // console.log(this.resArr, 'this.resArr')
    },

    getScroll: function () {
      return this.$refs.list.scrollTop
    },
    // _setScoll: (v) => {
    //   this.$refs.list.scrollToTop = v
    // },
    getSize: function () {
      return this.resArr.length
    },
    scrollToIndex: function (index) {
      this.listScrollIns.scrollToIndex(index)
    },
    MonitorScroll: function () {
      setInterval(() => {
        let paddingTop = parseInt(this.container.style.paddingTop.slice(0, -2))
        const scrollDom = this.scrollInstance()
        let scrollVTop = scrollDom.scrollTop
        // console.log(this.$refs.list.clientHeight, 'clientHeight')
        // console.log(scrollVTop, paddingTop, '---')
        if (paddingTop > scrollVTop + 200) {
          let [index, paddingV] = this.getIndex(scrollVTop)
          this.scrollToIndex(index)
        } else {
          const CHeight = this.$refs.list.clientHeight
          const PBottom = parseInt(
            this.container.style.paddingBottom.slice(0, -2)
          )
          if (scrollVTop > CHeight - PBottom + 100) {
            let [index, paddingV] = this.getIndex(scrollVTop)
            this.scrollToIndex(index)
          }
        }
      }, 100)
    },
    getIndex: function (topIns) {
      let count = 0
      let DomIndex = 0
      let DomsHeight = 0
      for (let i = 0; i < this.arrDomHeight.length; i++) {
        count += this.arrDomHeight[i]
        if (count <= topIns) {
          DomIndex = i
          DomsHeight = count
        } else {
          return [DomIndex, DomsHeight]
        }
      }
      return [DomIndex, DomsHeight]
    },
  },
}
</script>

<style lang="scss" scoped>
.list {
  display: flex;
  flex-direction: column;
  // overflow-y: auto;
}
</style>

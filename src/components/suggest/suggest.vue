<template>
  <scroll
    ref="suggest"
    class="suggest"
    :data="result"
    :pullup="pullup"
    @scrollToEnd="searchMore"
  >
    <ul class="suggest-list">
      <li class="suggest-item" v-for="(item, index) in result" :key="index">
        <div class="icon">
          <i :class="getIconCls(item)"></i>
        </div>
        <div class="name">
          <p class="text" v-html="getDisplayName(item)"></p>
        </div>
      </li>
      <loading v-show="hasMore" title=""></loading>
    </ul>
  </scroll>
</template>

<script>
import Scroll from 'base/scroll/scroll'
import Loading from 'base/loading/loading'
import { search } from 'api/search'
import { ERR_OK } from 'api/config'
import { createSong, isValidMusic } from 'common/js/song'

const TYPE_SINGER = 'singer'
const perpage = 20

export default {
  props: {
    query: {
      type: String,
      default: ''
    },
    showSinger: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      page: 1,
      result: [],
      pullup: true,
      hasMore: true
    }
  },
  methods: {
    getIconCls (item) {
      if (item.type === TYPE_SINGER) {
        return 'icon-mine'
      } else {
        return 'icon-music'
      }
    },
    getDisplayName (item) {
      if (item.type === TYPE_SINGER) {
        return item.singername
      } else {
        return `${item.name}-${item.singer}`
      }
    },
    search () {
      this.page = 1
      this.hasMore = true
      this.$refs.suggest.scrollTo(0, 0)
      search(this.query, this.page, this.showSinger, perpage).then((res) => {
        if (res === ERR_OK) {
          this.result = this._getResult(res.data)
          this._checkMore(res.data)
        }
      })
    },
    searchMore () {
      if (!this.hasMore) {
        return
      }
      this.page++
      search(this.query, this.page, this.showSinger, perpage).then((res) => {
        if (res === ERR_OK) {
          this.result = this.result.concat(this._getResult(res.data))
          this._checkMore(res.data)
        }
      })
    },
    _getResult (data) {
      let ret = []
      if (data.zhida && data.zhida.singerid) {
        ret.push({ ...data.zhida, ...{ type: TYPE_SINGER } })
      }
      if (data.song) {
        ret = ret.concat(this._normalizeSong(data.song.list))
      }
      return ret
    },
    _normalizeSong (list) {
      const ret = []
      list.forEach((musicData) => {
        if (isValidMusic(musicData)) {
          ret.push(createSong(musicData))
        }
      })
      return ret
    },
    _checkMore (data) {
      const song = data.song
      if (!song.list.length || (song.curnum + (song.curpage - 1) * perpage) >= song.totalnum) {
        this.hasMore = false
      }
    }
  },
  watch: {
    query () {
      this.search()
    }
  },
  components: {
    Scroll,
    Loading
  }
}
</script>

<style lang='stylus' scoped>
  @import '~common/stylus/variable'
  @import '~common/stylus/mixin'

  .suggest
    height 100%
    overflow hidden
    .suggest-list
      padding 0 30px
      .suggest-item
        display flex
        align-items center
        padding-bottom 20px
      .icon
        flex 0 0 30px
        width 30px
        [class^='icon-']
          font-size 14px
          color $color-text-d
      .name
        flex 1
        font-size $font-size-medium
        color $color-text-d
        overflow hidden
        .text
          no-wrap()
    .no-result-wrapper
      position absolute
      width 100%
      top 50%
      transform translateY(-50%)
</style>

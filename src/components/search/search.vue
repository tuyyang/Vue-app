<template>
  <div class="search" ref="search">
    <div class="search-box-wrapper">
      <search-box ref="searchBox" @query="onQueryChange"></search-box>
    </div>
    <div class="shortcut-wrapper" v-show="!query">
      <div class="shortcut">
        <div class="hot-key">
          <h1 class="title">热门搜索</h1>
          <ul>
            <li
              class="item"
              v-for="(item, index) in hotKey"
              :key="index"
              @click="addQuery(item.k)"
            >
              <span>{{ item.k }}</span>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <div class="search-result" v-show="query">
      <suggest :query="query"></suggest>
    </div>
  </div>
</template>

<script>
import SearchBox from 'base/search-box/search-box'
import Suggest from 'components/suggest/suggest'
import { getHotKey } from 'api/search'
import { ERR_OK } from 'api/config'

export default {
  data () {
    return {
      hotKey: [],
      query: ''
    }
  },
  created () {
    this._getHotKey()
  },
  methods: {
    onQueryChange (query) {
      this.query = query
    },
    addQuery (query) {
      this.$refs.searchBox.setQuery(query)
    },
    _getHotKey () {
      getHotKey().then((res) => {
        if (res.code === ERR_OK) {
          this.hotKey = res.data.hotkey.slice(0, 10)
        }
      })
    }
  },
  components: {
    SearchBox,
    Suggest
  }
}
</script>

<style lang='stylus' scoped>
  @import '~common/stylus/variable'
  @import '~common/stylus/mixin'

  .search
    .search-box-wrapper
      margin 20px
    .shortcut-wrapper
      position fixed
      top 178px
      bottom 0
      width 100%
      .shortcut
        height 100%
        overflow hidden
        .hot-key
          margin 0 20px 20px 20px
          .title
            margin-bottom 20px
            font-size $font-size-medium
            color $color-text-l
          .item
            display inline-block
            padding 5px 10px
            margin 0 20px 10px 0
            border-radius 6px
            background $color-highlight-background
            font-size $font-size-medium
            color $color-text-d
    .search-result
      position fixed
      width 100%
      top 178px
      bottom 0
</style>

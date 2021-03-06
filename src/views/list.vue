<template>
  <div class="palette-list">
    <div class="palette-list-top clearfix">
      <palette-searcher
        :placeholder="$t('list.searchTip')"
        @search="onSearchComponents"
      ></palette-searcher>
      <el-button type="primary" round icon="el-icon-circle-check" @click="goToGenerate">
        {{ $t('list.generateBtn') }}
      </el-button>
    </div>
    <template v-if="searchComponents.length">
      <div class="palette-list-category clearfix">
        <ul>
          <li
            v-for="(component, index) in searchComponents"
            :key="`item-${index}`"
            class="palette-list-category-item"
          >
            <palette-card
              :name="component.itemName"
              @click="onClick(component.moduleName, component.itemName)"
            ></palette-card>
          </li>
        </ul>
      </div>
    </template>
    <template v-else-if="theme">
      <div
        v-for="(module, moduleName) in theme.data"
        :key="`module-${moduleName}`"
        class="palette-list-category clearfix"
      >
        <h3 class="palette-list-category-title">
          <template v-if="moduleName === 'basic'">
            <p class="text">{{moduleName}}</p>
            <p class="tip">
              <i class="el-icon-info"></i>
              {{ $t('list.basicTip') }}
            </p>
          </template>
          <template v-else>
            <p class="text">{{moduleName}}</p>
            <p class="tip">
              <i class="el-icon-info"></i>
              {{ $t('list.componentsTip') }}
            </p>
          </template>
        </h3>
        <ul>
          <li
            v-for="(item, itemName) in module"
            :key="`item-${itemName}`"
            class="palette-list-category-item"
          >
            <palette-card
              :name="itemName"
              @click="onClick(moduleName, itemName)"
            ></palette-card>
          </li>
        </ul>
      </div>
    </template>
    <template v-else>
      <palette-status
        imgUrl="//manhattan.didistatic.com/static/manhattan/mfd/result-page/network"
        :describe="$t('list.errorTip')"
        :button="{
          text: $t('list.errorBtn'),
          handler () {
            $router.replace('/record')
          }
        }"
      ></palette-status>
    </template>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import PaletteSearcher from '../components/searcher'
import PaletteCard from '../components/card'
import PaletteEditor from '../components/editor'
import PaletteStatus from '../components/status'
import { traverseObject } from '../utils'

export default {
  components: {
    PaletteSearcher,
    PaletteCard,
    PaletteEditor,
    PaletteStatus
  },
  data () {
    return {
      themeIndex: -1,
      searchComponents: []
    }
  },
  computed: {
    ...mapState(['themes']),
    theme () {
      const themeIndex = this.themeIndex
      if (themeIndex < 0) {
        return null
      } else {
        return this.themes[themeIndex]
      }
    }
  },
  created () {
    this.themeIndex = this.$route.query.themeIndex
  },
  methods: {
    onClick (moduleName, itemName) {
      this.$router.push({
        path: '/edit',
        query: {
          themeIndex: this.themeIndex,
          moduleName,
          itemName
        }
      })
    },
    onSearchComponents (keyword) {
      if (keyword && this.theme) {
        const dataSource = this.theme.data
        const components = []
        traverseObject(dataSource, (key, value, path) => {
          const moduleName = path[0]
          const itemName = path[1]
          if (~itemName.toLocaleLowerCase().indexOf(keyword.toLocaleLowerCase())) {
            components.push({
              moduleName,
              itemName
            })
          }
        }, false)
        this.searchComponents = components
      } else {
        this.searchComponents = []
      }
    },
    goToGenerate () {
      this.$router.push({
        path: '/generate',
        query: {
          themeIndex: this.themeIndex
        }
      })
    }
  }
}
</script>

<style lang="stylus">
.palette-list
  padding-top 30px
  .palette-list-top
    .palette-searcher
      float left
    .el-button
      float right
  .palette-list-category
    margin-top 30px
    ul
      float left
      width 100%
    .palette-list-category-title
      position relative
      float left
      width 100%
      margin-bottom 20px
      // padding-left 35px
      color #333
      font-size 28px
      font-weight 400
      span.type
        position absolute
        top 0
        left 0
        font-size 12px
        color #999
      p.text
        float left
      p.tip
        float left
        margin-top 12px
        margin-left 20px
        font-size 12px
        color #999
    .palette-list-category-item
      position relative
      float left
      width 18%
      margin-bottom 15px
      margin-right 2.5%
      padding-bottom 18%
      box-sizing border-box
      &:nth-child(5n)
        margin-right 0
      // overflow hidden
      .palette-card
        position absolute
        left 0
        top 0
      .palette-card-img
        transition all .3s
      &:hover
        .palette-card-img
          transform translateY(-50px) scale(1.5)
  .palette-list-dialog
    // top 50%
    // height 800px
    // transform translateY(-50%)
    .el-dialog__header
      display none
    .el-dialog__body
      padding 0
  .palette-status
    margin-top 200px
</style>

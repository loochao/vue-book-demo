<template>
  <div id="app" class="daily">
    <div class="daily-menu-item">
      <div class="daily-menu-item"
           :class="{ on: type === 'recommend'}">每日推荐</div>
      <div class="daily-menu-item"
           :class="{ on: type === 'daily'}"
           @click="showThemes = !showThemes">主题日报</div>
      <ul v-show="showThemes">
        <li v-for="item in themes">
          <a :class="{ on: item.id === themeId && type === 'daily'}"
             @click="handleToTheme(item.id)">{{ item.name }}</a>
        </li>
      </ul>
    </div>
    <div class="daily-list" ref="list">
      <template v-if="type === 'recommend'">
        <div v-for="list in recommendList">
          <div class="daily-date">
            {{ formatDay(list.date) }}
          </div>
          <Item
            v-for="item in list.stories"
            :data="item"
            :key="item.id"
            @click.native="handleClick(item.id)">
          </Item>
        </div>
      </template>
      <template v-if="type === 'daily'">
        <Item
          v-for="item in list"
          :data="item"
          :key="item.id"
          @click.native="handleClick(item.id)">
        </Item>
      </template>
    </div>
    <daily-article :id="articleId"></daily-article>
  </div>
</template>

<script>
import $ from './libs/util'
import Item from './components/item.vue'
import dailyArticle from './components/daily-article.vue'
export default {
  components: {Item, dailyArticle},
  name: 'App',
  data () {
    return {
      themes: [],
      recommendList: [],
      dailyTime: $.getTodayTime(),
      isLoading: false,
      showThemes: false,
      type: 'recommend',
      list: [],
      themeId: 0,
      articleId: 0
    }
  },
  methods: {
    getThemes () {
      //axios send get
      $.ajax.get('themes').then(res => {
        this.themes = res.others
      })
    },
    handleToTheme (id) {
      //change menu type
      this.type = 'daily'
      this.themeId = id
      this.list = []
      $.ajax.get('theme/' + id).then(res => {
        this.list = res.stories.filter(item => item.type !== 1)
      })
    },
    handleToRecommend () {
      this.type = 'recommend'
      this.recommendList = []
      this.dailyTime = $.getTodayTime()
      this.getRecommendList()
    },
    getRecommendList () {
      this.isLoading = true
      const prevDay = $.prevDay(this.dailyTime + 86400000)
      $.ajax.get('news/before/' + prevDay).then(res => {
        this.recommendList.push(res)
        this.isLoading = false
      })
    },
    formatDay (date) {
      let month = date.substr(4, 2)
      let day = date.substr(6, 2)
      if (month.substr(0, 1) === '0') month = month.substr(1, 1)
      if (day.substr(0, 1) === '0') day = day.substr(1, 1)
      return `${month} 月 ${day} 日`
    },
    handleClick (id) {
      this.articleId = id
    }
  },
  mounted () {
    //initialize
    this.getThemes()
    this.getRecommendList()
    const $list = this.$refs.list
    $list.addEventListener('scroll', () => {
      if (this.type === 'daily' || this.isLoading) return ;
      if (
        $list.scrollTop + document.body.clientHeight >= $list.scrollHeight
      ) {
        this.dailyTime -= 86400000
        this.getRecommendList()
      }
    })
  },
  beforeDestroy () {
    this.removeEventListener('scroll')
  }
}
</script>


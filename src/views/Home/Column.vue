<template>
  <div class="home-lay index-container"
       id="index">
    <div class="row">
      <div class="col-xs-12 col-sm-8--4 col-md-8--4">
        <!--home-lay layout-content start-->
        <section class="client-card">
          <NavColumn :navItem="articleColumn.homeColumn" />
          <NavSort @navTap="navTap"
                   ref="navSort"></NavSort>
          <scroll-loading @scroll-loading="infiniteHandler"
                          :isLoading="isLoading"
                          :isMore="isMore">
            <ArticleItem v-for="(item,key) in home.article.article_list"
                         :key="key"
                         :articleItem="item" />

          </scroll-loading>
        </section>
        <!--home-lay layout-content end-->
      </div>
      <div class="col-xs-12 col-sm-3--6 col-md-3--6">
        <!--aside.html start-->
        <div class="home-aside">
          <ColumnAside />
        </div>
        <!--aside.html end-->
      </div>
    </div>
  </div>
</template>

<script>
import ColumnAside from "@views/Home/ColumnAside";
import NavHeader from "@views/Home/NavHeader";
import NavColumn from "@views/Home/NavColumn";
import NavSort from "@views/Home/NavSort";
import ArticleItem from "@views/Article/component/ArticleItem";
import { mapState } from "vuex";
import { ScrollLoading } from "@components";
import { baidu, google } from '@utils'
import googleMixin from '@mixins/google'

export default {
  name: "column",
  mixins: [googleMixin], //混合谷歌分析
  metaInfo () {
    return {
      title: this.currentColumn().name,
      titleTemplate: `%s - ${this.website.meta.website_name}`,
      meta: [
        {
          // set meta 
          name: "description",
          content: `${this.currentColumn().name} - ${this.currentColumn().description}`
        }
      ],
      htmlAttrs: {
        lang: "zh"
      },
      script: [
        ...baidu.resource({
          route: this.$route,
          config: this.website.config,
          random: this.$route.params.en_name
        }),
        ...google.statisticsCode({
          route: this.$route, googleCode: this.website.config.googleCode, random: ''
        })
      ],
      __dangerouslyDisableSanitizers: ['script']
    };
  },
  async asyncData ({ store, route, accessToken = "" }) {
    // 触发 action 后，会返回 Promise
    return Promise.all([
      store.commit(
        "articleColumn/SET_CURRENT_ARTICLE_COLUMN",
        route.params.en_name || ""
      ),
      store.commit("home/SET_INIT_INDEX_ARTICLE_LIST"), // 重置文章列表数据
      store.dispatch("articleColumn/GET_ARTICLE_COLUMN_ALL"),
      store.dispatch("articleColumn/GET_ARTICLE_COLUMN", {
        en_name: route.params.en_name || ""
      }),
      store.dispatch("home/GET_INDEX_COLUMN_ARTICLE_LIST", {
        columnEnName: route.params.en_name || "",
        type: route.query.type || '',
      })
    ]);
  },
  data () {
    return {
      page: 2,
      pageSize: 25,
      sort: "",
      isLoading: false,
      isMore: true
    };
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // this.initHomeDate()
    this.$refs.navSort.dafauleNav();
    this.isMore = true;
    next();
  },
  created () {
    this.$store.dispatch("home/GET_POPULAR_ARTICLE_TAG"); // 获取热门文章标签
  },
  methods: {
    navTap (val) {
      this.sort = val;
      this.initHomeDate();
    },
    initHomeDate () {
      this.$store.commit("home/SET_INIT_INDEX_ARTICLE_LIST"); // 重置文章列表数据
      this.isMore = true;
      this.page = 1;
      this.infiniteHandler();
    },
    currentColumn () {
      let _currentColumn = {}
      this.articleColumn.homeColumn.map(item => {
        if (item.en_name === this.$route.params.en_name) {
          _currentColumn = item
        }
      })
      return _currentColumn
    },
    infiniteHandler () {
      this.isLoading = true;
      this.$store
        .dispatch("home/GET_INDEX_COLUMN_ARTICLE_LIST", {
          columnEnName: this.$route.params.en_name,
          type: this.$route.query.type || '',
          pageSize: this.pageSize,
          sort: this.sort,
          page: this.page
        })
        .then(result => {
          this.isLoading = false;
          if (result.data.article_list.length === this.pageSize) {
            this.page += 1;
          } else {
            this.isMore = false;
          }
        })
        .catch(err => {
          this.isMore = false;
        });
    }
  },
  computed: {
    ...mapState(["home", "articleColumn", "website"]) // home:主页  articleColumn:文章的专栏
  },
  components: {
    ColumnAside,
    NavHeader,
    NavColumn,
    NavSort,
    ArticleItem,
    ScrollLoading
  }
};
</script>

<style scoped lang="scss">
.home-lay {
  .client-card {
    position: relative;
    border-radius: 2px;
    padding: 0 15px;
    /deep/ .article {
      border-bottom: 1px solid #f7f7f7;
    }
  }
}
</style>


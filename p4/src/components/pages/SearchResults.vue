<template>
  <div class="subCompoent">
    <p v-if="(newChannelCount > 0)">You have {{ newChannelCount }} newly saved channel(s).</p>
    <h2 v-if="searchingKeywords">Results for "{{ searchingKeywords }}"</h2>
    <p v-if="recentSearches">
      <strong>Recent Searches: </strong>
      <span
        v-for="searched in recentSearches.slice(0, 5)"
        :key="searched"
      >
        <router-link class="searchLink"
          :to='{ name: "search", params: {keywords: searched}}'> {{searched}} </router-link> | </span>
    </p>
    <news-list :articles="articles"></news-list>
  </div>
</template>

<script>
import * as app from "./../../app.js";
import NewsList from "../NewsList.vue";

export default {
  name: "SearchResults",
  components: { NewsList },
  props: ["keywords"],
  data: function() {
    return {
      searchingKeywords: "",
      articles: [],
      recentSearches: null
    };
  },
  methods: {
    searchByKeywords: function() {
      let keywords_encoded = this.keywords;
      this.searchingKeywords = decodeURI(keywords_encoded);
      this.saveSearchHistory();

      // Search on NewsAPI
      app.axios
        .get(app.config.searchNewsApi + keywords_encoded)
        .then(response => {
          this.articles = response.data.articles;
        });
    },
    saveSearchHistory: function() {
      let recentSearch = [];
      if (typeof localStorage !== 'undefined'){
        recentSearch = JSON.parse(
          localStorage.getItem("recentSearch") || "[]"
        );
      }
      recentSearch.push(this.searchingKeywords);
      this.recentSearches = [...new Set(recentSearch)];
      if (typeof localStorage !== 'undefined'){
        localStorage.setItem("recentSearch", JSON.stringify(this.recentSearches));
      }
    }
  },
  mounted() {
    this.searchByKeywords();
  },
  computed: {
    newChannelCount: function() {
        return this.$store.state.newChannelCount;
    }
  },
  watch: {
    keywords: function() {
      this.searchByKeywords();
    }
  }
};
</script>


<style scoped>
.searchLink {
  color: #230885;
  text-decoration: underline;
  margin-left: 0px;
  font-size: 18px;
}

.router-link-active {
    background-color: #ffffff;
}
</style>
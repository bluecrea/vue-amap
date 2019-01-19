<template>
  <div class="el-vue-search-box-container" @keydown.up="selectTip('up')" @keydown.down="selectTip('down')">
    <div class="search-box-wrapper">
      <input type="text" v-model="keyword" @keyup.enter="search" @input="autoComplete">
    </div>
    <div class="search-tips">
      <ul>
        <li v-for="(tip, index) in tips" :key="index" @click="changeTip(tip)" @mouseover="selectedTip=index" :class="{'autocomplete-selected': index === selectedTip}">
          <h4>{{tip.name}}</h4>
          <p>{{tip.address}}</p>
        </li>
      </ul>
    </div>
  </div>
</template>
<style lang="less">
  .el-vue-search-box-container {
    position: relative;
    width: 100%;
    height: 1rem;
    z-index: 10;
    .search-box-wrapper {
      position: absolute;
      display: flex;
      align-items: center;
      left: 0;
      top: 0;
      width: 100%;
      height: 100%;
      box-sizing: border-box;
      input {
        flex: 1;
        height: 20px;
        line-height: 20px;
        letter-spacing: .5px;
        font-size: 14px;
        text-indent: 10px;
        box-sizing: border-box;
        border: none;
        outline: none;
      }
    }
    .search-tips {
      position: absolute;
      top: 2rem;
      overflow: auto;
      left: -2.5rem;
      right: -2.5rem;
      ul {
        padding: 0;
        margin: 0;
        li {
          height: 40px;
          line-height: 40px;
          border-bottom:1px solid #dbdbdb;
          padding: 0 10px;
          cursor: pointer;
          h4 {
            font-weight: 600;
            line-height:1rem;
          }
          p {
            color: #999;
            font-size: .55rem;
            padding-right: 1.3rem;
            line-height: .5rem;
          }
          &.autocomplete-selected {
            background: #eee;
          }
        }
      }
    }
  }
</style>
<script>
import RegisterComponentMixin from '../mixins/register-component';
import {lazyAMapApiLoaderInstance} from '../services/injected-amap-api-instance';
export default {
  name: 'el-amap-search-box',
  mixins: [RegisterComponentMixin],
  props: ['searchOption', 'onSearchResult', 'events', 'default'],
  data() {
    return {
      keyword: this.default || '',
      tips: [],
      selectedTip: -1,
      loaded: false
    };
  },
  mounted() {
    let _loadApiPromise = lazyAMapApiLoaderInstance.load();
    _loadApiPromise.then(() => {
      this.loaded = true;
      this._onSearchResult = this.onSearchResult;
      // register init event
      this.events && this.events.init && this.events.init({
        autoComplete: this._autoComplete,
        placeSearch: this._placeSearch
      });
    });
  },
  computed: {
    _autoComplete() {
      if (!this.loaded) return;
      return new AMap.Autocomplete(this.searchOption || {});
    },
    _placeSearch() {
      if (!this.loaded) return;
      return new AMap.PlaceSearch(this.searchOption || {});
    }
  },
  methods: {
    autoComplete() {
      if (!this.keyword || !this._autoComplete) return;
      this._autoComplete.search(this.keyword, (status, result) => {
        if (status === 'complete') {
          this.tips = result.tips;
        }
      });
    },
    search() {
      this.tips = [];
      if (!this._placeSearch) return;
      this._placeSearch.search(this.keyword, (status, result) => {
        if (result && result.poiList && result.poiList.count) {
          let {poiList: {pois}} = result;
          let LngLats = pois.map(poi => {
            poi.lat = poi.location.lat;
            poi.lng = poi.location.lng;
            poi.name = poi.name;
            poi.address = poi.address;
            return poi;
          });
          this._onSearchResult(LngLats);
        } else if (result.poiList === undefined) {
          throw new Error(result);
        }
      });
    },
    changeTip(tip) {
      this.keyword = tip.name;
      this.search();
    },
    selectTip(type) {
      if (type === 'up' && this.selectedTip > 0) {
        this.selectedTip -= 1;
        this.keyword = this.tips[this.selectedTip].name;
      } else if (type === 'down' && this.selectedTip + 1 < this.tips.length) {
        this.selectedTip += 1;
        this.keyword = this.tips[this.selectedTip].name;
      }
    }
  }
};
</script>

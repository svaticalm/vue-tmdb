<template lang="html">
  <div class="fast-search">
    <input type="text" v-model="keywords" name="" placeholder="Что хотите посмотреть?" @click="resopened  = true;" id="fast-search">
    <div class="search-butt">
      <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" viewBox="0 0 512 512" enable-background="new 0 0 512 512" width="512px" height="512px" class=""><g><g>
          <path d="M495,466.2L377.2,348.4c29.2-35.6,46.8-81.2,46.8-130.9C424,103.5,331.5,11,217.5,11C103.4,11,11,103.5,11,217.5   S103.4,424,217.5,424c49.7,0,95.2-17.5,130.8-46.7L466.1,495c8,8,20.9,8,28.9,0C503,487.1,503,474.1,495,466.2z M217.5,382.9   C126.2,382.9,52,308.7,52,217.5S126.2,52,217.5,52C308.7,52,383,126.3,383,217.5S308.7,382.9,217.5,382.9z" data-original="#000000" class="active-path" data-old_color="#000000" fill="#FFFFFF"/>
        </g></g> </svg>
    </div>
    <ul class="fast-search-list" v-if="resopened">
      <li v-for="res in result" :key="res.id" @click="$emit('getfilm',[res.media_type,res.id]);resopened = false;" v-show="res.media_type != 'person'" class="fs-item">
          <span v-show="res.title">{{res.title}}</span>
          <span v-show="res.name">{{res.name}}</span>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'FastSearch',
  data: function(){
    return {
      result: [],
      keywords: '',
      errors: [],
      resopened: false,
    }
  },
  watch: {
    keywords(after){
      this.search(after);
    }
  },
  mounted(){
    let self = this;
    document.body.onclick = function (e) {
        e = e || event;
        let target = e.target || e.srcElement;
        if (target.id == "fast-search") {
          return;
        } else {
            self.resopened = false;
        }
    }
  },
  methods: {
    flClose: function(){
      this.resopened = false;
    },
    search: function(keywords){
      this.resopened = true;
      //Запрашиваем массив с фильмами, серилами и персонами. Сортируем элементы по убыванию популярности (функция sort) + reverse
      axios.get('https://api.themoviedb.org/3/search/multi?api_key=7e00b848ffbb0a2bb957f6631e1ad255&language=ru-RU&include_adult=false&query=' + keywords)
       .then(response => {this.result = response.data == 'error' ? [] : response.data.results.sort((a, b) => b.popularity - a.popularity );})
       .catch(error => {
             this.errors.push(error);
       });
    },
  },
}
</script>

<style lang="css" scoped>
  .search-butt{
    position: absolute;
    top: 17px;
    right: 20px;
  }
  .search-butt svg{
    width: 25px;
    height: 25px;
    cursor: pointer;
  }
  .fast-search{
    height: 60px;
    width: 100%;
    position: relative;
  }
  .fast-search input::-webkit-input-placeholder{
    color: #c8c8c8;
  }
  .fast-search input{
    width: 100%;
    height: 60px;
    background-color: #050a1f;
    padding-left: 20px;
    box-sizing: border-box;
    font-size: 18px;
    font-weight: 900;
    border-radius: 5px;
    color: #fff;
    outline: none;
    border: none;
    border-bottom-left-radius: 0;
    border-bottom-right-radius: 0;
    border-bottom: 2px solid #fff;
    box-shadow: inset 0 0 15px 2px rgba(0,0,0,0.0), 0 0 10px 1px rgba(255,255,255,0.0);
    transition: all .2s;
  }
  .fast-search input:focus{
    background-color: #fff;
    color: #292d3e;
  }
  .fast-search input:focus + .search-butt svg path{
    fill: #050a1f;
  }
  .fast-search-list{
    padding-left: 0;
    position: absolute;
    z-index: 10;
    background-color: #fff;
    top: 40px;
    width: 100%;
    color: #000;
    box-shadow: 0 0 10px 3px rgba(0,0,0,0.1);
    border-bottom-left-radius: 5px;
    border-bottom-right-radius: 5px;
    max-height: 290px;
    overflow: hidden;
    overflow-y: auto;
  }
  .fast-search-list::-webkit-scrollbar { width: 0; }
  .fast-search-list { -ms-overflow-style: none; }
  .fast-search-list li{
    list-style: none;
    height: 40px;
    color: #1c2134;
    display: flex;
    align-items: center;
    padding-left: 30px;
    border-bottom: 1px solid #d4d4d4;
    cursor: pointer;
    font-size: 14px;
    font-weight: 900;
    transition: all .15s;
  }
  .fast-search-list li:hover{
    height: 50px;
    background-color: #ff5959;
    color: #fff;
  }
  .fast-search-list li:last-child{
    border-bottom-left-radius: 5px;
    border-bottom-right-radius: 5px;
    border-bottom: 0;
  }
</style>

<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <h1>Фильмы The Movie Database</h1>
    <ul class="cats">
      <li :class="{'cat-item': true,'cn-active': index == currentIndex}" v-for="(cat,index) in cats" :key="cat.id"  @click="fetchData(cat.catname);currentIndex = index">{{cat.name}}</li>
    </ul>
    <ul class="films-list">
      <li v-for="(result,index) in results" :key="result.id" class="list-item">
        <img :src="'https://image.tmdb.org/t/p/w500/' + result.poster_path" alt="12">
        <div class="info">
          <p :id="'id'+index" class="title"><b>{{result.title}}</b> <br> {{result.original_title}}</p>
          <p class="desc">{{result.overview.trimtxt(200)}}</p>
          <div class="butt-detail">
            Подробнее
          </div>
        </div>
      </li>
    </ul>
  </div>
</template>
<script>
String.prototype.trimtxt = function (length) {
  return this.length > length ? this.substring(0, length) + "..." : this;
}
import axios from 'axios'
export default {
  name: 'app',
  data: function(){
    return {
      results: [],
      errors: [],
      currentIndex: '',
      cats: [{id: 1,name: 'Популярные',catname:'popular'},{id: 2,name:'Лучшие',catname:'top_rated'},{id: 3,name:'Ожидаемые',catname: 'upcoming'},{id: 4, name: 'Смотрят сейчас',catname: 'now_playing'}],
    }
  },
  mounted(){
    axios.get('https://api.themoviedb.org/3/movie/popular?api_key=7e00b848ffbb0a2bb957f6631e1ad255&language=ru-RU')
     .then(response => {this.results = response.data == 'error' ? [] : response.data.results;})
     .catch(error => {
           this.errors.push(error);
     });
  },
  methods: {
    fetchData: function(catname){
      let cat = catname;
      axios.get('https://api.themoviedb.org/3/movie/' + cat + '?api_key=7e00b848ffbb0a2bb957f6631e1ad255&language=ru-RU')
       .then(response => {this.results = response.data == 'error' ? [] : response.data.results;})
       .catch(error => {
             this.errors.push(error);
       });
    },
  },
}
</script>

<style>
.cn-active{
  color: #000 !important;
}
.cats{
  padding: 0;
  width: 500px;
  margin: 0 auto;
  display: flex;
  justify-content: space-between;
  list-style: none;
  margin-bottom: 40px;
}
.cat-item{
  cursor: pointer;
  color: #878787;
  transition: all .2s;
}
.cat-item:hover{
  color: #000;
}
h1{
  margin-bottom: 30px;
}
.info{
  position: relative;
}
.butt-detail{
  color: #ff2e2e;
  border: 1px solid #ff2e2e;
  width: 170px;
  height: 30px;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: center;
  position: absolute;
  bottom: 15px;
  font-size: 14px;
  transition: all .1s
}
.butt-detail:hover{
  background-color: #ff2e2e;
  color: #fff;
}
.films-list{
  width: 1020px;
  margin: 0 auto;
  padding: 0;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  flex-wrap: wrap;
}
.list-item{
  width: 480px;
  display: flex;
  text-align: left;
  padding-right: 20px;
  border-radius: 5px;
  overflow: hidden;
  cursor: pointer;
  margin-bottom: 20px;
  transition: all .2s;
}
.list-item:hover{
  box-shadow: 0 0 20px 3px rgba(0,0,0,0.25);
}
.list-item img{
  width: 180px;
  margin-right: 20px;
}
.list-item .title{
  color: #6e6e6e;
  font-size: 12px;
  line-height: 22px;
}
.list-item .title b{
  color: #000;
  font-size: 16px;
}
.list-item .desc{
  font-size: 14px;
  color: #6e6e6e;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 60px;
}
</style>

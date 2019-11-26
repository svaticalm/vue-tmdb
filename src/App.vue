
<template>
  <div id="app">
    <!-- Модальное окно детальной информации о фильме (вывод ифнформации при клике на кнопку "подробнее")  -->
    <div class="film-detail-modal" id="film-detail-modal" :class="{'modal-open': modalBool}" >
      <div class="back-black" @click="modalBool =false;videosrc = '';rScroll();"></div>
      <div class="modal-inner">
        <div class="modal-close" @click="modalBool = false; videosrc = '';rScroll();">
          <svg version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
          viewBox="0 0 47.971 47.971" style="enable-background:new 0 0 47.971 47.971;" xml:space="preserve">
            <g>
              <path d="M28.228,23.986L47.092,5.122c1.172-1.171,1.172-3.071,0-4.242c-1.172-1.172-3.07-1.172-4.242,0L23.986,19.744L5.121,0.88
              c-1.172-1.172-3.07-1.172-4.242,0c-1.172,1.171-1.172,3.071,0,4.242l18.865,18.864L0.879,42.85c-1.172,1.171-1.172,3.071,0,4.242
              C1.465,47.677,2.233,47.97,3,47.97s1.535-0.293,2.121-0.879l18.865-18.864L42.85,47.091c0.586,0.586,1.354,0.879,2.121,0.879
              s1.535-0.293,2.121-0.879c1.172-1.171,1.172-3.071,0-4.242L28.228,23.986z"/>
            </g>
          </svg>
        </div>
        <div class="overview-block">
          <img :src="'https://image.tmdb.org/t/p/w500/' + filmdetail.poster_path" alt="">
          <div class="info">
            <p :id="'id'+index" class="title"><b>{{filmdetail.title}}</b> <br> {{filmdetail.original_title}}</p>
            <div class="votes">
              <svg version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
              viewBox="0 0 53.867 53.867" style="enable-background:new 0 0 53.867 53.867;" xml:space="preserve">
              <polygon style="fill:#EFCE4A;" points="26.934,1.318 35.256,18.182 53.867,20.887 40.4,34.013 43.579,52.549 26.934,43.798
              10.288,52.549 13.467,34.013 0,20.887 18.611,18.182 "/>
              </svg>
              <span class="vote-average">{{filmdetail.vote_average}}</span>
              <span class="vote-count">({{filmdetail.vote_count}})</span>
            </div>
            <p class="genres-ttl">Жанры:</p>
            <ul class="genres-list">
              <li v-for="genre in filmdetail.genres" :key="genre.id">{{genre.name}}</li>
            </ul>
            <p class="desc">{{filmdetail.overview}}</p>
          </div>
        </div>
        <!-- Вывод первого трейлера -->
        <p class="trailer">Трейлер</p>
        <iframe v-if="videosrc != ''" width="100%" height="315" :src="videosrc" frameborder="0" allowfullscreen></iframe>
        <div v-if="videosrc == ''">
          Трейлер не найден!
        </div>
      </div>
    </div>
    <img alt="Vue logo" src="./assets/logo.png">
    <h1>Фильмы The Movie Database</h1>
    <!-- Вывод категорий фильмов  -->
    <ul class="cats">
      <li :class="{'cat-item': true,'cn-active': index == currentIndex}" v-for="(cat,index) in cats" :key="cat.id"  @click="fetchData(cat.catname);currentIndex = index">{{cat.name}}</li>
    </ul>
    <!-- Вывод списка фильмов  -->
    <ul class="films-list">
      <li v-for="(result,index) in results" :key="result.id" class="list-item">
        <img :src="'https://image.tmdb.org/t/p/w500/' + result.poster_path" alt="12">
        <div class="info">
          <p :id="'id'+index" class="title"><b>{{result.title}}</b> <br> {{result.original_title}}</p>
          <p class="desc">{{result.overview.trimtxt(200)}}</p>
          <div class="butt-detail" v-on:click="getFilm(result.id);">
            Подробнее
          </div>
        </div>
      </li>
    </ul>
  </div>
</template>
<script>
//Функция обрезания текста по кол-ву символов.
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
      filmdetail: {videos: ''},
      videosrc: '',
      modalBool: false,
      cats: [{id: 1,name: 'Популярные',catname:'popular'},{id: 2,name:'Лучшие',catname:'top_rated'},{id: 3,name:'Ожидаемые',catname: 'upcoming'},{id: 4, name: 'Смотрят сейчас',catname: 'now_playing'}],
    }
  },
  mounted(){
    this.fetchData('popular'); //Назначаем изначальный вывод списка фильмов из категории ( в данном случае "популярные")
  },
  methods: {
    rScroll: function(){
      document.getElementsByTagName('body')[0].classList.remove('nonscroll');
    },
    //Функция запроса списка фильмов по определенной категории
    fetchData: function(catname){
      let cat = catname;
      axios.get('https://api.themoviedb.org/3/movie/' + cat + '?api_key=7e00b848ffbb0a2bb957f6631e1ad255&language=ru-RU')
       .then(response => {this.results = response.data == 'error' ? [] : response.data.results;})
       .catch(error => {
             this.errors.push(error);
       });
    },
    //Функция получения подробной информации о выбранном фильме и получение трейлера
    getFilm: function(id){
      //запрос на информацию
      axios.get('https://api.themoviedb.org/3/movie/' + id + '?api_key=7e00b848ffbb0a2bb957f6631e1ad255&language=ru-RU')
       .then(response => {this.filmdetail = response.data == 'error' ? [] : response.data;})
       .catch(error => {
             this.errors.push(error);
       });
       //запрос на трейлер
       axios.get('https://api.themoviedb.org/3/movie/' + id + '/videos?api_key=7e00b848ffbb0a2bb957f6631e1ad255&language=ru-RU')
        .then(response => {this.filmdetail.videos = response.data == 'error' ? [] : response.data.results; this.videosrc = this.filmdetail.videos.length > 0 ? 'https://www.youtube.com/embed/' + this.filmdetail.videos[0].key : ''})
        .catch(error => {
              this.errors.push(error);
        });
        document.getElementsByTagName('body')[0].classList.add('nonscroll');
        let self = this;
        setTimeout(function(){
          self.modalBool = true;
        },300);
    },
  },
}
</script>

<style>
.nonscroll{
  overflow: hidden;
}
/* хром, сафари */
.film-detail-modal::-webkit-scrollbar { width: 0; }

/* ie 10+ */
.film-detail-modal { -ms-overflow-style: none; }

/* фф (свойство больше не работает, других способов тоже нет)*/
.film-detail-modal { overflow: -moz-scrollbars-none; }
.trailer{
  text-align: left;
  font-weight: 700;
  font-size: 14px;
  border-bottom: 3px solid #d4d4d4;
  padding-bottom: 7px;
  position: relative;
}
.trailer:after{
  content: '';
  width: 60px;
  height: 3px;
  background-color: #000;
  position: absolute;
  bottom: -3px;
  left: 0;
}
.genres-ttl{
  font-size: 12px;
  margin-bottom: 5px;
  color: #878787;
  font-weight: 700;
  margin-top: 20px;
}
.genres-list{
  list-style: none;
  padding: 0;
  margin: 0;
}
.genres-list li{
  display: inline-block;
  cursor: pointer;
  color: #878787;
  font-size: 12px;
  margin-right: 10px;
  text-transform: capitalize;
}
.votes{
  margin: 0;
  display: flex;
  align-items: center;
}
.votes span{
  padding-top: 3px;
}
.votes .vote-average{
  margin-right: 5px;
  font-weight: 700;
}
.votes .vote-count{
  color: #878787;
  font-size: 12px;
}
.votes svg{
  width: 20px;
  margin-right: 5px;
}
.overview-block{
  display: flex;
  margin-bottom: 40px;
  align-items: flex-start;
  justify-content: flex-start;
}
.overview-block img{
  width: 200px;
  height: auto;
  margin-right: 20px;
}
.overview-block .info{
  text-align: left;
}
.overview-block .info .title{
  font-size: 16px;
  color: #6e6e6e;
  margin-top: 0;
  line-height: 25px;
  margin-bottom: 3px;
}
.overview-block .info .title b{
  color: #000;
  font-size: 18px;
}
.overview-block .info .desc{
  color: #6e6e6e;
  font-size: 14px;
  line-height: 22px;
}
.back-black{
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  overflow: hidden;
  position: fixed;
  background-color: rgba(0,0,0,0.3);
}
.film-detail-modal{
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  overflow-y: auto;
  z-index: 1000;
  position: fixed;
  display: none;
}
.modal-open{
  display: block;
}
.modal-inner{
  width: 600px;
  height: auto;
  border-radius: 5px;
  padding: 20px;
  margin: 0 auto;
  margin-top: 20px;
  background-color: #fff;
  position: relative;
  margin-bottom: 30px;
  animation: modalopen 0.5s ease;
}
.modal-close{
  width: 15px;
  cursor: pointer;
  position: absolute;
  z-index: 10;
  right: 20px;
  top: 20px;
  height: 15px;
}
@keyframes listupdate {
  0%{
    opacity: 0;
    transform: scale(0.9);
  }
  100%{
    opacity: 1;
    transform: scale(1);
  }
}
@keyframes modalopen{
  0%{
    opacity: 0;
    transform: scale(0.5);
  }
  100%{
    opacity: 1;
    transform: scale(1);
  }
}
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
  animation: listupdate 0.5s ease;
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

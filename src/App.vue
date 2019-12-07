
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
            <p class="title"><b>
              <span v-show="filmdetail.title">{{filmdetail.title}}</span>
              <span v-show="filmdetail.name">{{filmdetail.name}}</span>
            </b> <br> {{filmdetail.original_title}}</p>
            <div class="votes">
              <svg version="1.1" id="Capa_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
              viewBox="0 0 53.867 53.867" style="enable-background:new 0 0 53.867 53.867;" xml:space="preserve">
              <polygon style="fill:#EFCE4A;" points="26.934,1.318 35.256,18.182 53.867,20.887 40.4,34.013 43.579,52.549 26.934,43.798
              10.288,52.549 13.467,34.013 0,20.887 18.611,18.182 "/>
              </svg>
              <span class="vote-average">{{filmdetail.vote_average}}</span>
              <span class="vote-count">({{filmdetail.vote_count}})</span>
            </div>
            <p class="genres-ttl modal-ttl">Жанры:</p>
            <ul class="genres-list">
              <li v-for="genre in filmdetail.genres" :key="genre.id">{{genre.name}}</li>
            </ul>
            <p v-if="filmdetail.release_date" class="modal-ttl">Год: {{filmdetail.release_date.split('-')[0]}}</p>
            <p class="desc">{{filmdetail.overview}}</p>
          </div>
        </div>
        <!-- Вывод первого трейлера -->
        <p class="trailer">Трейлер</p>
        <iframe v-if="videosrc != ''" width="100%" height="315" :src="videosrc" frameborder="0" allowfullscreen></iframe>
        <div v-if="videosrc == ''" class="non-trailer">
          Трейлер не найден!
        </div>
      </div>
    </div>

    <!-- Шапка с поиском -->
    <div class="header">
      <!--
        <div class="images">
          <img alt="Vue logo" src="./assets/logo.png" class="vue-img">
          <img class="tmdb-img" src="https://www.themoviedb.org/assets/2/v4/logos/primary-green-d70eebe18a5eb5b166d5c1ef0796715b8d1a2cbc698f96d311d62f894ae87085.svg" alt="">
        </div>
      <h1>Фильмы The Movie Database</h1>
      -->
      <div class="search-block">
          <!-- Компонент быстрого поиска -->
          <FastSearch @getfilm="getFilm($event)"></FastSearch>
      </div>
    </div>

    <!-- Вывод категорий фильмов  -->
    <ul class="tv-movie">
      <li v-for="(type,index) in types" :key="type.id" :class="{'active': index == currentIndexType}" @click="currenttype = type.name;cats  = type.name == 'movie' ? filmcats : tvcats;fetchData(cats[currentIndex].catname, currentpage, currenttype);currentIndexType = index;">{{type.title}}</li>
    </ul>
    <ul class="cats">
      <li :class="{'cat-item': true,'cn-active': index == currentIndex}" v-for="(cat,index) in cats" :key="cat.id"  @click="currentpage = 1;fetchData(cat.catname, currentpage, currenttype);currentIndex = index;">{{cat.name}}</li>
    </ul>

    <!-- Вывод списка фильмов  -->
    <ul class="films-list">
      <li v-for="(result,index) in results" :key="result.id" class="list-item">
        <img :src="'https://image.tmdb.org/t/p/w500/' + result.poster_path" alt="12" v-if="result.poster_path != null">
        <div class="block-no-img" v-if="result.poster_path == null" style="width: 180px;height: 270px; background-color: #050a1f;margin-right: 20px;border-radius: 5px;color: #fff;display: flex;justify-content: center;align-items: center;text-align: center;font-weight: 700;font-size: 16px;">
            Постер<br>не найден
        </div>
        <div class="info">
          <p :id="'id'+index" class="title">
            <!-- Для фильма -->
            <span v-if="result.title"><b>{{result.title}}</b> <br> {{result.original_title}}</span>
            <!-- Для Сериала -->
            <span v-if="result.name"><b>{{result.name}}</b> <br> {{result.original_name}}</span>
          </p>
          <p class="release-date">Год: <span v-if="result.release_date">{{result.release_date.split('-')[0]}}</span> <span v-if="result.first_air_date">{{result.first_air_date.split('-')[0]}}</span></p>
          <p class="desc">{{result.overview.trimtxt(200)}}</p>
          <div class="butt-detail" v-on:click="getFilm([currenttype,result.id]);">
            Подробнее
          </div>
        </div>
      </li>
    </ul>

    <!-- Переключение между страницами -->
    <div class="arrows">
      <div :class="{'prev': true, 'disabled': currentpage == 1}" @click="currentpage = currentpage == 1 ? 1 : currentpage-1;fetchData(cats[currentIndex].catname, currentpage, currenttype);">
        Назад
      </div>
      <div :class="{'next': true, 'disabled': currentpage == totalpages}" @click="currentpage = currentpage == totalpages ? totalpages : currentpage+1;fetchData(cats[currentIndex].catname, currentpage, currenttype);">
        Далее
      </div>
    </div>

    <p class="copyright copy-footer">Приложение разработано <a href="https://github.com/svaticalm" target="_blank">svaticalm</a></p>
  </div>
</template>
<script>
import FastSearch from './components/fastsearch.vue'
import axios from 'axios'

//Функция обрезания текста по кол-ву символов.
String.prototype.trimtxt = function (length) {
  return this.length > length ? this.substring(0, length) + "..." : this;
}

export default {
  name: 'app',
  components: {
    FastSearch,
  },
  data: function(){
    return {
      results: [], //Результаты запроса к бд по фильмам или сериалам
      errors: [], //Ошибки при запросе
      currentIndex: 0, //Индекс выбранной категории
      currentIndexType: 0,
      filmdetail: [], //Детальное описание фильма при нажатии на кнопку подробнее
      videosrc: '', //Путь к первому видео из filmdetail
      modalBool: false, //Отвечает за вызов модалки (true - открыта, false - закрыта)
      totalpages: '', //Кол-во страниц для переключения
      currentpage: 1, //Текущая страница
      currenttype: 'movie',
      randomFilm: '',
      //Медиа тип
      types: [{id: 1, name: 'movie', title: 'Фильмы'},{id: 2, name: 'tv', title: 'Сериалы'}],
      //Категории
      filmcats: [{id: 1,name: 'Популярные',catname:'popular'},{id: 2,name:'Лучшие',catname:'top_rated'},{id: 3,name:'Ожидаемые',catname: 'upcoming'},{id: 4, name: 'Смотрят сейчас',catname: 'now_playing'}],
      tvcats: [{id: 1,name: 'Популярные',catname:'popular'},{id: 2,name:'Лучшие',catname:'top_rated'},{id: 3,name:'На этой неделе',catname: 'on_the_air'},{id: 4, name: 'Смотрят сейчас',catname: 'airing_today'}],
      cats: [{id: 1,name: 'Популярные',catname:'popular'},{id: 2,name:'Лучшие',catname:'top_rated'},{id: 3,name:'Ожидаемые',catname: 'upcoming'},{id: 4, name: 'Смотрят сейчас',catname: 'now_playing'}],
    }
  },
  mounted(){
    this.fetchData('popular', this.currentpage, this.currenttype); //Назначаем изначальный вывод списка фильмов из категории ( в данном случае "популярные")
    let self = this;
    setTimeout(function(){
      self.getFilm(['movie', self.results[self.randomFilm].id]);
    }, 300);
  },
  methods: {
    //Функция запроса списка фильмов по определенной категории
    fetchData: function(catname,page,type){
      let cat = catname;
      let curpage = page;
      axios.get('https://api.themoviedb.org/3/'+ type +'/' + cat + '?api_key=7e00b848ffbb0a2bb957f6631e1ad255&language=ru-RU&page='+ curpage)
       .then(response => {this.results = response.data == 'error' ? [] : response.data.results;this.totalpages = response.data.total_pages;this.randomFilm = Math.floor(Math.random() * (20 - 0) + 0);})
       .catch(error => {
             this.errors.push(error);
       });
    },
    getRandom: function(min,max){
      return Math.floor(Math.random() * (max - min)) + min;
    },
    //Функция получения подробной информации о выбранном фильме
    getFilm: function(arr){
      //В аргументе передается массив с двумя значениями (arr[0] - тип контента фильм или сериал, arr[1] - id контента для поиска)
      axios.get('https://api.themoviedb.org/3/'+ arr[0] + '/' + arr[1] + '?api_key=7e00b848ffbb0a2bb957f6631e1ad255&language=ru-RU&append_to_response=videos')
       .then(response => {this.filmdetail = response.data == 'error' ? [] : response.data;this.videosrc = this.filmdetail.videos.results.length > 0 ? 'https://www.youtube.com/embed/' + this.filmdetail.videos.results[0].key : ''})
       .catch(error => {
             this.errors.push(error);
       });
       //
        document.getElementsByTagName('body')[0].classList.add('nonscroll'); // Убираем скролл если открыта модалка
        let self = this;
        setTimeout(function(){
          self.modalBool = true;  //Открываем модалку с таймаутом
        },300);
    },
    rScroll: function(){
      document.getElementsByTagName('body')[0].classList.remove('nonscroll'); //Вовзращаем скролл
    },
  },
}
</script>

<style>
.non-trailer{
  color: #fff;
  margin-top: 20px;
}
.search-block{
  width: 1020px;
  margin: 0 auto;
  margin-bottom: 20px;
  margin-top: 20px;
}
.arrows{
  width: 1018px;
  margin: 0 auto;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: justify;
      -ms-flex-pack: justify;
          justify-content: space-between;
  margin-top: 20px;
}
.arrows .next,
.arrows .prev{
    border-radius: 5px;
    background-color: #3f4757;
    color: #fff;
    padding: 10px 20px 10px 20px;
    cursor: pointer;
    -webkit-transition: all .2s;
    -o-transition: all .2s;
    transition: all .2s;
}
.arrows .next:hover,
.arrows .prev:hover{
  background-color: #ff6161;
}
.arrows .next:active,
.arrows .prev:active{
  background-color: #ff4444;
}
.arrows .disabled{
  background-color: #2c2f3d;
  color: #545969;
}
.arrows .disabled:hover{
  background-color: #2c2f3d;
}
.arrows .disabled:active{
  background-color: #2c2f3d;
}
.tmdb-img{
  width: 55px;
  margin-left: 20px;
}
.vue-img{
  width: 50px;
}
.copyright{
  font-size: 12px;
  color: #3f4757;
  margin-bottom: 30px;
  margin-top: 30px;
}
.copyright a{
  color: #3f4757;
  font-weight: 700;
}
.copy-footer{
  margin-bottom: 0;
  padding-bottom: 30px;
}
.nonscroll{
  overflow: hidden;
  padding-right: 18px;
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
  border-bottom: 3px solid #3d4d6c;
  padding-bottom: 7px;
  position: relative;
  color: #fff;
}
.trailer:after{
  content: '';
  width: 60px;
  height: 3px;
  background-color: #fffe;
  position: absolute;
  bottom: -3px;
  left: 0;
}
.modal-ttl{
  font-size: 12px;
  margin-bottom: 5px;
  color: #808eab;
  font-weight: 700;
  margin-top: 10px;
}
.genres-ttl{
  font-size: 12px;
  margin-bottom: 5px;
  color: #808eab;
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
  color: #808eab;
  font-size: 12px;
  margin-right: 10px;
  text-transform: capitalize;
}
.votes{
  margin: 0;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
}
.votes span{
  padding-top: 3px;
}
.votes .vote-average{
  margin-right: 5px;
  color: #808eab;
  font-weight: 700;
}
.votes .vote-count{
  color: #808eab;
  font-size: 12px;
}
.votes svg{
  width: 20px;
  margin-right: 5px;
}
.overview-block{
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  margin-bottom: 40px;
  -webkit-box-align: start;
      -ms-flex-align: start;
          align-items: flex-start;
  -webkit-box-pack: start;
      -ms-flex-pack: start;
          justify-content: flex-start;
}
.overview-block img{
  width: 200px;
  height: auto;
  margin-right: 20px;
  border-radius: 5px;
}
.overview-block .info{
  text-align: left;
}
.overview-block .info .title{
  font-size: 16px;
  color: #808eab;
  margin-top: 0;
  line-height: 25px;
  margin-bottom: 3px;
}
.overview-block .info .title b{
  color: #fff;
  font-size: 18px;
}
.overview-block .info .desc{
  color: #808eab;
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
  background-color: rgba(0,0,0,0.8);
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
  background-color: #13182b;
  position: relative;
  margin-bottom: 30px;
  -webkit-animation: modalopen 0.5s ease;
          animation: modalopen 0.5s ease;
}
.modal-close{
  width: 12px;
  cursor: pointer;
  position: absolute;
  z-index: 10;
  right: 20px;
  top: 20px;
  height: 12px;
  -webkit-transition: all .1s;
  -o-transition: all .1s;
  transition: all .1s;
}
.modal-close path{
  fill: #fff;
}
.modal-close:hover{
  -webkit-transform: scale(1.1);
      -ms-transform: scale(1.1);
          transform: scale(1.1);
}
@-webkit-keyframes listupdate {
  0%{
    opacity: 0;
    -webkit-transform: scale(0.9);
            transform: scale(0.9);
  }
  100%{
    opacity: 1;
    -webkit-transform: scale(1);
            transform: scale(1);
  }
}
@keyframes listupdate {
  0%{
    opacity: 0;
    -webkit-transform: scale(0.9);
            transform: scale(0.9);
  }
  100%{
    opacity: 1;
    -webkit-transform: scale(1);
            transform: scale(1);
  }
}
@-webkit-keyframes modalopen{
  0%{
    opacity: 0;
    -webkit-transform: scale(0.5);
            transform: scale(0.5);
  }
  100%{
    opacity: 1;
    -webkit-transform: scale(1);
            transform: scale(1);
  }
}
@keyframes modalopen{
  0%{
    opacity: 0;
    -webkit-transform: scale(0.5);
            transform: scale(0.5);
  }
  100%{
    opacity: 1;
    -webkit-transform: scale(1);
            transform: scale(1);
  }
}
body{
  margin: 0;
}
.cn-active, .tv-movie li.active{
  color: #fff !important;
}
.cats,.tv-movie{
  padding: 0;
  width: 1020px;
  margin: 0 auto;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: justify;
      -ms-flex-pack: justify;
          justify-content: flex-start;
  list-style: none;
  margin-bottom: 40px;
  margin-top: 40px;

}
.tv-movie{
  margin-top: 30px
}
.tv-movie li{
  font-weight: 900
}
.cat-item, .tv-movie li{
  cursor: pointer;
  color: #303d57;
  -webkit-transition: all .2s;
  -o-transition: all .2s;
  transition: all .2s;
  margin-right: 50px;
}
.cat-item:hover, .tv-movie li:hover{
  color: #fff;
}
h1{
  margin-bottom: 30px;
}
.info{
  position: relative;
}
.release-date{
  font-size: 12px;
  color: #808eab;
  font-weight: 700;
}
.butt-detail{
  color: #ff4444;
  border: 2px solid #ff4444;
  width: 170px;
  height: 30px;
  font-weight: 700;
  border-radius: 5px;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-align: center;
      -ms-flex-align: center;
          align-items: center;
  -webkit-box-pack: center;
      -ms-flex-pack: center;
          justify-content: center;
  position: absolute;
  bottom: 15px;
  font-size: 14px;
  -webkit-transition: all .1s;
  -o-transition: all .1s;
  transition: all .1s
}
.butt-detail:hover{
  background-color: #ff4444;
  color: #fff;
}
.butt-detail:active{
  transform: scale(0.9);
}
.films-list{
  width: 1020px;
  margin: 0 auto;
  padding: 0;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-align: start;
      -ms-flex-align: start;
          align-items: flex-start;
  -webkit-box-pack: justify;
      -ms-flex-pack: justify;
          justify-content: space-between;
  -ms-flex-wrap: wrap;
      flex-wrap: wrap;
}
.list-item{
  width: 480px;
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  text-align: left;
  padding-right: 20px;
  border-radius: 5px;
  overflow: hidden;
  cursor: pointer;
  margin-bottom: 30px;
  -webkit-transition: all .2s;
  -o-transition: all .2s;
  transition: all .2s;
  -webkit-animation: listupdate 0.5s ease;
          animation: listupdate 0.5s ease;
}
.list-item:hover{
  -webkit-box-shadow: 0 0 20px 3px rgba(0,0,0,0.2);
          box-shadow: 0 0 20px 3px rgba(0,0,0,0.2);
  -webkit-transform: perspective(1010px) rotateY(10deg);
          transform: perspective(1010px) rotateY(10deg);
}
.list-item img{
  width: 180px;
  margin-right: 20px;
  border-radius: 5px;
}
.list-item .title{
  color: #808eab;
  font-size: 12px;
  line-height: 22px;
}
.list-item .title b{
  color: #fff;
  font-size: 16px;
}
.list-item .desc{
  font-size: 12px;
  line-height: 18px;
  color: #808eab;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  background-color: #13182b;
}
.header{
  background-color: #050a1f;
  color: #fff;
  padding-top: 1px;
  padding-bottom: 5px;
  margin-bottom: 30px;
}
@media screen and (max-width: 1050px){
  .films-list{
    width: 600px;
    margin: 0 auto;
  }
  .list-item{
    width: 600px;
  }
  .search-block{
    width: 100%;
    padding-left: 15px;
    padding-right: 15px;
    box-sizing: border-box;
  }
  .cats{
    width: 100%;
    box-sizing: border-box;
    padding-right: 15px;
    padding-left: 15px;
    justify-content: center;
  }
  .arrows{
    width: 100%;
    padding-left: 15px;
    padding-right: 15px;
    box-sizing: border-box;
  }
  .nonscroll{
    padding-right: 0;
  }
}
@media screen and (max-width: 700px){
  .films-list{
    width: 90%;
    margin: 0 auto;
  }
  .cats{
    width: 100%;
    -ms-flex-wrap: wrap;
        flex-wrap: wrap;
    -webkit-box-pack: center;
        -ms-flex-pack: center;
            justify-content: center;
  }
  .cats li{
    margin-right: 10px;
    margin-bottom: 10px;
  }
  .butt-detail{
    bottom: -40px;
  }
  .list-item{
    border-radius: 0;
    width: 100%;
    -webkit-box-orient: vertical;
    -webkit-box-direction: normal;
        -ms-flex-direction: column;
            flex-direction: column;
    height: auto;
    -webkit-box-pack: start;
        -ms-flex-pack: start;
            justify-content: flex-start;
    -webkit-box-align: start;
        -ms-flex-align: start;
            align-items: flex-start;
    padding-bottom: 60px;
    margin-bottom: 40px;
  }
  .list-item img{
    width: 100% !important;
    height: auto;
  }
  .list-item:hover{
    -webkit-box-shadow: none;
            box-shadow: none;
  }
  .modal-inner{
    width: 100%;
    padding: 0;
    border-radius: 0;
    margin: 0 auto;
  }
  .modal-inner img{
    margin-bottom: 15px;
  }
  .trailer{
    margin-left: 20px;
    margin-right: 20px;
  }
  iframe{
    height: 170px;
    margin-bottom: 10px;
  }
  .modal-inner .overview-block{
    -webkit-box-orient: vertical;
    -webkit-box-direction: normal;
        -ms-flex-direction: column;
            flex-direction: column;
    padding-left: 20px;
    padding-top: 20px;
    padding-right: 20px;
  }
}
</style>

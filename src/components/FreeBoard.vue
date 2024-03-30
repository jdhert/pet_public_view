<template>
  <header class="banner">
      <h1 class="banner-title">반려동물의 실시간 인기 사진첩</h1>
  </header>
  <section class="ftco-section bg-light">
    <div class="freeboard">
      <div class="row1 d-flex">
        <div v-for="(post, index) in posts" :key="index" class="col-md-4 d-flex" style="cursor: pointer;">
          <div class="content-entry align-self-stretch">
            <a @click.prevent="openModal(post)" class="block-20 rounded" :style="{backgroundImage: 'url(' + (post.imgPath ? post.imgPath : '@/assets/images/gallery-6.jpg') + ')'}"></a>
            <div class="text p-4">
              <div class="meta mb-2">
                <div><a href='#'>{{ formatDate(post.createdAt) }}</a></div>
                <div><a href="#">{{ post.writer }}</a></div>
                <div class="meta-chat">
                  <span class="fa fa-comment"></span> {{ post.commentCount }}
                  <span class="fa fa-heart" style="margin-left: 5px;"></span> {{ post.likeCount }}
                </div>
              </div>
              <h3 class="heading"><a href="#">{{ post.title }}</a></h3>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
  <header>
      <img src="../assets/images/banner.png" alt="Banner" class="banner-image">  
      <h2>반려동물과의 일상을 사진과 함께 사람들과 공유하세요</h2>
  </header>
  <div class="search-bar" style="display: flex; align-items: center;">
  <div class="search-bar1">
    <select class="search-select1" v-model="type1">
      <option value="Latest">최신순</option>
      <option value="Oldest">오래된순</option>
    </select>
  </div>
  <div style="flex-grow: 0.08;">
    <select class="search-select" v-model="type">
      <option value="title">제목</option>
      <option value="content">내용</option>
      <option value="tag">태그</option>
      <option value="writer">작성자</option>
    </select>
  </div>
  <form @submit.prevent="searching">
    <input type="search" class="search-input"  placeholder="검색어를 입력할거냥" v-model="search">
    <input type="submit" class="search-button" value="검색">
  </form>
</div>
  <section class="ftco-section1 bg-light">
    <div class="freeboard2">
      <div class="row2">
        <div class="col-md-4 addpost-item" v-for="(addpost, index) in addposts" :key="index">
      <div class="content-entry align-self-stretch" @click="openModal(addpost)">
        <a class="block-20 rounded" :style="{backgroundImage: 'url(' + (addpost.imgPath ? addpost.imgPath : '@/assets/images/gallery-6.jpg') + ')'}"></a>
        <div class="text p-4">
          <div class="meta">
            <div class="createdAt"><a href="#">{{ formatDate(addpost.createdAt) }}</a></div>
            <div class="wirter"><a href="#">{{ addpost.writer }}</a></div>
            <div class="meta-chat">
              <span class="fa fa-comment"></span> {{ addpost.commentCount }}
              <span class="fa fa-heart" style="margin-left: 5px;"></span> {{ addpost.likeCount }}
            </div>
          </div>
          <h3 class="heading"><a href="#">{{ addpost.title }}</a></h3>
        </div>
      </div>
    </div>
  </div>
</div>
</section>
<button v-if="isLogin" class="btn btn-success mt-3 custom-button" @click="goToWrite">글쓰기</button>

<div class="row mt-5">
    <div class="col text-center">
      <div class="block-27">
        <ul>
            <li><a href="#" @click="currentSwap(this.currentPage-1)">&lt;</a></li>
            <li><a href="#"  v-for="n in this.numbers" :key="n" @click="currentSwap(n)"  :class="{ 'active': currentPage === n }" style="margin: 5px;">{{ n }}</a></li>
            <li><a href="#" @click="currentSwap(this.currentPage+1)">&gt;</a></li>
        </ul>
      </div>
    </div>
  </div>
  <!-- <detailFreeBoard v-if="showModal" :selectedCard="selectedCard" @closeModal="showModal = false" @tagSearch="handleTagSearch" @deleteBoard="realDelete" @updateViewCount="increaseViewCount" :viewCount="selectedCard.viewCount"/> -->
  <detailFreeBoard v-if="showModal" :selectedCard="selectedCard" @closeModal="showModal = false" @tagSearch="handleTagSearch" @deleteBoard="realDelete" @updateViewCount="increaseViewCount" :viewCount="selectedCard.viewCount" @openModal="handleModalOpen"/>
</template>

<script>
import detailFreeBoard from './detailFreeBoard.vue';

export default {
  emits: ['forceRerender'],
  computed:{
    isLogin() {
        return this.$cookies.isKey('id') ? true : false;
    }
  },
  components : {
		detailFreeBoard
	},
  props: {
    showModal: Boolean,
    selectedCard: Object
  },
  data() {
    return {
      showModal: false, // 모달창 열림 여부
      selectedCard: {}, // 선택된 카드 정보,
      posts: [],
      addposts: [],
      maxPage : 1,
      paginationLimit : 5,
      currentPage : 1,
      search : "",
      type : "title",
      type1 : "Latest",
      numbers : [],
      postId: null,
    }
  },
  methods: {
      formatDate(dateString) {
        const date = new Date(dateString);
        const year = date.getFullYear();
        const month = ('0' + (date.getMonth() + 1)).slice(-2);
        const day = ('0' + date.getDate()).slice(-2);
        return `${year}-${month}-${day}`;
      },
      handleModalOpen(post) {
        this.increaseViewCount(post.id);
      },
      goToWrite() {
        this.$router.push(`/addphoto`); 
      }, 
      currentSwap(n) {
        this.currentPage = Math.max(1, Math.min(n, this.maxPage));
        let startPage = Math.max(1, Math.floor((this.currentPage - 1) / this.paginationLimit) * this.paginationLimit + 1);
        this.getBoard(this.currentPage);
      },
      getPageNumbers() {
        this.numbers = [];
        let startPage = Math.max(1, Math.floor((this.currentPage - 1) / this.paginationLimit) * this.paginationLimit + 1);
        let endPage = Math.min(startPage + this.paginationLimit - 1, this.maxPage);

        for (let i = startPage; i <= endPage; i++) {
            this.numbers.push(i);
        }       
      },
      getBoard(startPage) {
        this.addposts = [];
        this.axios.get(`/api/free/search/${startPage}`, {
          params: { 
            search: this.search,
            type: this.type,
            type1: this.type1,
            subject : 0
          }
        }).then((res) => {
          console.log(res.data);
          this.addposts = res.data;
          this.maxPage= Math.ceil(this.addposts[0].totalRowCount/8);
          if(this.maxPage == 0)
            this.maxPage = 1;
          this.getPageNumbers();
        }).catch((error) => {
          console.error(error);
        });
      },
      openModal(post) {
        this.selectedCard = post;
        this.$emit('updateViewCount', post.id);
        this.showModal = true;
      },
      async increaseViewCount(postId) {
        console.log(postId)
        try {
          const response = await this.axios.put(`/api/free/view/${postId}`);
          console.log('응답:', response.data);
          await this.fetchViewCount(postId);
          return response.data;
        } catch (error) {
          console.error('에러:', error);
          throw error;
        }
      },
      async fetchViewCount(postId) {
        try {
          const response = await this.axios.get(`/api/free/view/${postId}`);
          console.log('조회수:', response.data);
          return response.data;
        } catch (error) {
          console.error('조회수를 불러오는 중 오류 발생:', error);
          throw error;
        }
      },
      closeModal() {
        this.$emit('closeModal');
      },
      searching() {
        this.addposts = [];
        this.axios.get(`/api/free/search/1`, {
          params: { 
            search: this.search,
            type: this.type,
            type1: this.type1,
            subject : 0
          }
        }).then((res) => {
          console.log(res.data);
          this.addposts = res.data;
          this.maxPage = Math.ceil(this.addposts[0].totalRowCount/8);
          if(this.maxPage == 0)
            this.maxPage = 1;
          this.getPageNumbers();

        }).catch((error) => {
          console.error(error);
        });
      },
      realDelete(id){
        this.showModal = false;
        console.log(id);
        this.axios.delete(`/api/free/${id}`)
        .then(() => {
          console.log('게시글이 성공적으로 삭제되었습니다.');
          this.getBoard();
          this.$cookies.remove('boardId');
          this.$router.push(`/freeboard3`).then(() => {
            window.location.reload();
          });
        })
        .catch(error => {
          console.error('게시글 삭제 중 오류가 발생했습니다.', error);
        });
      },
      handleTagSearch(tag){
        this.showModal=false;
        this.axios.get(`/api/free/search/1`, {
          params: { 
            search: tag,
            type: 'tag',
            type1: 'Latest',
            subject : 0
          }
        }).then((res) => {
            this.addposts = res.data;
            this.maxPage= Math.ceil(this.addposts[0].totalRowCount/8);
            if(this.maxPage == 0)
              this.maxPage = 1;
            this.getPageNumbers();
        }).catch();
      },
      openModalForPost(postId) {  
        this.axios.get(`/api/free/get/${postId}`)
          .then((res) => {
            this.selectedCard = res.data;
            this.showModal = true;
          })
          .catch((error) => {
            console.error('게시물 정보를 가져오는 중 오류 발생:', error);
          });
      },
      fetchPostData(boardId) {
        // Axios를 사용하여 서버에 HTTP GET 요청을 보냅니다.
        this.axios.get(`/api/free/get/${boardId}`)
        .then(response => {
          // 응답으로 받은 데이터를 처리합니다.
          console.log(response.data);
          this.selectedCard = response.data;
          this.showModal = true;
        })
        .catch(error => {
          console.error('Error fetching post data:', error);
        });
      }
  },
  mounted(){
    this.axios.get(`/api/free/1`, {
      params: {
        subject: 0
      }
    }).then((res) => {
      this.addposts = res.data;
      this.maxPage = Math.ceil(this.addposts[0].totalRowCount/8);
      if(this.maxPage == 0)
        this.maxPage = 1;
      this.getPageNumbers();
      // console.log("이거 확인",this.addposts)

      }).catch((error) => {
          console.error('Error fetching data:', error);
      });

    this.axios.get(`/api/free/popular`,{
      params: {
        subject : 0
      }
    }).then((res) =>{
      console.log(res)
      this.posts = res.data;
      console.log("인기게시글", this.posts)
    }).catch();
    
    // 예를 들어 쿼리 매개변수로부터 ID를 가져올 때:
    // this.postId = this.$route.query.postId;
    // 또는 쿠키로부터 ID를 가져올 때:
    this.postId = this.$cookies.get('postId');
    if (this.postId) {
      this.openModalForPost(this.postId);
      this.$cookies.remove("postId");
    }

    // 추출한 ID를 사용하여 서버에 데이터를 요청하는 로직
    if (this.$route.path.includes('/get/') && this.$route.params.id) {
      const postId = this.$route.params.id;
      this.fetchPostData(postId);
    }

    // 추출한 ID를 사용하여 서버에 데이터를 요청하는 로직
    if (this.$route.path.includes('/get/') && this.$route.params.id) {
      const postId = this.$route.params.id;
      this.fetchPostData(postId);
    }
  }
}

</script>

<style scoped>



@font-face {
font-family: 'Ownglyph_meetme-Rg';
src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2402_1@1.0/Ownglyph_meetme-Rg.woff2') format('woff2');
font-weight: normal;
font-style: normal;
}

* {
    font-family: 'Ownglyph_meetme-Rg';
}

.content-entry:hover{
  
  box-shadow: 0 8px 12px rgba(0, 0, 0, 0.2);
  transform: translateY(-3px);

}


.banner {
margin-top: 120px;
text-align: center;
}

.banner-title {
font-family: 'Ownglyph_meetme-Rg';
font-size: 2.3rem;
}

.banner-subtitle {
font-family: 'Ownglyph_meetme-Rg';
font-size: 1.3rem;
color: #777;
}
.banner-image {
  margin-top: 50px;
      width: 50%;
  }
  
.freeboard {
display: flex;
justify-content: center; /* 가운데 정렬합니다 */
max-width: 1200px;
/* 원하는 너비로 조정 */
margin: 0 auto;
/* 가운데 정렬 */
background-color: white;
}

.freeboard2 {
    /* display: grid; */
    grid-template-columns: repeat(4, 1fr); /* 한 줄에 4개의 열을 생성 */
    gap: 20px; /* 아이템 사이의 간격 */
    justify-items: start; /* 아이템을 왼쪽 정렬 */
    width: 100%;
    margin: 0 auto; /* 페이지 중앙 정렬 */
    background-color: white;
}
.col-md-4 {
  margin-top: 20px;
  margin-right: 25x;
  margin-left: 25px;
}

.search-bar {
  display: flex;
  align-items: center;
  width: 1000px; /* 원하는 너비로 조정하세요 */
  margin: 0 auto; /* 가운데 정렬 */
    border: 3px solid #4ea3ff; /* 테두리 추가 */
  border-radius: 50px; /* 테두리의 모양을 더 둥글게 만들기 위해 추가 */
  padding: 5px; /* 내부 여백 추가 */

}

.search-bar1 {
  margin-right: 5px;
}

.search-select1 {
  font-family: 'Ownglyph_meetme-Rg';
  color: #222222;
  border-radius: 50px;
  width: 130px;
  border: none;
  border: 2px solid #4ea3ff; /* 테두리의 스타일과 색상을 지정합니다 */
  background: #fcfdff; 
  padding: 10px;
  font-size: 20px;
  text-align: center;
  outline: none;
  cursor: pointer;
}
.search-select {
  font-family: 'Ownglyph_meetme-Rg';
  color: #222222;
  border-radius: 50px; /* 테두리의 둥근 정도를 조절합니다 */
  width: 130px;
  border: 2px solid #4ea3ff; /* 테두리의 스타일과 색상을 지정합니다 */
  background: #fcfdff;
  padding: 10px;
  font-size: 20px;
  text-align: center;
  outline: none;
  cursor:pointer;
}


.search-input {
  font-family: 'Ownglyph_meetme-Rg';
  width: 610px;
  border: none;
  background: none;
  padding: 5px;
  font-size: 20px;
  border-radius: 60px;
  text-align: center;
  outline: none;
}

.search-select option {
color: #222222;
background-color: #fcfdff;
padding: 5px;
border-radius: 60px;
font-size: 20px;
}

.search-select option:hover {
background-color: #4ea3ff;
color: #ffffff;
}
.search-button {
  font-family: 'Ownglyph_meetme-Rg';
  color: #ffffff;
  border: none;
  background-color: #8d8d8d;
  font-size: 20px;
  border-radius: 80px;
  cursor: pointer;
  outline: none;
}
.search-button:hover {
background-color: #4ea3ff;
}
h1, h2, h3, h4, h5, h6 {
  font-family: 'Ownglyph_meetme-Rg';
margin-top: 0;
margin-bottom: 0.5rem; 
line-height: 1.5;
color: rgba(0, 0, 0, 0.8);
font-weight: 400;}


a {
color: #007bff;
text-decoration: none;
background-color: transparent; }
a:hover {
  color: #0056b3;
  text-decoration: underline; }

  .rounded {
border-radius: 0.25rem !important; }

a {
-webkit-transition: .3s all ease;
-o-transition: .3s all ease;
transition: .3s all ease;
color:  #007bff; }
a:hover, a:focus {
  text-decoration: none;
  color:  #007bff;
  outline: none !important; }

  /* .bg-light {
background: #f8f9fd !important; } */

.bg-secondary {
background: #207dff !important; }

.bg-primary {
background:  #007bff !important; }

.p-4 {
padding: 1.5rem !important; }

a.text-dark:hover, a.text-dark:focus {
color: #121416 !important; }

.freeboard {
font-family: "Montserrat", Arial, sans-serif;
background: #fff;
font-size: 15px;
line-height: 1.8;
font-weight: 400;
color: gray; }
.freeboard.menu-show {
  overflow: hidden;
  position: fixed;
  height: 100%;
  width: 100%; }

.row1 {
display: -webkit-box;
display: -ms-flexbox;
display: flex;
-ms-flex-wrap: wrap;
flex-wrap: wrap;
margin-right: -15px;
margin-left: -15px; }




/*이미지 css부분*/
.block-20 {
    overflow: hidden;
    background-size: cover;
    background-repeat: no-repeat;
    background-position: center center;
    height: 250px;
    position: relative;
    display: block;
}

.block-20 img {
height: 250px;
}


.row2 {
    display: grid;
    grid-template-columns: repeat(4, 1fr); /* 한 줄에 4개의 열을 생성 */
    gap: 20px; /* 열 사이의 간격 설정 */
    margin: 0 auto;
    max-width: 1450px;
    background-color: white;
}


/* 컨텐츠 부분*/
.content-entry {
background: #fff;
-webkit-box-shadow: 0px 10px 18px -8px rgba(0, 0, 0, 0.1);
-moz-box-shadow: 0px 10px 18px -8px rgba(0, 0, 0, 0.1);
box-shadow: 0px 10px 18px -8px rgba(0, 0, 0, 0.1); 
width: 300px}
.content-entry .text {
  position: relative;
  border-top: 0;
  border-radius: 2px; }
.content-entry .text .tag {
  color: #b3b3b3; }
.content-entry .text .heading {
  font-size: 18px;
  margin-bottom: 16px;
  font-weight: 400; 
  cursor: pointer;}
.content-entry .text .heading a {
  color: #000000; }
.content-entry .text .heading a:hover, .blog-entry .text .heading a:focus, .blog-entry .text .heading a:active {
  color:  #007bff; }
.content-entry .text .meta-chat {
  color:  #007bff; }
.content-entry .text .read {
  color: #000000;
  font-size: 14px; }
.content-entry .meta > div {
  display: inline-block;
  margin-right: 5px;
  margin-bottom: 5px;
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 1px; }
.content-entry .meta > div a {
    color: #999999;
    font-weight: 500; }
.content-entry .meta > div a:hover {
      color: #333333; }
.content-entry .btn-custom {
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: 500;
  color:  #007bff; }
.content-entry .meta .createdAt {
  width: 100%;
}

form {
  margin-bottom: 0px;
}
  

  .addpost-item {
margin: 20px; /* 원하는 간격으로 조정 */
width: 300px; 
cursor: pointer;
}

.addpost-item:hover {
    box-shadow: 0 8px 12px rgba(0, 0, 0, 0.2);
    transform: translateY(-3px);
}

/*페이지네이션 부분*/
.mt-5 {
  display: flex;
justify-content: center;
}

.block-27 {
margin-top: 50px; /* 페이지네이션과의 간격 조정 */
justify-items: center; /* 페이지네이션 가운데 정렬 */
}

.block-27 ul {
padding: 0;
margin: 0;
display: inline-block;
}

.block-27 ul li {
display: inline-block;
margin-bottom: 4px;
font-weight: 400;
margin-right: 5px; /* 페이지네이션 간격 조정 */
}

.block-27 ul li a,
.block-27 ul li span {
color: gray;
text-align: center;
display: inline-block;
width: 40px;
height: 40px;
line-height: 40px;
border-radius: 50%;
border: 2px solid #e6e6e6;
}

.block-27 ul li.active a,
.block-27 ul li.active span {
background: #007bff;
color: #fff;
border: 1px solid transparent;
}
.custom-button {
  font-family: 'Ownglyph_meetme-Rg';
  margin-top: -300px;
  margin-left: 1260px;
  /* 왼쪽 여백을 auto로 설정하여 오른쪽으로 정렬 */
  padding: 10px 20px;
  font-size: 16px;
  background-color: #8d8d8d;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

/* 모달창 */
.modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
	  margin: 0;
	  padding: 0;
    background: rgba(0, 0, 0, 0.5);
}
.modal.show {
  display: block;
}
.preview {
	position: fixed;
	margin: 0.5rem auto; 
	margin-top: 100px;
	pointer-events: none;
}
.preview {
	position: relative;
	display: flex;
	flex-direction: column;
	pointer-events: auto;
	background-color: #fff;
	background-clip: padding-box;
	border: 1px solid rgba(0, 0, 0, 0.2);
	border-radius: 0.3rem;
	outline: 0;
}
.preview {
  display: flex;
  align-items: center;
  padding: 0.5rem;
  border-bottom: 1px solid #dee2e6;
  background-color: #f7f7f7;
}

.modal {
  display: flex;
  justify-content: center;
}

.btn-close {
  background-color: transparent;
  border: none;
  border-radius: 100%;
  padding: 0;
  cursor: pointer;
  width: 15px;
  height: 15px;
  transition: background-color 0.3s ease;
  position: absolute; /* preview를 기준으로 절대 위치 지정 */
  top: 10px; /* preview 상단에서의 위치 */
  right: 10px; /* preview 우측에서의 위치 */
}


.btn-close:hover {
  background-color: rgba(0, 0, 0, 0.1);
}

@media (max-width: 1200px) {
  .addpost-item {
    width: 33.33%; /* 세 개의 요소가 한 줄에 배치될 수 있도록 너비 조정 */
  }
}

@media (max-width: 768px) {
  .addpost-item {
    width: 50%; /* 두 개의 요소가 한 줄에 배치될 수 있도록 너비 조정 */
  }
}

@media (max-width: 576px) {
  .addpost-item {
    width: 100%; /* 한 개의 요소만 한 줄에 배치될 수 있도록 너비 조정 */
  }
}


.btn-close {
  background-color: transparent;
  border: none;
  border-radius: 100%;
  padding: 0;
  cursor: pointer;
  width: 15px;
  height: 15px;
  transition: background-color 0.3s ease;
  align-self: flex-end;
}
.btn-close:hover {
  background-color: rgba(0, 0, 0, 0.1);
}

/* @media screen and (min-width: 1440px) and (max-width: 2560px) {
  .content-entry {
    width: 22vw;
  }
}
@media screen and (min-width: 1024px) and (max-width: 1440px) { 
  .content-entry {
    width: 19vw;
  }
}
@media screen and (min-width: 768px) and (max-width: 1024px) {

}
@media (min-width: 768px) {
    .card-columns {
        column-count: 3;
        column-width: 80%;
    }
} */
.active {
    background-color: #61bffd;
    color: #fff !important;
    border: 1px solid transparent;
}
</style>



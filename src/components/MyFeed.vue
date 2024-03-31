<template>
	<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
	<body id="top">
		<section id="MyBanner">
			<div class="inner"></div>
		</section>
		<!-- Page Content-->
		<section class="py-5">
			<div class="container px-5 my-5" style="box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);">
				<div class="row gx-5">
					<div class="col-lg-3" id="MyDetail">
						<div class="d-flex align-items-center mt-lg-5 mb-4" id="mt-lg-5">
							<img class="img-fluid rounded-circle" :src="this.user.imgPath" alt="..." id="profil-img"/>
							<div class="Myprofil">
								<div class="" id="myname">
									<div class="fw-bold">{{this.user.name}}</div>
									<div class="text-muted d-flex align-items-center">{{this.user.email}}</div>
								</div>
								<div id="contentCount">
									<div class="text-muted" id="PageCount">게시물 수 : {{ this.user.boardCount }}</div><hr>
									<div class="text-muted" id="PageCount">댓글 수 : {{ this.user.commentCount }}</div><hr>
									<div class="text-muted" id="PageCount">기록일지 수 : {{  this.user.diaryCount }}</div><hr>
								</div>
							</div>
                            <div class="edit-button">
                                <button class="edit-butto my-1"  @click="goToBack()" title="뒤로가기">
                                    <img src="../assets/images/back-page.png">
                                </button>
                            </div>
						</div>
					</div>
					
					<div class="col-lg-9" id="content">
						<article>
								<section>
									<div class="card board-container">
										<ul class="tabs">
											<li class="tab-link" :class="{ 'current': currentTab === tab.id }" v-for="tab in tabs" :key="tab.id" @click="changeTab(tab.id)" :data-tab="tab.id">{{ tab.content }}</li>
										</ul>
											<div class="board-header">
                                                <h4 class="header-number">번호</h4>
												<h4 class="header-title">{{ tabHeader.header_1 }}</h4>
												<h4 class="header-name">{{ tabHeader.header_2 }}</h4>
												<h4 class="header-view">{{ tabHeader.header_3 }}</h4>
												<h4 class="header-date">{{ tabHeader.header_4 }}</h4>
											</div>
											<div class="board-content">
												<div class="board-item" v-for="(post, n) in currentPagePosts" :key="post.id" >
													<div class="item-header px-3">
                                                        <h5 class="item-number">{{ calculatePostNumber(n) }}</h5>
														<div class="item-content">
                                                            <h5 v-if="currentTab != 'tab-4'" ><a href="#" @click.prevent="goToEdit(post.id)">{{ post.title }}</a></h5>
                                                            <h5 v-if="currentTab === 'tab-4'" ><a href="#" @click.prevent="goToEdit(post.boardId)">{{ post.content }}</a></h5>
                                                        </div>
                                                        <h5 class="item-viewCount"><a href="#">{{ post.viewCount }}</a></h5>
                                                        <div class="item-content2">
                                                            <h5><a href="#">{{ post.likeCount }}</a></h5>    
														    <h5><a href="#" @click.prevent="goToPet(post.petId)">{{ post.petName }}</a></h5>
                                                        </div>
														<span>{{ post.createdAt ? post.createdAt.split('T')[0] : '' }}</span>
													</div>
													<hr class="item-divider">
												</div>
											</div>
											<div class="pagination">
												<button class="page-link" @click="goToPreviousPage">«</button>
												<button class="page-link" v-for="n in displayedPages" :key="n" :class="{ 'current-page-link': n === currentPage }" @click="goToPage(n)">{{ n }}</button>
												<button class="page-link" @click="goToNextPage">»</button>
											</div>
									</div>
								</section>
						</article>
					</div>
				</div>
			</div>
		</section>
	</body>	 
</template>

<script>
	export default {
	  data() {
		return {
		  posts: [ ],
		  user:{ },
		  maxpage : 5,
          currentPage: 1, // 현재 페이지를 추적하는 데이터 추가
          itemsPerPage: 10, // 페이지당 아이템 수
		  currentTab: 'tab-1',
      	  tabs: [
        	{ id: 'tab-1', content: '마이 피드' },
        	{ id: 'tab-2', content: '마이 Q&A' },
        	{ id: 'tab-3', content: '좋아요 한 글' },
			{ id: 'tab-4', content: '내 댓글' },
			{ id: 'tab-5', content: '내 기록일지' }
      		],
          tabHeader: {
            header_1: '제목',
            header_2: '조회수',
            header_3: '좋아요',
            header_4: '날짜'
            }, 
          page : 1
		};
	  },
	  computed: {
            // 현재 페이지의 시작 인덱스
            startIndex() {
                return (this.currentPage - 1) * this.itemsPerPage;
            },
            // 현재 페이지의 끝 인덱스
            endIndex() {
                return Math.min(this.currentPage * this.itemsPerPage, this.posts.length);
            },
            // 현재 페이지에 표시할 데이터
            currentPagePosts() {
                return this.posts.slice(this.startIndex, this.endIndex);
            },
            // 전체 페이지 개수
            pageCount() {
                return Math.ceil(this.posts.length / this.itemsPerPage);
            },
            displayedPages() {

                const totalPages = Math.ceil(this.maxPage / this.itemsPerPage);
                let startPage;
                let endPage;
                if (this.currentPage <= 3) {
                    startPage = 1;
                    endPage = Math.min(totalPages, 5);
                } else if (this.currentPage >= totalPages - 2) {
                    startPage = Math.max(1, totalPages - 4);
                    endPage = totalPages;
                } else {
                    startPage = this.currentPage - 2;
                    endPage = this.currentPage + 2;
                }
                const displayedPages = [];
                for (let i = startPage; i <= endPage; i++) {
                    displayedPages.push(i);
                }
                return displayedPages;
                
            },
        },
	  mounted() {
	    if (!this.$cookies.get("id")) {
	    	alert("로그인이 필요합니다.");
	    	this.$router.push('/login');
	    	return;
	    }
		this.axios.get(`/api/myinfo/${this.$cookies.get("id")}`).then((res) => {
			this.user = res.data;
            this.axios.get(`/api/myinfo/img/${this.$cookies.get("id")}`).then((res) => {
                this.user.imgPath = res.data;
            });
		}).catch();
		this.axios.get(`/api/free/getMyBoard/${this.$cookies.get('id')}`,{
			params: { 
        		subject: 0,
        		page: this.page,
        }}).then((res) => {
				this.posts = res.data;
                if(this.posts)
                    this.maxpage = this.posts[0].totalRowCount;

		}).catch();
	  },

	  methods: {
    	changeTab(tabId) {
      		this.currentTab = tabId;
            this.currentPage = 1;
			switch ( this.currentTab ) {
				case "tab-1":
                    this.tabHeader = {
                            header_1: '제목',
                            header_2: '조회수',
                            header_3: '좋아요',
                            header_4: '날짜'
                        };
					this.axios.get(`/api/free/getMyBoard/${this.$cookies.get('id')}`,{
						params: { 
        				    subject: 0,
        				    page: this.page,
        				}
					}).then((res) => {
						this.posts = res.data;
                        // if(this.posts)
                        //     this.posts[0].totalRowCount;
					}).catch();
					break; 
				case "tab-2":
                    this.tabHeader = {
                            header_1: '제목',
                            header_2: '조회수',
                            header_3: '좋아요',
                            header_4: '날짜'
                        };
                    this.axios.get(`/api/free/getMyBoard/${this.$cookies.get('id')}`,{
                            params: { 
                                subject: 1,
                                page: this.page,
                            }
                        }).then((res) => {
                            this.posts = res.data;
                        }).catch();
					break; 
				case "tab-3":
                    this.tabHeader = {
                            header_1: '제목',
                            header_2: '조회수',
                            header_3: '좋아요',
                            header_4: '날짜'
                        };
					this.axios.get(`/api/free/getMyLike/${this.$cookies.get('id')}`,{
                        params: {
                            page: this.page,
                        }
                    }).then((res) => {
                        this.posts = res.data;
                        if(this.posts.length > 0)
                            this.maxpage = this.posts[0].totalRowCount;
                    });
					break; 	
				case "tab-4":
                    this.tabHeader = {
                        header_1: '내용',
                        header_2: '',
                        header_3: '좋아요',
                        header_4: '날짜'
                    };
				    this.axios.get(`/api/comment/mycomment/${this.$cookies.get('id')}`
                        ).then((res) => {
                            this.posts = res.data;
                        }).catch();
					break; 	
				case "tab-5":
                    this.tabHeader = {
                        header_1: '제목',
                        header_2: '',
                        header_3: '펫이름',
                        header_4: '날짜'
                    };
					this.axios.get(`/api/myinfo/diary/${this.$cookies.get('id')}`).then((res) =>{
						this.posts = res.data;
						this.maxpage = Math.ceil(this.posts.length / this.itemsPerPage);
					}).catch();
					break; 	
			}
    	},
		filteredPosts(tabId) {
      		return this.posts.filter(post => {
        		return post.tab === tabId;
      		});
    	},

		goToPage(n) {
            this.currentPage = n;
        },
        goToBack() {
            this.$router.push('/mypage');
        },
        goToEdit(id){
			switch ( this.currentTab ) {
                case "tab-1": 
                    this.$cookies.set('postId', id);
			        this.$router.push('/freeboard3');
                    break;
                case "tab-2": 
                    this.$cookies.set('postId', id);
			        this.$router.push('/qnaboard');
                    break;
                case "tab-3": 
                    if ( true ) {
                        this.$cookies.set('postId', id);
                        this.$router.push('/freeboard3');
                    }
                    break;
                case "tab-4": 
                    if ( this.posts.subject ) {
                        this.$cookies.set('postId', id);
                        this.$router.push('/qnaboard');
                    } else {
                        this.$cookies.set('postId', id);
                        this.$router.push('/freeboard3');
                    }
                    break;
                case "tab-5": 
                    this.$cookies.set('diaryId', id);
			        this.$router.push('/carousel');
                    break;
            }
		},
        goToPet(id){
            this.$router.push({ path: '/petdetail', query: { petId: id } });
		},
        // 이전 페이지로 이동하는 함수
        goToPreviousPage() {
            if (this.currentPage > 1) {
                this.currentPage--;
            }
        },
        // 다음 페이지로 이동하는 함수
        goToNextPage() {
            if (this.currentPage < this.pageCount) {
                this.currentPage++;
            }
        },
        calculatePostNumber(index) {
            // 현재 페이지의 첫 번째 게시물의 번호 계산
            return this.posts.length - ((this.currentPage - 1) * this.itemsPerPage + index);

        },
  		}
  	}
</script>

	
<style scoped>
	@import '../assets/css/skel.css';
	@import '../assets/css/style.css';
	@import '../assets/css/style-xlarge.css';
/* Font */

@import url('https://fonts.googleapis.com/css2?family=Black+Han+Sans&display=swap');

@font-face {
    font-family: 'KBO-Dia-Gothic_bold';
    src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2304-2@1.0/KBO-Dia-Gothic_bold.woff') format('woff');
    font-weight: 700;
    font-style: normal;
}

@font-face {
    font-family: 'SUITE-Regular';
    src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2304-2@1.0/SUITE-Regular.woff2') format('woff2');
    font-weight: 400;
    font-style: normal;
}

@font-face {
    font-family: 'Ownglyph_meetme-Rg';
    src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2402_1@1.0/Ownglyph_meetme-Rg.woff2') format('woff2');
    font-weight: normal;
    font-style: normal;
}

/* MyBanner */

#MyBanner {
    background-image: url(../assets/images/MyPage.jpg);
    background-size: cover;
    background-position: center ;
    background-repeat: no-repeat;
    color: #d8d8d8;
    padding: 14em 0;
    text-align: center;
    position: relative;
    height: 200px;
}

#MyBanner:before {
    content: '';
      position: absolute;
      left: 50%;
      top: 0;
     transform: translateX(-50%); /* 가운데로 이동 */
      width: 100%; /* 너비를 60%로 설정 */
      height: 100%;
      background: rgba(64, 72, 80, 0.25);			
}

#MyBanner .inner {
    position: relative;
    z-index: 1;
}

#MyBanner .inner :last-child {
    margin-bottom: 0;
}

/* content */

.gx-5 {
    flex-direction: column;
    align-items: center;
	position: relative;
	top: -100px;
}

.py-5 {
    background-color: #fafafa;
    margin-bottom: -330px;
}

.px-5 {
    position: relative;
    top: -330px;
    background-color: white;
    padding: 30px;
    border-radius: 5px;
    min-width: 950px;
	padding-bottom: 0; 
}

#content {
    width: 95%;
}

.myPetList {
    margin-bottom: 5rem;
}

#PostHeader {
    text-align: left;
    margin-top: 10px;
    /* margin-bottom: 0.5rem!important; */
}

.mb-1 {
    display: inline;
    font-family: 'KBO-Dia-Gothic_bold'; 
    letter-spacing: normal;
    margin-left: 15px;
    font-size: 35px;
    color: #1897f1;
}

.mb-2 {
    display: inline;
    margin-left: 10px;
    font-family: 'SUITE-Regular';
    font-weight: bold;
}


/* board */

   .board-container {
    background-color: #fff; /* White background */
    border: 1px solid #DEE2E6; /* Light grey border */
    border-radius: 4px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
    /* padding: 1rem; */
    margin: auto;
  }

  .board-header {
    font-family: 'Ownglyph_meetme-Rg';
    background-color: aliceblue;
    display: flex;
    justify-content: space-between;
    padding: 0.8rem 2.5rem 0.5rem 2.5rem;
    border-bottom: 2px solid #DEE2E6;
  }

  .header-number {
    width: 6%;
  }

  .header-title {
    width: 53%;
  }

  .header-name {
    width: 10%;
  }
  
  .header-view {
    width: 10%;
  }

  .header-date {
    width: 17%;
  }
  .board-content {
    margin-bottom: 25px;
  }

  .board-item {
    background-color: white;
    padding: 0.5rem 0;
    margin-left: 1rem;
    margin-right: 1rem;
    border-bottom: 1px solid #DEE2E6; /* Light grey border for each item */
  }

  .item-header {
    font-family: 'Ownglyph_meetme-Rg';
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .item-header > * {
    margin: 0;
    font-size: 18px;
    color: #495057; /* Dark grey color for text */
    text-overflow:ellipsis;
    white-space:nowrap; 
    overflow:hidden;  
}

  .item-header img {
    margin-left: 5px;
    width: 22px;
  }
  
  .item-header > .item-number {
    width: 7%;
  }

  .item-header > .item-content {
    width: 53%;
  }
  
  .item-header > .item-viewCount {
    width: 11%;
  }
  
  .item-header > .item-content2 {
    width: 11%;
  }

  .item-header > * > p a {
    color: inherit;
    text-decoration: none;
  }

  .item-header span {
    width: 18%;
  }

  .item-details {
    text-align: right;
  }

  .item-author {
    font-size: 0.875rem;
    color: #6C757D; /* Light grey color for author */
  }

  .item-date {
    font-size: 0.875rem;
    color: #ADB5BD; /* Even lighter grey for date */
    margin-left: 0.5rem;
  }

  .item-content p {
    margin: 0;
    color: #495057; /* Match the title color */
  }
  
  .item-divider {
    display: none; /* Remove the hr as we're using border-bottom for each item */
  }

  /* Board_Pagination*/

.pagination {
    display: flex;
    justify-content: center;
    gap: 2px;
	margin-bottom: 20px;
}

.page-link {
    font-family: 'Ownglyph_meetme-Rg';
    width: 36px;
    border: none;
    background-color: transparent;
    color: #333;
    padding: 5px 10px;
    border-radius: 50%;
    cursor: pointer;
}

.page-link:hover {
    background-color: #f0f0f0;
}

.current-page-link {
    border: 1px solid #e0e0e0;
}

/* sideBar */

#MyDetail {
    width: 90%;    
	background-color: white;
    border: 3px solid #f0f0f0;
    border-radius: 10px;
	margin-bottom: 2rem;
}

#mt-lg-5 {
    /* div 내에 중앙 정렬 */
    display : flex;
    justify-content: left;
    align-items : center;
    /* 여러 요소 세로 정렬 */
    flex-direction: row;
	margin-left: 20px;
	margin-top: 0px !important;
	margin-bottom: 0px !important;
}

#profil-img {
	margin: 10px;
    /* border 테두리 지정 */
    border: 5px;
    border-style: solid;
    border-color: #BDE3FF;
	max-width: 100%;
    max-height: 100%;
    height: 150px;
    width: 150px;
    object-fit: contain;	
}

.Myprofil {
    display: inline;
	margin-left: 50px;
}

#myname {
	display: flex;
    margin-bottom: 1rem !important;
}

#myname .fw-bold {
    font-family: 'Ownglyph_meetme-Rg';
    font-size: 28px;
	margin-right: 1rem;
}

#contentCount {
	display: flex;
    text-align: left;
    width: 100%;
}

#PageCount {
    color: #929292 !important;
    font-family: 'Ownglyph_meetme-Rg';
    font-size: 20px;
    border-bottom: 0rem !important;
	margin-right: 1em;
}

hr {
    margin-top: 0rem !important;
    /* size: 0.1px;
    background-color:#85ccff;
    height: 2px; */
}

/* tab */

ul.tabs {
    margin: 0;
    padding: 0;
    list-style: none;
	display: flex;
}

ul.tabs li {
	font-family: 'Ownglyph_meetme-Rg';
	font-size: 20px;
    background: none;
    color: #222;
	background-color: rgb(245, 250, 255);
    display: inline-block;
    border-bottom: 1px solid #DEE2E6;
	border-right: 1px solid #DEE2E6;
    padding: 10px 15px;
    cursor: pointer;
	flex-grow: 1;
}
  
ul.tabs li.current {
    background: aliceblue;
    color: #222;
	border-bottom: 0px solid aliceblue;
}

.edit-button {
    margin-left: auto;
}
  
.edit-butto {
    background-color: #fff;
    margin: 2px;
    padding: 15px;
    border: 5px solid #BDE3FF;
    border-radius: 20px;
}
</style>

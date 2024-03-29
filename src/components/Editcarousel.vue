<template>
  <section>
      <div class="main">
          <div id="app">
            <div class="background-image">
              <div class="card">
                <form v-on:submit.prevent='postData'>
                  <div class="header">
                      <h1>{{ this.name }}의 집사일기</h1>
                      <div class="emoji-container">
                          <img src="../assets/images/cogi.png" alt="Emoji 1">
                          <img src="../assets/images/cute.png" alt="Emoji 2">
                          <img src="../assets/images/bdog.png" alt="Emoji 3">
                          <img src="../assets/images/gdog.png" alt="Emoji 4">
                          <b>수정하개!</b>
                      </div>
                  </div>
                  <div class="option animal-as">
                    <label>나의 반려동물</label>
                    <div class="select-menu">
                      <h1 class="ChangepetName">{{ this.name }}</h1>
                  </div>
            </div>
            <div class="option-container">
                    <div class="option mood-as">
                      <label>오늘의 기분</label>
                      <input type="text" spellcheck="false" placeholder="Good~" v-model="mood"/>
                    </div>
                    <div class="option save-as">
                      <label>오늘의 날씨</label>
                      <div class="select-menu">
                        <select v-model="weather">
                          <option value="sunny">햇빛 쨍쨍</option>
                          <option value="cloudy">흐림</option>
                          <option value="wind">바람 쌩쌩</option>
                          <option value="rain">비 주룩주룩</option>
                          <option value="snow">눈 펑펑</option>
                        </select>
                      </div>
                    </div>
                  </div>
                  
                  <div class="date">
                      <input class="title" placeholder="제목을 입력해주세요." type="text" name="text" style="border: none; background: transparent;" v-model="title">
                      <hr>
                      <div class="file-options">
                        <div class="slide-container">

                          <div class="slide">
                          <div v-for="(slide, index) of slides" :key="slide.imgId" class="image-container">
                            <img :src="slide.imgPath" :alt="'올린 이미지 ' + (index + 1)" class="uploaded-image"
                            style="border-color: black;  width:65%; height: 15vh;"/>
                            <button @click.prevent="slidedelete(slide)" class="delete-button">X</button>
                          </div>
                        </div>

                          <div class="slide">
                            <div v-for="(file1, index) of fileList" :key="index" class="image-container">
                              <img :src="imageUploaded[index]" alt="추가한 이미지" class="uploaded-image" 
                              style="border-color: black;  width:65%; height: 15vh;"/>
                              <button @click.prevent="filedelete(file1)" class="delete-button">X</button>
                            </div>
                          </div>

                        </div>
                        </div>
                      <div class="dateCalendar">
                          <span> 날짜 :  {{ selectedDate }}</span>
                          <a href="#" @click="toggleCalendar"><img src="../assets/images/calendar.png" alt="Calendar"></a>
                          <input class="date1" v-if="showCalendar" type="date" @change="selectDate($event.target.value)">
                      </div>
                      </div>
                  <div class="content">
                      <textarea placeholder="오늘의 집사 일기를 입력해주세요." v-model="content">{{ this.content }}</textarea>
                  </div>
                  <div class="card-body-footer">
                  <button type="submit" class="register-btn" onclick = "location.href = '/mypage'">수정 취소하기</button>
                  <button class="list-btn">수정하기</button>
                  <label for="file-upload" class="custom-file-upload" @click.prevent="openFileInput">
                      파일 선택
                  </label>
                  <input type="file" id="fileInput"  ref="image" accept="image/*" multiple style="display: none;" @change="previewImages">
                  </div>
                </form>
              </div>
            </div>
          </div>
      </div>
  </section>
  </template>
  
  <script>
  
  export default {
      data() {
      return { 
        slides: [ ],
        imageList: [],
        imageUrl: null,
        fileList: [],
        file: "",
        showCalendar: false,
        selectedDate: '',
        pets: {},
        petSelect: "",
        title: "",
        content: "",
        mood: "",
        weather: "sunny",
        name: "",
        createdAt: "",
        deleteList : [],
        original : {}
      };
  },
  methods: {
    weatherchange(weather){
      if(weather == "sunny"){
        return "햇빛 쨍쨍"
      }
      if(weather == "cloudy"){
        return "흐림"
      }
      if(weather == "wind"){
        return "바람 쌩쌩"
      }
      if(weather == "rain"){
        return "비 주룩주룩"
      }
      if(weather == "snow"){
        return "눈 펑펑"
      }
    },
      openFileInput(){
      const fileInput = document.getElementById('fileInput');
          fileInput.click();
      },
      previewImages(event) {
          const files = event.target.files;
          this.imageUploaded=[];
          this.fileList = files;
          this.fileList = Array.from(event.target.files);
          for(let file1 of this.fileList){
              this.imageUploaded.push(URL.createObjectURL(file1));
          }

          console.log(files);
          // 파일 미리보기 로직
      },

      filedelete(file1) {
      this.fileList = this.fileList.filter(file => file !== file1);
      },
      slidedelete(slide) {
        this.deleteList.push(slide.imgId);
        this.slides = this.slides.filter(s => s !== slide);
      },

    toggleCalendar() {
        this.showCalendar = !this.showCalendar;
      },
      selectDate(date) {
          this.selectedDate = date;
          this.showCalendar = false;
      },
      change(petname){
        console.log(petname);
        this.name = petname;
      },
      changePet() {
      const selectedPetId = this.petSelect;
      console.log('Selected pet ID:', selectedPetId);
      console.log('Pets:', this.pets);
      const selectedPet = this.pets.find(pet => pet.id === selectedPetId);
      console.log('Selected pet:', selectedPet);
      if (selectedPet) {
        this.name = selectedPet.name;
      }
    },
  
  postData() {
    console.log(this.deleteList)
    if (this.deleteList.length > 0) {
      const idsString = this.deleteList.join(',');
      this.axios.delete(`/api/myinfo/delete?ids=${idsString}`)
        .then(()=> {
          console.log("이미지가 성공적으로 삭제되었습니다.");
        })
        .catch(error => {
          console.error("이미지 삭제 중 오류가 발생했습니다:", error);
        });
    }


  const data2 = {
      mood: this.mood,
      weather: this.weather,
      title: this.title,
      content: this.content,
      createdAt: this.selectedDate,
      id: this.$cookies.get('diaryId'),
      img: [],
      userId: this.original.userId,
      petId: this.original.petId
  };

  // Create FormData object
  const formData = new FormData();
  
  // Append each file in fileList array to FormData
  this.fileList.forEach((file) => {
      formData.append('image', file);

  });

  if(this.fileList.length > 0){

    this.axios.post(`/api/free/img`, formData, {
        headers: {
            'Content-Type': 'multipart/form-data'
        }
    }).then((res) => {
      console.log(res.data);
      console.log(data2);
      data2.img = res.data;
      console.log("성공");
      // console.log(this.imageList);
        this.axios.put(`/api/myinfo/edit/${this.diaryId}`, data2).then(() => this.$router.push('/carousel'))
          .catch(error => {
            // 실패 시 로직
            console.error('데이터 전송 실패:', error);
          });
    }).catch(error => {
        console.error("이미지 업로드 중 오류가 발생했습니다:", error);
    });
}

  // this.axios.put(`/api/myinfo/edit/${this.diaryId}`, data2).then(() =>
  //         this.$router.push('/carousel')).catch(error => {
  //             console.log("데이터 보내기 실패")
  //         });
  }

},
  mounted() {
    this.axios.get(`/api/myinfo/DiaryImages/${this.$cookies.get('diaryId')}`).then((res) => {
    this.slides= [];
    for(let a of res.data){
      this.slides.push(a); 
    }
    this.diaryId = this.$cookies.get('diaryId');
    this.axios.get(`/api/myinfo/select/${this.diaryId}`).then(res => {
    console.log(this.diaryId)
    console.log(res.data);
    this.original = res.data;
    this.content = res.data.content;
    this.title = res.data.title;
    this.name = res.data.name;
    this.selectedDate = res.data.createdAt;
    this.mood = res.data.mood;
    this.weather = res.data.weather;
    console.log(this.weather)
    }).catch((error) => console.log(error))});
  }
}
  </script>
  
  <style scoped>
  .date1{
      border-radius: 10px;
      font-size: 18px;
  }
  
  .dateCalendar input[type="date"].show {
    display: block; /* this class will be toggled to show the calendar */}
  @font-face {
      font-family: 'EF_watermelonSalad';
      src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2210-EF@1.0/EF_watermelonSalad.woff2') format('woff2');
      font-weight: normal;
      font-style: normal;
  }
  @font-face {
      font-family: 'Ownglyph_meetme-Rg';
      src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_2402_1@1.0/Ownglyph_meetme-Rg.woff2') format('woff2');
      font-weight: normal;
      font-style: normal;
  }
  
  *{
    font-family: 'Ownglyph_meetme-Rg';
  }
  
  .date1 {
    margin-top: 0.4rem;
  }
  
  .option :where(input, .select-menu){
  height: 50px;
  padding: 0 13px;
  margin-top: 6px;
  border-radius: 5px;
  border: 1px solid #bfbfbf;
  }
  .option .select-menu select{
    height: 50px; /* select 요소의 높이 설정 */
    width: 100%; /* select 요소의 너비를 부모 요소에 맞게 설정 */
    background: none;
  }
  div.select-menu{
    margin: 2.4px 0px 0px;
    width: 60%;
  }
  .option .animal-as{
    margin: 2.4px 0px 8px;
    width: 60%;
  }
  .option.mood-as{
    margin: 0px 15px 0px;
    width: 40%;
  }
  .option.save-as{
    margin: 2.4px 10px 0px;
  }
  .option {
    /* 기존 스타일 유지 */
    display: flex;
    align-items: center; /* 세로 중앙 정렬 */
  }
  
  .option label {
    /* 기존 스타일 유지 */
    margin-right: 10px; /* 라벨과 선택 메뉴 사이의 간격 조정 */
    font-size: 18px;
  }
  
  .select-menu {
    /* 기존 스타일 유지 */
    width: 20%; /* 선택 메뉴의 너비 조정 */
  
  }
  
  input {
    margin-top: 0.4rem;
    background-color: #ffffff;
    font-family: 'Ownglyph_meetme-Rg';
  
  }
  /* .image{
      font-family: 'Ownglyph_meetme-Rg';
      padding: 10px 20px;
      margin-right: 10px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      background-color: #7ab7e0;
      color: white;
      font-family: 'Ownglyph_meetme-Rg';
      font-size: 20px;
  } */
  .custom-file-upload {
      /* 파일 선택 버튼의 스타일 지정 */
      font-family: 'Ownglyph_meetme-Rg';
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 10px 20px;
      margin-right: 10px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      background-color: #edf6fc;
      color: #7ab7e0;
      font-size: 20px;
      width: 25%;
      height: 100%;
      border: 2px solid #7ab7e0;
  }
  
  /* input[type="file"] 요소 숨기기 */
  .image {
      display: none;
  }
  
  
    #app {
      background-size: 100%;
      position: relative; 
      width: 100%; 
      height: 100%;
      margin-top: 95px;
      /* padding-left: 60px; 
      padding-right: 60px;  */
    }
  
    hr{
      margin-top: 0px;
      margin-bottom: 50px;
      color: #000;
    }

  b{
    font-size: small;
  }
  
.background-image {
/* background-image: url(../assets/images/수정배경화면.jpg); */
background-size: 100%;
background-repeat: no-repeat;
position: relative; /* This is necessary for absolute positioning of the paw borders */
width: 100%; /* Adjust as needed */
height: 100%;
}

.card {
  width: 50%;
  height: 100%;
  /* position: relative;
  top: 80px; */
  z-index: 1;
  margin: 70px auto 70px auto;
  padding: 20px;
  border: 1px solid #eee;
  border-radius: 10px;
  background: #fff;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  display: inline-block; /* 또는 inline-flex, inline 등 */
  height: auto; 
  min-height: calc(100% - 240px); 
}

    .title{
      font-family: 'Ownglyph_meetme-Rg';
      font-size: 20px;
    }
    .paw-border {
      margin-top: 120px;
    padding: 60px;
    position: absolute;
    top: 0;
    bottom: 0;
    display: flex;
    flex-direction: column;
    justify-content: center; /* This will center the content vertically */
  }
  
  .paw-border.left {
    left: 0;
  }
  .paw-border.right {
    right: 0;
  }
  
  .paw-print {
    /* Size as needed */
    width: 70px; /* Example size, adjust as needed */
    margin: 10px 0; /* This adds spacing between the paw prints */
  }
  .header {
    /* Style the header to align items as needed */
    display: flex;
    flex-direction: column;
    align-items: center;
  }
    .header h1 {
      margin: 0;
      font-size: 40px;
      font-family: 'Ownglyph_meetme-Rg';
    }
    .date span {
      font-size: 25px;
      font-family: 'Ownglyph_meetme-Rg';
    }
     
    .dateCalendar {
      display: flex;
      justify-content: center;
      align-items: center;
      top: 10px;
    }
   
    .dateCalendar > a {
      margin-left: 5px;
    }
  
    .emoji-container {
    /* Style to display emojis in a row */
    display: flex;
    justify-content: center;
    margin-top: 20px; /* Adjust as needed */
  }
  .emoji-container img {
    /* Style your emojis */
    margin: 0 5px; /* Adjust spacing between emojis */
    width: 50px; /* Adjust size as needed */
    height: auto;
  }
  .date {
    /* Style the date */
    margin-top: 20px; /* Adjust as needed */
    font-size: 25px; /* Adjust as needed */
  }
  .content textarea {
    /* Style the text area */
    width: 100%;
    height: 150px;
    margin-top: 20px; /* Adjust as needed */
    padding: 10px;
    border: 1px solid #eee;
    border-radius: 5px;
    font-family: 'Ownglyph_meetme-Rg';
    font-size: 23px;
    background-color: #fff; /* 배경색을 흰색으로 설정 */
    }
    .card-body-footer {
      display: flex;
      justify-content: space-between;
      margin-top: 20px; /* Adjust as needed */
      margin-bottom: 20px; /* 추가된 부분: 버튼 아래 공간 */
      height: 40px;
    }
  .register-btn, .list-btn {
    /* Style the buttons */
    padding: 10px 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    background-color: #7ab7e0;
    color: white;
    width: 48%; /* Adjust as needed to make both buttons equal width */
    font-family: 'Ownglyph_meetme-Rg';
    font-size: 20px;
  }
.card-body-footer button {
  padding: 10px 20px;
  margin-right: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  background-color: #7ab7e0;
  color: white;
  font-family: 'Ownglyph_meetme-Rg';
  font-size: 20px;
  height: 100%;
}
.option-container {
display: flex;
justify-content: space-between;
align-items: center;
margin-top: 20px; /* Adjust as needed */
}
  
  .option {
    /* Common styles for both options */
    width: 48%; /* Adjust as needed to make both options equal width */
  }
  
  .option label {
    /* Styles for labels */
    font-size: 17px;
  }
  
  .option :where(input, .select-menu) {
    /* Styles for input/select elements */
    height: 50px;
    padding: 0 13px;
    margin-top: 6px;
    border-radius: 5px;
    border: 1px solid #bfbfbf;
  }
  
  .option .select-menu select {
    /* Styles for select elements */
    height: 50px;
    background: none;
  }
  .ChangepetName{
      font-size: 20px;
      margin-top: 5%;
  }

  .content-wrapper {
display: flex;
}

.carousel-container {
flex: 1;
margin-top: 15%;
padding-left: 20px; /* Adjust spacing between card and carousel */
}

.uploaded-image {
border-color: black;
border: thick double #32a1ce;
width: 22%;
height: 15vh;
margin: 5px;
}



.image-container {
position: relative;
display: inline-block; /* 이미지들을 가로로 나란히 배치 */
margin-right: 10px; /* 이미지 사이의 간격 조정 */
margin-bottom: 10px; /* 아래쪽 여백 추가 */
width: calc(25% - 10px); /* 네 개의 이미지가 한 줄에 배치되도록 너비 설정 */
box-sizing: border-box; /* 패딩과 테두리를 너비에 포함 */
vertical-align: top; /* 이미지를 컨테이너의 상단에 정렬 */
}
.uploaded-image {
width: 100%;
height: auto; 
border: none; 
}

.delete-button {
position: absolute;
top: 5px; 
right: 5px; 
background: none;
border: none;
color: #7ab7e0;
font-size: 20px;
cursor: pointer;
}

body{
  height: 100%;
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
}
#footer{
  margin-top: 30%;
}

  @media screen and (max-width: 475px) {
    .wrapper{
      padding: 25px 18px 30px;
    }
    .wrapper :where(textarea, input, select, button, label){
      font-size: 16px!important;
    }
    .file-options .option{
      width: calc(100% / 2 - 5px);
    }
    .option :where(input, .select-menu){
      padding: 0 10px;
    }
    .wrapper .save-btn{
      padding: 15px 0;
    }
  }

  
    </style>
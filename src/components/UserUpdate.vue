<template>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
  <div class="card">
    <form @submit.prevent="userUpdate">
    <div class="top-container">
      <h1>회원 정보 수정</h1>
    </div>
      <div class="image_container">
        <div class="file-upload-buttons">
          <input type="file" id="image" accept="image/*" @change="setThumbnail($event)" style="display: none;"/>
        </div>
        <img :src="this.user.imgPath ? this.user.imgPath : defaultImage" alt="Thumbnail" class="thumbnail" @click="openFileInput" />
      </div>
      <div class="input_container">
        <div class="addPetName mb-3">
          <label class="m-2">닉네임</label>
          <input type="text" placeholder="이름을 수정하시겠습니까?" v-model="this.user.name"/>
        </div>
        <div class="addPetName mb-3">
          <label class="m-2">Email</label>
          <input type="email" placeholder="메일을 수정시겠습니까?" v-model="this.user.email"/>
        </div>
        <!-- <div class="addPetName mb-3">
          <label class="m-2">비밀번호</label>
          <input type="password" placeholder="비밀번호" v-model="password" />
        </div>
        <div class="addPetName mb-3">
          <label class="m-2">비밀번호 확인</label>
          <input type="password" placeholder="비밀번호 확인" v-model="passwordConfirm" />
        </div> -->
        <div class="addPetName mb-3">
          <label class="m-2">주소</label>
          <div class="searchAddress d-flex">
            <div class="d-flex align-items-center">
                <input class="postcode" type="text" v-model="postcode" placeholder="우편번호">
                <input class="postBtn" type="button" @click="execDaumPostcode" value="주소 찾기">
            </div>
            <input class="roadAddress" type="text" v-model="roadAddress" placeholder="도로명주소">
            <input class="jibunAddress" type="text" v-model="jibunAddress" placeholder="지번주소" style="display: none;">
            <span id="guide" style="color:#999;display:none"></span>
            <input class="detailAddress" type="text" v-model="detailAddress" placeholder="상세주소">
          </div>
        </div>
      </div>        
      <div class="submit-button-container">
        <button type="submit">수정하기</button>
      </div>
    </form>
  </div>    
</template>

<script>
export default {
  data() {
    return {
      user:{},
      imgPath: '',
      thumbnail: '',
      postcode: '',
      roadAddress: '',
      jibunAddress: '',
      detailAddress: '',
      defaultImage: require('../assets/images/plus.png'),
      // password: '',
      // passwordConfirm: '',
    };
  },

  methods: {
    // 'image' 클릭 이벤트 시 fileInput 작동
    openFileInput() {
     const fileInput = document.getElementById('image');
     fileInput.click();
    },

    // 썸네일 출력 
    setThumbnail(event) {
      // const reader = new FileReader();
      // const files = event.target.files;

      // this.fileList = files;
      // this.fileList = Array.from(event.target.files);

      // reader.onload = (event) => {
      //   this.user.imgPath = event.target.result; // Set the thumbnail URL
      // };
      // reader.readAsDataURL(files[0]);
      // this.fileList = Array.from(files);

      const files = event.target.files;
    if (files && files.length > 0) {
      const reader = new FileReader();
      reader.onload = (event) => {
        this.user.imgPath = event.target.result; // Set the thumbnail URL for preview
      };
      reader.readAsDataURL(files[0]); // Read the selected file as Data URL
      // Assign the selected file to fileList
      this.fileList = Array.from(files);
    } else {
      console.error("No files selected or unable to read the selected file.");
    }

    },

    // // 비밀번호 확인
    // isSubmitDisabled() {
    //   return this.password === this.passwordConfirm;
    // },

    userUpdate() {
      // if (!this.user.id) {
      //   alert("사용자 정보를 가져오는 중입니다. 잠시만 기다려주세요.");
      //   return;
      // }

      // if (this.password !== this.passwordConfirm) {
      //   alert("비밀번호와 비밀번호 확인이 일치하지 않습니다.");
      //   return;
      // }

      // if (this.password !== '') {
      //   this.user.password = this.password;
      // }

      if (!this.roadAddress && this.detailAddress) {
            alert("주소를 입력 후 상세주소를 입력해주세요.");
            return;
        }
      
      if (this.fileList && this.fileList.length > 0) {
        let formData = new FormData();
        formData.append('image', this.fileList[0]);
        this.axios.post(`/api/free/img`, formData, {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
          }).then((res) => {
            for(let s of res.data)
              this.imgPath = s;

              this.axios.put(`/api/myinfo`, {
                userId :  this.$cookies.get("id"),
                name : this.user.name, 
                email : this.user.email, 
                imgPath : this.imgPath,
                address : `${this.roadAddress}/${this.detailAddress}`,
              }).then(() => {
                this.$router.push('/mypage');
                }).catch();   
            }).catch();      
      } else {
        this.axios.put(`/api/myinfo`, {
          userId :  this.$cookies.get("id"),
          name : this.user.name, 
          email : this.user.email, 
          imgPath : this.user.imgPath,
          address : `${this.roadAddress}/${this.detailAddress}`,
        }).then(() => {
          this.$router.push('/mypage');
          }).catch();
      }
    },

    // 주소 찾기
    execDaumPostcode() {
      new daum.Postcode({
        oncomplete: (data) => {
          let roadAddr = data.roadAddress;
          let extraRoadAddr = '';

          if (data.bname !== '' && /[동|로|가]$/g.test(data.bname)) {
            extraRoadAddr += data.bname;
          }
          if (data.buildingName !== '' && data.apartment === 'Y') {
            extraRoadAddr += (extraRoadAddr !== '' ? ', ' + data.buildingName : data.buildingName);
          }
          if (extraRoadAddr !== '') {
            extraRoadAddr = ' (' + extraRoadAddr + ')';
          }

          this.postcode = data.zonecode;
          this.roadAddress = roadAddr;
          this.jibunAddress = data.jibunAddress;
          this.extraAddress = extraRoadAddr;

          let guideTextBox = document.getElementById("guide");
          if (data.autoRoadAddress) {
            let expRoadAddr = data.autoRoadAddress + extraRoadAddr;
            guideTextBox.innerHTML = '(예상 도로명 주소 : ' + expRoadAddr + ')';
            guideTextBox.style.display = 'block';
          } else if (data.autoJibunAddress) {
            let expJibunAddr = data.autoJibunAddress;
            guideTextBox.innerHTML = '(예상 지번 주소 : ' + expJibunAddr + ')';
            guideTextBox.style.display = 'block';
          } else {
            guideTextBox.innerHTML = '';
            guideTextBox.style.display = 'none';
          }
        }
      }).open();
    },
    // Daum 주소 검색 API
    loadDaumPostcodeScript() {
      const script = document.createElement('script');
      script.src = "//t1.daumcdn.net/mapjsapi/bundle/postcode/prod/postcode.v2.js";
      document.head.appendChild(script);
    },

  },
  mounted() {
    this.loadDaumPostcodeScript();

	  if (!this.$cookies.get("id")) {
	    alert("로그인이 필요합니다.");
	    this.$router.push('/login');
	    return;
	  }

		this.axios.get(`/api/myinfo/${this.$cookies.get("id")}`).then((res) => {
			this.user = res.data;

      // 회원 주소 정보를 도로명 / 상세 주소로 나누기
      if (this.user.address) {
        const addressParts = this.user.address.split('/');
        this.roadAddress = addressParts[0] || ''; // 첫 번째 부분은 도로명주소
        this.detailAddress = addressParts[1] || ''; // 두 번째 부분은 상세주소
      } else {
        this.roadAddress = '';
        this.detailAddress = '';
      }
      this.axios.get(`/api/myinfo/img/${this.$cookies.get("id")}`).then((res) => {
            this.user.imgPath = res.data;
        });
		}).catch(
        (error) => {
          console.error("사용자 정보를 가져오는 중 오류 발생:", error);
        });

    this.user.address
	}
};

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

form {
    margin: 2em 0 2em 0;
}

.card {
  border: 1px solid #e0e0e0;
  border-radius: 20px;
  padding: 20px;
  max-width: 1000px;
  max-height: 1500px;
  margin: auto;
  margin-top: 120px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.petUpdateLogo {
  max-width: 330px;
  height: auto;
}

label {
  display: block;
  font-size: 24px;
  color: #666;
  text-align: left;
  margin: 0 0px 10px 10px;

}

/* main */

.image_container {
  padding-top: 10px;
  padding-bottom: 10px;
}

.thumbnail {
  width: 200px;
  height: 200px;
  background-color: #ffffff;
  border: 5px solid rgb(206, 206, 206);
  border-radius: 50%;
  object-fit: contain;
}

.input_container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.input_container > * {
  width: 50%;
}

.form-select {
  appearance: none;
  background: rgba(144, 144, 144, 0.075);
  border-radius: 6px;
  border: none;
  border: solid 1px rgba(144, 144, 144, 0.25);
  display: block;
  outline: 0;
  padding: 0 1em;
  text-decoration: none;
}

.form-select:invalid {
  color: #bbbbbb; /* 선택되지 않았을 때의 색상 설정 */
} 

.form-check {
  text-align: left;
  padding-left: 0px;
}

input[type="checkbox"]:checked + label:before, input[type="radio"]:checked + label:before {
  background: #a9e0ff;
  border-color: #d7effd;
}

.searchAddress {
    display: flex;
    flex-direction: column;
}

.searchAddress > * {
    margin-bottom: 5px;
}

.postcode {
    display: inline;
    width: 50%;
}

.postBtn {
  color: #ffffff;
  background-color: #a7d3f3;
  border: 2px solid #b6e0ff;
}

.postBtn:hover {
  background-color: #4eaef3;
  border: 2px solid #a2d8ff;
}

.postBtn:active {
    background-color: #6ac1ff !important;
    border-color: #9ed2f8 !important;
}

/* category */

.category-section {
  display: flex;
  flex-direction: column; /* 아래로 정렬하도록 설정 */
  align-items: flex-start; /* 좌측 정렬 */
}

.category-section > label {
  margin-left: 5px;
  margin-bottom: 5px;
}

.category-buttons {
  display: flex;
  flex-wrap: wrap; /* 버튼들이 넘칠 경우 다음 줄로 넘어가도록 설정 */
}

.category-buttons button {
  color: white;
  margin-right: 10px;
  margin-bottom: 10px; /* 버튼 아래 간격 추가 */
  padding: 8px 12px;
  border: none;
  border-radius: 20px;
  background-color: #a7a7a7e5;
  cursor: pointer;
  transition: background-color 0.3s;
}

.category-buttons button:hover {
  background-color: #a9d3ff;
}

.category-buttons button.active {
  background-color: #89c2ff;
}
  
/* submit-button */

.submit-button-container {
  display: flex;
  justify-content: center;
  margin-top: 30px; /* 원하는 여백 설정 */
  margin-bottom: 30px;
}

.submit-button-container button {
  font-family: 'KBO-Dia-Gothic_bold';
  color: #ffffff;
  background-color: #a7d3f3;
  border: 2px solid #a7d3f3;
  border-radius: 20px;
  width: 50%;
  height: 40px;
}

.submit-button-container button:hover {
  background-color: #85ccff;
  border: 2px;
  border-style: solid;
  border-color: #9ed2f8;
}

.submit-button-container button:active {
    background-color: #6ac1ff !important;
    border-color: #9ed2f8 !important;
}
</style>
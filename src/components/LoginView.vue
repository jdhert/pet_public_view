<template>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH" crossorigin="anonymous">
  <section id="banner1">
    <div class="inner">
      <h2 style="font-family: 'continuous', self">Login</h2>
      <form class="loginForm" @submit.prevent="tryLogin">
        <input type="text" placeholder="email을 입력하세요" v-model="email" required>
        <input type="password" placeholder="비밀번호를 입력하세요" v-model="password" required>
        <input type="submit" value="로그인" style="background-color: #44608a;">
        <div class="form-check d-flex">
          <input class="form-check-input" type="checkbox" value="" id="flexCheckDefault" v-model="autologin" :checked="autologin == true">
          <label class="form-check-label" for="flexCheckDefault">자동 로그인</label>
        </div>
      </form>
      <div style=" display: flex; flex-wrap: wrap; align-items: center; padding: 5px;" class="my-2">
        <hr class="mx-2"><span>또는</span><hr class="mx-2">
      </div>
      <div class="socialLogin">
        <div>
          <div id = "naverIdLogin"></div>
            <!-- <button type="button" @click="logout">로그아웃</button> -->
          </div>
          <div id='G_OAuth_btn'></div>
          <!-- <button v-if="Object.keys(user).length != 0" @click="handleSignOut">Sign Out</button> -->
          <div>
            <a id="custom-login-btn" @click="kakaoLogin()">
              <img
                src="https://k.kakaocdn.net/14/dn/btroDszwNrM/I6efHub1SN5KCJqLm1Ovx1/o.jpg"
                width="170"
                alt="카카오 로그인 버튼"
              />
            </a>
            <!-- <div @click="kakaoLogout()">로그아웃</div> -->
        </div>
      </div>
      <div class="d-flex flex-column my-3">
        <span style="color: #8f8f8f;">계정이 없으신가요?<a href="/signup"> 회원가입</a></span>
        <span style="color: #8f8f8f;">비밀번호를 잃어버리셨나요?<a href="/findpassword"> 비밀번호 찾기</a></span>
      </div>
    </div>
  </section>
</template>

<script>
import VueJwtDecode from 'vue-jwt-decode'

export default {
    data() {
        return {
            email : "",
            password : "" ,
            user: {},
            naverLogin: null,
            autologin: false,
        }
    },
    methods : {
        tryLogin() {
          const autologinValue = this.autologin ? true : false;

          this.axios.post('/api/login', {
            email: this.email,
            password: this.password,
            autologin: autologinValue // autologin 값을 서버로 전송
          }).then((res) => {
            if(res.data != null){
              this.$store.commit('setLoginStatus', true);
              this.$store.commit('setUser', this.$cookies.get("id"));
              this.$router.push('/');
            } else {
              alert("로그인 실패!!");
            }
          }).catch(error => {
            console.error("로그인 시도 중 오류 발생:", error);
            alert("로그인 과정에 문제가 발생했습니다.");
          });
        },
        handleCallbackResponse(res){
          console.log('Encoded JWT ID token: ' + res.credential);
          let userObject = VueJwtDecode.decode(res.credential);
          console.log(userObject);
          this.user = userObject;
          this.axios.post('/api/login/social',{
            email : this.user.email,
            name : this.user.name,
            image : this.user.picture,
            password : ""
          }).then(() => {
              this.$store.commit('setLoginStatus', true);
              this.$store.commit('setUser', this.$cookies.get("id"));
              this.$router.push('/');
          }).catch();
          document.querySelector('#G_OAuth_btn').hidden = true;

        }, 
        handleSignOut(){
          this.user = {};
          document.querySelector('#G_OAuth_btn').hidden = false;
        },
        logout() {
            const accessToken = this.naverLogin.accessToken.accessToken;
            const url = `/oauth2.0/token?grant_type=delete&client_id=hxXy5oDjkXc5qMctxsf7&client_secret=aR2llTmtrs&access_token=${accessToken}&service_provider=NAVER`;
            this.axios.get(url).then((res) => {
                console.log(res.data)
            });
        },
        kakaoLogin() {
          window.Kakao.Auth.login({
            scope: "profile_image, account_email, profile_nickname",
            success: this.getKakaoAccount,
          });},getKakaoAccount() {
          window.Kakao.API.request({
            url: "/v2/user/me",
            success: (res) => {
            const kakao_account = res.kakao_account;
            const nickname = kakao_account.profile.nickname;
            const email = kakao_account.email;
            console.log("ninkname", nickname);
            console.log("email", email);
            console.log(kakao_account);
            //로그인처리구현
              this.axios.post('/api/login/social',{
                email : email,
                name : nickname,
                image : kakao_account.profile.profile_image_url,
                password : ""
              }).then(() => {
                this.$store.commit('setLoginStatus', true);
                  // 선택적으로 사용자 정보 저장 (응답에 따라)
                  this.$store.commit('setUser', this.$cookies.get("id"));
                  this.$router.push('/');
              }).catch();
            },
            fail: (error) => {
            console.log(error);
            },
            });
        }
    },
    mounted() {
          const id = this.$cookies.get("id");
        if (id) {
          // 이미 로그인된 상태이면 홈 페이지로 이동
          this.$router.push('/');
          alert("이미 로그인된 상태입니다.");
        }
        if (this.$cookies.get("autologin")) {
          this.autologin = this.$cookies.get("autologin");
        }

        this.$emit('forceRerender');
        let google = window.google;
        google.accounts.id.initialize({
          client_id: '659801624717-hr9ivvbekfr3s4gvt6p6jvpegk9hnn2r.apps.googleusercontent.com', 
          callback: this.handleCallbackResponse
        })
        google.accounts.id.renderButton(
          document.getElementById('G_OAuth_btn'), 
          { theme: 'outline', size: 'large' }
        )
        
        this.naverLogin = new window.naver.LoginWithNaverId({
            clientId: "hxXy5oDjkXc5qMctxsf7", // 개발자센터에 등록된 ClientID
            callbackUrl: "http://localhost:3000/login",   // 개발자센터에 등록한 callback Url
            isPopup: true, // 팝업을 통한 연동처리 여부
            loginButton:{
                color: "green", type:3, width : 190, height: 40},  // 로그인 버튼의 타입을 지정
        });

            // 설정 정보를 초기화하고 연동을 준비
        this.naverLogin.init();
 
        this.naverLogin.getLoginStatus((status) =>{
          if(status){
              console.log(status);
              console.log(this.naverLogin.user);

              // 필수적으로 받아야 하는 프로필 정보가 있다면 callback 처리 시점에 체크
              var email = this.naverLogin.user.getEmail();
              if(email == undefined || email ==null){
                  alert("이메일은 필수 정보입니다. 정보 제공을 동의해주세요.");
                  // 사용자 정보 재동의를 위하여 다시 네아로 동의 페이지로 이동함
                  this.naverLogin.reprompt();
                  return;
              }
              
              this.axios.post('/api/login/social',{
                  email : this.naverLogin.user.getEmail(),
                  name : this.naverLogin.user.getName(),
                  image : this.naverLogin.user.profile_image,
                  password : ""
                }).then((res) => {
                  window.close();
                  this.$store.commit('setLoginStatus', true);
                    // 선택적으로 사용자 정보 저장 (응답에 따라)
                  this.$store.commit('setUser',
                  this.$cookies.get("id"));
                  // this.$router.push('/');
                }).catch();
          }else{
              console.log("callback 처리에 실패했습니다.")
          }
        })
    }
}
</script>

<style scoped>

#custom-login-btn {
    cursor: pointer;
}

.socialLogin {
    display: flex;
}
.socialLogin > div {
    margin: 5px;
}

@font-face { 
  font-family: "continuous";
  src: url("../assets/fonts/continuous.ttf") format("truetype");
  font-weight: normal;
}

#banner1 {
		color: white;
		background-image: url("../assets/images/loginBanner.jpg");
		background-size: cover;
		background-position: center center;
		background-repeat: no-repeat;
		color: #d8d8d8;
		padding: 10em 0;
		text-align: center;
		position: relative;
		height: 100%;
	}

    #banner1 .inner {
      height: 36em;
        background-color: white;
        position: relative;
        z-index: 1;
        max-width: 700px; 
        margin: 0 auto; 
        display: flex; 
        flex-direction: column;            
        align-items: center; 
        justify-content: center;
        padding: 2em; 
        border-radius: 10px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }


    #banner1 .inner a {
        text-decoration: none;
    }

    #banner1 .inner hr {
        width: 245px;  
    }


		#banner1 h2, #banner1 h3, #banner1 h4, #banner1 h5, #banner1 h6 {
			color: #000000;
            font-weight: bold;
            text-transform: none;
		}
	

		#banner1 h2 {
			font-size: 3.5em;
			line-height: 1em;
			margin: 0 0 0.5em 0;
			padding: 0;
		}

		#banner1 p {
			font-size: 1.5em;
			margin-bottom: 2em;
		}

		#banner1 p a {
			color: #ffffff;
			text-decoration: none;
		}

.loginForm {
    align-content: center;
    margin-bottom: 0;
}

.loginForm > input {
    position: relative;
    width: 500px;
    margin: 10px;
}


.btn {
    border:none;
    border-radius:10px;
    color: rgb(233, 211, 211);
    
    background-color: rgb(255, 255, 41);
    align-content: center;
}

.form-check {
  justify-content: flex-start;
  margin-left: 2em;
}

input[type="checkbox"]:checked + label:before, input[type="radio"]:checked + label:before {
    background: #cbecff;
    border-color: #e6e6e6;
    color: #ffffff;
    content: '\f00c';
}

</style>
<template>
<body>
  <header class="banner">
      <h1 class="banner-title">비만 계산기</h1>
      <img src="../assets/images/banner2.png" alt="Banner" class="banner-image">
      <br>
    </header>
    <section class="hero-section">
    </section>
    <section class="shop-section">
      <div class="shop-images">
        <div class="shop-link">
          <img src="../assets/images/비만a.png" alt="card">
          <p class="step">1 단계</p>
        </div>
        <div class="shop-link">
          <img src="../assets/images/비만b.png" alt="card">
          <p class="step">2 단계</p>
        </div>
        <div class="shop-link">
          <img src="../assets/images/비만c.png" alt="card">
          <p class="step">3 단계</p>
        </div>
        <div class="shop-link">
          <img src="../assets/images/비만d.png" alt="card">
          <p class="step">4 단계</p>
        </div>
        <div class="shop-link">
          <img src="../assets/images/비만e.png" alt="card">
          <p class="step">5 단계</p>
        </div>
        <div class="shop-link">
          <img src="../assets/images/비만1-1.png" alt="card">
          <p class="step">1 단계</p>
        </div>
        <div class="shop-link">
          <img src="../assets/images/비만2-1.png" alt="card">
          <p class="step">2 단계</p>
        </div>
        <div class="shop-link">
          <img src="../assets/images/비만3-1.png" alt="card">
          <p class="step">3 단계</p>
        </div>
        <div class="shop-link">
          <img src="../assets/images/비만4-1.png" alt="card">
          <p class="step">4 단계</p>
        </div>
        <div class="shop-link">
          <img src="../assets/images/비만5-1.png" alt="card">
          <p class="step">5 단계</p>
        </div>
     
      </div>
      <form @submit.prevent="handleSubmit">
      <div class="submit">
        <select class="input-text" v-model="selectedPet">
            <option value="dog">강아지</option>
            <option value="cat">고양이</option>
          </select>
          <select v-model="selectedOption1" class="input-text">
            <option :value="option.value" v-for="option in options1" :key="option.value">{{ option.label }}</option>
          </select>
          <select v-model="selectedOption2" class="input-text" @change="handleCatChange">
            <option :value="option.value" v-for="option in options2" :key="option.value">{{ option.label }}</option>
          </select>
          <div class="input-container">
            <input type="text" class="petweight" v-model="inputValue" placeholder="5" @input="validateInput">
          </div>
          <div v-if="!selectedPet || !selectedOption1 || !selectedOption2 || !inputValue" class="error-message">모든 값을 선택해주세요.</div>
        <input type="submit" class="submit-button" value="결과보기">
      </div>
    </form>
    </section>
    <footer>
      <div class="footer-items">
        <h1 class="result">반려동물 비만도 결과</h1>
        <div class="wrapper">
          <h4>우리 아이는</h4>
          <br>
          <h1 class="box-fat">{{ resultText }}</h1>
        </div>
      </div>
      <a href="#" class="footer-title">
        Back to top
      </a>
    </footer>
</body>
</template>
<script>
export default {
  data() {
    return {
      inputValue: '',
      selectedPet: 'dog',
      inputValue: '', 
      resultText: '', 
      selectedOption1: '1', 
      selectedOption2: '3', 
      options1: [ 
        { value: '1', label: '1 단계' },
        { value: '3', label: '2 단계' },
        { value: '5', label: '3 단계' },
        { value: '7', label: '4 단계' },
        { value: '9', label: '5 단계' },
      ],
      options2: [ 
        { value: '2.5', label: '4개월 이하' },
        { value: '2.5', label: '5 ~ 12 개월' },
        { value: '1.2', label: '중성화' },
        { value: '1.4', label: '비 중성화' },
        { value: '1.0', label: '비만 경향' },
        { value: '0.8', label: '체중 감량용' },
      ],
    };
  },
  methods: {
    handleSubmit() {
      // 결과 계산 로직
      const inputValue = parseInt(this.inputValue);
      const selectedOption1Value = parseInt(this.selectedOption1);
      const selectedOption2Value = parseFloat(this.selectedOption2);
      const calculatedValue = 70 * Math.pow(inputValue, 0.75) * selectedOption2Value;
      const roundedValue = Math.floor(calculatedValue * 10) / 10;
      const calculatedValue1 = inputValue * 100 / (100 + (selectedOption1Value - 5) * 10);
      const roundedValue1 = Math.floor(calculatedValue1 * 10) / 10;
      const dogResultPrefix = `우리 아이의 표준 체중은 ${roundedValue1}kg 이고 하루 섭취 칼로리는 ${roundedValue} kcal 입니다! `;
      const catResultPrefix = `우리 아이의 기초 대사량은 ${30 * inputValue + 70} kcal 입니다. 하루 섭취 칼로리는 ${roundedValue}kcal 입니다! `;

      console.log(this.selectedOption2)
      if(this.selectedPet === 'dog'){
        switch (selectedOption1Value) {
        case 1:
          this.resultText = ` 심한 저체중 입니다. 밥을 훨씬 더 줘도 괜찮아요 :) ${dogResultPrefix}`;
          break;
        case 3:
          this.resultText = ` 저체중 입니다. 밥을 조금 더 줘도 괜찮아요 :) ${dogResultPrefix}`;
          break;
        case 5:
          this.resultText = ` 적정 체중이네요 아주 좋아요 :) ${dogResultPrefix}`;
          break;
        case 7:
          this.resultText = ` 과체중입니다. 산책 10분 더 늘려주세요 :) ${dogResultPrefix}`;
          break;
        case 9:
          this.resultText = ` 비만입니다. 산책 30분 더 늘리고 간식을 :) ${dogResultPrefix}`;
          break;
        default:
          this.resultText = '';
          break;
      }
      }else if (this.selectedPet === 'cat'){
        switch (selectedOption1Value) {
          case 1:
            this.resultText = ` 심한 저체중 입니다. 밥을 훨씬 더 줘도 괜찮아요 :) ${catResultPrefix}`;
            break;
          case 3:
            this.resultText = ` 저체중 입니다. 밥을 조금 더 줘도 괜찮아요 :) ${catResultPrefix}`;
            break;
          case 5:
            this.resultText = ` 적정 체중이네요 아주 좋아요 :) ${catResultPrefix}`;
            break;
          case 7:
            this.resultText = ` 과체중입니다. 산책 10분 더 늘려주세요 :) ${catResultPrefix}`;
            break;
          case 9:
            this.resultText = ` 비만입니다. 산책 30분 더 늘리고 간식을 줄여주세요 :) ${catResultPrefix}`;
            break;
          default:
            this.resultText = '';
            break;
        }
      }
    },
    handleCatChange() { 
      // 사용 안됨
      if (this.selectedOption1 === 'cat') {
        // 두 번째 select 요소의 옵션을 고양이에 맞게 변경
        this.options2 = [
          { value: '3', label: '8개월 이하' },
          { value: '2', label: '5 ~ 12 개월' },
          { value: '1.6', label: '중성화' },
          { value: '1.8', label: '비 중성화' },
          { value: '1.4', label: '비만 경향' },
          { value: '1.0', label: '체중 감량용' },
        ];
      } else { //강아지
        this.options2 = [
          { value: '2.5', label: '4개월 이하' },
          { value: '2.5', label: '5 ~ 12 개월' },
          { value: '1.2', label: '중성화' },
          { value: '1.4', label: '비 중성화' },
          { value: '1.0', label: '비만 경향' },
          { value: '0.8', label: '체중 감량용' },
        ];
      }
      this.selectedOption2 = this.options2[0].value;
    },
    validateInput() {
      this.inputValue = this.inputValue.replace(/[^\d.]/g, '');
    }
  },
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
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Ownglyph_meetme-Rg';
}
body{
  text-align: center;
}
form{
  width: 40%;
}
.banner {
  margin-top: 120px;
  text-align: center;
}

.banner-title {
  font-family: 'Ownglyph_meetme-Rg';
  font-size: 3.3rem;
}

.banner-subtitle {
  font-family: 'Ownglyph_meetme-Rg';
  font-size: 1.3rem;
  color: #777;
}
.banner-image {
        width: 25%;
    }
.result{
  font-size: 50px;
  margin-bottom: 20px;
}

/* Shop Section */
.shop-section {
  display: flex;
  align-items: center;
  flex-direction: column;
  background-color: #f3f3f3;
  padding: 20px 0;
}
.shop-images {
  display: grid;
  grid-template-columns: repeat(5, 1fr); /* 5개의 카드를 한 줄에 배치 */
  gap: 40px;
  max-width: 1280px;
  width: 100%;
  justify-content: center;
}
.shop-link img {
  width: 100%;
  height: auto; /* 가로 세로 비율 유지 */
  object-fit: cover; /* 이미지 비율 유지하며 요소에 맞춤 */
  margin-bottom: 10px;
}
.shop-link img {
  width: 100%;
  height: 280px;
  object-fit: cover;
  margin-bottom: 10px;
}
.shop-link h3 {
  margin-bottom: 10px;
}
.shop-link a {
  display: inline-block;
  margin-top: 10px;
  font-size: 0.9rem;
  color: blue;
  font-weight: 500;
  transition: color 0.3s ease;
}
.shop-link:hover a {
  color: #c7511f;
  text-decoration: underline;
}
.shop-link img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  margin-bottom: 10px;
}
.shop-link {
  position: relative;
}

.step {
  position: relative;
  bottom: 50px; /* 필요에 따라 이 값을 조정합니다 */
  left: 50%;
  transform: translateX(-50%);
  background-color: rgba(255, 255, 255, 0.8);
  padding: 5px 10px;
  border-radius: 5px;
  font-size: 14px;
  margin-top: 1px;
}
/* Footer */
.footer-title {
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #a9c7eb;
  color: #000000;
  font-size: 0.875rem;
  font-weight: 600;
  height: 60px;
}

.footer-items {
  display: flex;
    width: 100%;
    flex-direction: column;
    -webkit-box-align: center;
    align-items: center;
    text-align: center;
    padding-top: 40px;
  width: 100%;
  margin: 0 auto;
  background: #e2efff;
  height: 300px;
}
.footer-items h3 {
  font-size: 1rem;
  font-weight: 500;
  color: #000000;
  margin: 20px 0 10px 0;
}
.footer-items ul {
  list-style: none;
  margin-bottom: 20px;
}
.footer-items li a {
  color: #000000;
  font-size: 0.875rem;
}
.footer-items li a:hover {
  text-decoration: underline;
}

.footer-items .box-result {
  text-align: center;
  font-size: 40px;
  height: 20px;
  /* 중앙 정렬을 위한 스타일 추가 */
  display: flex;
  justify-content: center;
  align-items: center;
}
.wrapper {
  padding: 30px 40px;
  border-radius: 12px;
  background: #fff;
  text-align: center;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.1);
  width: 60%;
  height: 65%;
}
.wrapper header {
  font-size: 18px;
  font-weight: 400;
  color: #333;
}
.wrapper p {
  color: #333;
}
.wrapper .input-field {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin: 25px 0;
}
.input-field input,
.input-field button {
  height: 50px;
  width: calc(100% / 2 - 20px);
  outline: none;
  padding: 0 20px;
  border-radius: 8px;
  font-size: 18px;
}
.input-field input {
  text-align: center;
  color: #707070;
  width: 110px;
  border: 1px solid #aaa;
}
input::-webkit-inner-spin-button,
input::-webkit-outer-spin-button {
  display: none;
}
.input-field input:disabled {
  cursor: not-allowed;
}
.input-field button {
  border: none;
  background: #4a98f7;
  color: #fff;
  cursor: pointer;
  transition: 0.3s;
  
}
.input-field button:active {
  transform: scale(0.97);
}
.box-fat{
  font-size: 30px;
  color: #2d399e;
}
.footer-items .box-result {
  text-align: center;
  font-size: 40px;
  height: 20px;
}

.submit {
  display: flex;
  justify-content: space-between;
  margin-top: 10%;
  position: relative; /* 추가 */
}

.submit-button {
  width: calc(50% - 10px);
  height: 48px;
  border-radius: 14px;
  border: none;
  outline: none;
  color: rgb(0, 0, 0);
  background-color: rgb(252, 209, 30);
  font-family: 'Ownglyph_meetme-Rg';
  font-size: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative; /* 추가 */
}
.submit-button:hover {
  background-color:  rgba(255, 204, 0, 0.582);
}
.input-text {
  margin-top: auto;
  width: calc(40% - 10px);
  text-align: center; 
  margin-right: 10px;
}

.petweight{
  margin-top: auto;
  width: calc(40% - 10px);
  text-align: center; 
  margin-right: 20px;
}
.revel{
  margin-top: auto;
  width: calc(40% - 10px);
  text-align: center; 
  margin-right: 10px;
}
.lifestyle{
  margin-top: auto;
  width: calc(40% - 10px);
  text-align: center; 
  margin-right: 10px;
}
.input-container {
  position: relative;
}

.petweight {
  width: calc(100% - 30px); 
  padding-right: 30px; 
  margin-top: 5px;
}

.input-container::after {
  content: 'kg';
  position: absolute;
  top: 50%;
  right: 10px; /* 아이콘 간격 조정 */
  transform: translateY(-50%);
  color: #777;
}

.error-message {
  color: #ff0000;
  font-size: 1.0rem;
  position: absolute;
  bottom: calc(100% + 5px); /* 버튼 아래로 위치 */
  left: 50%;
  transform: translateX(-50%);
}

</style>
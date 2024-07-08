<script setup>
import axios from 'axios'
import { ref } from 'vue'

const TOKEN_ID = '1c5ac405-1919-0ac1-e063-63199f0ac402'
const TOKEN_KEY =
  'MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKj15ikj3q4kNESLkfLrJzGAOFsiScOw3iByTrhj8+c3HAnIGMZ9m+lQFUkfBOuTOdgNQdyA8ReoQniKKWZM63sCAwEAAQ=='
const ACCESS_TOKEN =
  'Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxYzVhYzI3Yy05OGM3LTFhMzktZTA2My02MzE5OWYwYTkyOWQiLCJhdWQiOlsicmVzdHNlcnZpY2UiXSwidXNlcl9uYW1lIjoiaG9hbmdhYXRyb3gyMzUwNEBnbWFpbC5jb20iLCJzY29wZSI6WyJyZWFkIl0sImlzcyI6Imh0dHBzOi8vbG9jYWxob3N0IiwibmFtZSI6ImhvYW5nYWF0cm94MjM1MDRAZ21haWwuY29tIiwidXVpZF9hY2NvdW50IjoiMWM1YWMyN2MtOThjNy0xYTM5LWUwNjMtNjMxOTlmMGE5MjlkIiwiYXV0aG9yaXRpZXMiOlsiVVNFUiJdLCJqdGkiOiI1NjA3OGYyYi03NjgzLTRmMDItOTlhOC1jODQ2NzVhYTc1ODIiLCJjbGllbnRfaWQiOiJhZG1pbmFwcCJ9.1i3TXpOJiCMnu3OXFZVYkL7X6N8qC7AwBlevX5JRS6HWICkKraCoC_IIBx3IblL3nRCFIi9q0TGwQeoSuvXExaTUN2VygEL89qCocojVJvSEermeEoLNVvBpnSjb-MrraCLuCJPX5jvk3yYkZm3LC2mNNnc8AeT5c0oZAxyL6tRLArm_f7AVGkfJpMKWz2J0Q4MEmxiV0QxvoZeRMbTf_LTPnkxOrqp5l2iRh5BQ0YfD5nJEFrmgmwPEP8NVXVuoVzca84oWzJAEHUII9Xv4LEZvxf4HliRwZZFlZAq8vxRv25uaNRJVpgMj2Yw442X41UVc33UUJIHveeXKFZdm_g'

const API_URL = 'https://api.idg.vnpt.vn/tts-service/v2/grpc'

const callTtsApi = async (text, region, speed) => {
  const axiosConfig = {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': ACCESS_TOKEN,
      'Token-id': TOKEN_ID,
      'Token-key': TOKEN_KEY
    }
  }

  const data = {
    text,
    region,
    speed,
    text_split: false
  }

  console.log('Header:', axiosConfig)
  console.log('Data:', data)

  try {
    const response = await axios.post(API_URL, data, axiosConfig)
    console.log('RESPONSE: ', response)
    return response.data
  } catch (error) {
    console.error('ERROR:', error.response ? error.response.data : error)
    throw error
  }
}

const inputText = ref('')
const selectedRegion = ref('female_north')
const selectedSpeed = ref(1.0)
const responseData = ref(null)
const audioSrc = ref('')

const handleTtsConversion = async () => {
  try {
    responseData.value = await callTtsApi(
      inputText.value,
      selectedRegion.value,
      selectedSpeed.value
    )
    audioSrc.value = responseData.value?.object?.playlist?.[0]?.audio_link || ''
  } catch (error) {
    console.error('Không thể chuyển đổi văn bản thành giọng nói:', error)
    responseData.value = {
      error: 'Không thể chuyển đổi văn bản thành giọng nói. Vui lòng thử lại.'
    }
  }
}
</script>

<template>
  <main class="container">
    <div class="input-container">
      <label for="">Văn bản muốn chuyển đổi:</label>
      <textarea
        v-model="inputText"
        placeholder="Nhập văn bản để chuyển đổi thành giọng nói"
        rows="5"
        cols="50"
      ></textarea>
      <label for="">Chọn giọng đọc:</label>
      <select v-model="selectedRegion">
        <option value="female_north">Nữ Bắc</option>
        <option value="female_central">Nữ Trung</option>
        <option value="female_south">Nữ Nam</option>
        <option value="male_north">Nam Bắc</option>
        <option value="male_central">Nam Trung</option>
        <option value="male_south">Nam Nam</option>
      </select>
      <label for="">Chọn tốc độ:</label>
      <input type="number" v-model.number="selectedSpeed" step="0.1" min="0.1" max="2.0" />
      <button @click="handleTtsConversion">Chuyển đổi thành giọng nói</button>
    </div>
    <div class="result-container">
      <h2>Kết quả:</h2>
      <pre>{{ JSON.stringify(responseData, null, 2) }}</pre>
      <div v-if="audioSrc">
        <h3>Âm thanh đã tạo:</h3>
        <audio controls :src="audioSrc"></audio>
      </div>
    </div>
  </main>
</template>

<style>
* {
  font-family: 'Be Vietnam Pro';
}
.container {
  display: flex;
  height: 100vh;
}

.input-container,
.result-container {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
  box-sizing: border-box;
}

.input-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
  border-right: 1px solid #ccc;
}

textarea {
  width: 100%;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
  resize: vertical;
  outline: 1px solid #d9d9d9;
}

select,
input[type='number'],
button,
input[type='text'] {
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 5px;
  outline: 1px solid #d9d9d9;
}

button {
  background-color: #0c61a1;
  color: white;
  border: none;
  cursor: pointer;
  transition: all 0.2s;
}

button:hover {
  filter: brightness(1.2);
}

.result-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: #f9f9f9;
}

audio {
  width: 100%;
  margin-top: 10px;
}
</style>

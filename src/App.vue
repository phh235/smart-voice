<script setup>
import { ref, computed } from 'vue'

const TOKEN_ID = '1c5ac405-1919-0ac1-e063-63199f0ac402'
const TOKEN_KEY =
  'MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKj15ikj3q4kNESLkfLrJzGAOFsiScOw3iByTrhj8+c3HAnIGMZ9m+lQFUkfBOuTOdgNQdyA8ReoQniKKWZM63sCAwEAAQ=='
const ACCESS_TOKEN =
  'Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxYzVhYzI3Yy05OGM3LTFhMzktZTA2My02MzE5OWYwYTkyOWQiLCJhdWQiOlsicmVzdHNlcnZpY2UiXSwidXNlcl9uYW1lIjoiaG9hbmdhYXRyb3gyMzUwNEBnbWFpbC5jb20iLCJzY29wZSI6WyJyZWFkIl0sImlzcyI6Imh0dHBzOi8vbG9jYWxob3N0IiwibmFtZSI6ImhvYW5nYWF0cm94MjM1MDRAZ21haWwuY29tIiwidXVpZF9hY2NvdW50IjoiMWM1YWMyN2MtOThjNy0xYTM5LWUwNjMtNjMxOTlmMGE5MjlkIiwiYXV0aG9yaXRpZXMiOlsiVVNFUiJdLCJqdGkiOiI1NjA3OGYyYi03NjgzLTRmMDItOTlhOC1jODQ2NzVhYTc1ODIiLCJjbGllbnRfaWQiOiJhZG1pbmFwcCJ9.1i3TXpOJiCMnu3OXFZVYkL7X6N8qC7AwBlevX5JRS6HWICkKraCoC_IIBx3IblL3nRCFIi9q0TGwQeoSuvXExaTUN2VygEL89qCocojVJvSEermeEoLNVvBpnSjb-MrraCLuCJPX5jvk3yYkZm3LC2mNNnc8AeT5c0oZAxyL6tRLArm_f7AVGkfJpMKWz2J0Q4MEmxiV0QxvoZeRMbTf_LTPnkxOrqp5l2iRh5BQ0YfD5nJEFrmgmwPEP8NVXVuoVzca84oWzJAEHUII9Xv4LEZvxf4HliRwZZFlZAq8vxRv25uaNRJVpgMj2Yw442X41UVc33UUJIHveeXKFZdm_g'

const API_URL = 'https://api.idg.vnpt.vn/tts-service/v2/grpc'

const callTtsApi = async (text, region, speed) => {
  const headers = {
    'Content-Type': 'application/json',
    'Authorization': ACCESS_TOKEN,
    'Token-id': TOKEN_ID,
    'Token-key': TOKEN_KEY
  }

  console.log('Request headers:', headers)
  console.log('Request body:', {
    text,
    region,
    speed,
    text_split: false
  })

  try {
    const response = await fetch(API_URL, {
      method: 'POST',
      headers: headers,
      body: JSON.stringify({
        text,
        region,
        speed,
        text_split: false
      })
    })

    if (!response.ok) {
      const errorData = await response.json()
      console.error('Response status:', response.status)
      console.error('Response status text:', response.statusText)
      console.error('Error response data:', errorData)
      throw new Error(`Error: ${errorData.message || 'Unknown error'}`)
    }

    const data = await response.json()
    console.log('Full response:', data)
    return data
  } catch (error) {
    console.error('Error details:', error)
    throw error
  }
}

const inputText = ref('')
const selectedRegion = ref('female_north')
const selectedSpeed = ref(1.0)
const responseData = ref(null)
const isLoading = ref(false)
const audioSrc = ref('')

const isInputValid = computed(() => inputText.value.trim() !== '')

const handleTtsConversion = async () => {
  if (!isInputValid.value) {
    alert('Vui lòng nhập văn bản để chuyển đổi')
    return
  }

  isLoading.value = true
  try {
    const data = await callTtsApi(inputText.value, selectedRegion.value, selectedSpeed.value)
    responseData.value = data
    audioSrc.value = data.object?.playlist?.[0]?.audio_link || ''
  } catch (error) {
    console.error('Không thể chuyển đổi văn bản thành giọng nói:', error)
    responseData.value = {
      error: 'Không thể chuyển đổi văn bản thành giọng nói. Vui lòng thử lại.'
    }
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <main>
    <div>
      <textarea
        v-model="inputText"
        placeholder="Nhập văn bản để chuyển đổi thành giọng nói"
        rows="4"
        cols="50"
      ></textarea>
      <br />
      <select v-model="selectedRegion">
        <option value="female_north">Nữ Bắc</option>
        <option value="female_central">Nữ Trung</option>
        <option value="female_south">Nữ Nam</option>
        <option value="male_north">Nam Bắc</option>
        <option value="male_central">Nam Trung</option>
        <option value="male_south">Nam Nam</option>
      </select>
      <input type="number" v-model.number="selectedSpeed" step="0.1" min="0.1" max="2.0" />
      <button @click="handleTtsConversion" :disabled="isLoading || !isInputValid">
        {{ isLoading ? 'Đang chuyển đổi...' : 'Chuyển đổi thành giọng nói' }}
      </button>
    </div>
    <div v-if="responseData">
      <h2>Phản hồi từ API:</h2>
      <pre>{{ JSON.stringify(responseData, null, 2) }}</pre>
      <div v-if="audioSrc">
        <h3>Âm thanh đã tạo:</h3>
        <audio controls :src="audioSrc"></audio>
      </div>
    </div>
  </main>
</template>

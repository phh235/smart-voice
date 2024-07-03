<script setup>
import axios from 'axios'
import { ref } from 'vue'

const token_id = '1c5ac405-1919-0ac1-e063-63199f0ac402'
const token_key = 'MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBAKj15ikj3q4kNESLkfLrJzGAOFsiScOw3iByTrhj8+c3HAnIGMZ9m+lQFUkfBOuTOdgNQdyA8ReoQniKKWZM63sCAwEAAQ=='
const access_token = 'Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxYzVhYzI3Yy05OGM3LTFhMzktZTA2My02MzE5OWYwYTkyOWQiLCJhdWQiOlsicmVzdHNlcnZpY2UiXSwidXNlcl9uYW1lIjoiaG9hbmdhYXRyb3gyMzUwNEBnbWFpbC5jb20iLCJzY29wZSI6WyJyZWFkIl0sImlzcyI6Imh0dHBzOi8vbG9jYWxob3N0IiwibmFtZSI6ImhvYW5nYWF0cm94MjM1MDRAZ21haWwuY29tIiwidXVpZF9hY2NvdW50IjoiMWM1YWMyN2MtOThjNy0xYTM5LWUwNjMtNjMxOTlmMGE5MjlkIiwiYXV0aG9yaXRpZXMiOlsiVVNFUiJdLCJqdGkiOiI1NjA3OGYyYi03NjgzLTRmMDItOTlhOC1jODQ2NzVhYTc1ODIiLCJjbGllbnRfaWQiOiJhZG1pbmFwcCJ9.1i3TXpOJiCMnu3OXFZVYkL7X6N8qC7AwBlevX5JRS6HWICkKraCoC_IIBx3IblL3nRCFIi9q0TGwQeoSuvXExaTUN2VygEL89qCocojVJvSEermeEoLNVvBpnSjb-MrraCLuCJPX5jvk3yYkZm3LC2mNNnc8AeT5c0oZAxyL6tRLArm_f7AVGkfJpMKWz2J0Q4MEmxiV0QxvoZeRMbTf_LTPnkxOrqp5l2iRh5BQ0YfD5nJEFrmgmwPEP8NVXVuoVzca84oWzJAEHUII9Xv4LEZvxf4HliRwZZFlZAq8vxRv25uaNRJVpgMj2Yw442X41UVc33UUJIHveeXKFZdm_g'

const apiUrl = 'https://api.idg.vnpt.vn'
const subUrl = 'tts-service/v2/grpc'

const callTtsApi = async (text, region = 'female_north', speed = '1') => {
  const config = {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': access_token,
      'Token-id': token_id,
      'Token-key': token_key
    }
  }
  const data = {
    text: text,
    region: region,
    speed: speed,
    text_split: false
  }
  
  console.log('Request URL:', `${apiUrl}/${subUrl}`)
  console.log('Request Headers:', config.headers)
  console.log('Request Data:', data)

  try {
    const response = await axios.post(`${apiUrl}/${subUrl}`, data, config)
    return response.data
  } catch (error) {
    console.error('Error calling TTS API:', error.response ? error.response.data : error.message)
    console.error('Status:', error.response ? error.response.status : 'Unknown')
    console.error('Headers:', error.response ? error.response.headers : 'Unknown')
    console.error('Request config:', error.config)
    throw error
  }
}

const inputText = ref('')
const selectedRegion = ref('female_north')
const selectedSpeed = ref('1')
const responseData = ref(null)
const isLoading = ref(false)
const audioSrc = ref('')

const handleTtsConversion = async () => {
  if (!inputText.value) {
    alert('Please enter some text')
    return
  }
  
  isLoading.value = true
  try {
    const data = await callTtsApi(inputText.value, selectedRegion.value, selectedSpeed.value)
    responseData.value = data
    if (data.object && data.object.playlist && data.object.playlist.length > 0) {
      audioSrc.value = data.object.playlist[0].audio_link
    }
  } catch (error) {
    console.error('Failed to convert text to speech:', error)
    responseData.value = { error: 'Failed to convert text to speech. Please try again.' }
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <main>
    <div>
      <textarea v-model="inputText" placeholder="Enter text to convert to speech" rows="4" cols="50"></textarea>
      <br>
      <select v-model="selectedRegion">
        <option value="female_north">Female North</option>
        <option value="female_central">Female Central</option>
        <option value="female_south">Female South</option>
        <option value="male_north">Male North</option>
        <option value="male_central">Male Central</option>
        <option value="male_south">Male South</option>
      </select>
      <input type="text" v-model="selectedSpeed" placeholder="Speed (e.g. '1')">
      <button @click="handleTtsConversion" :disabled="isLoading">Convert to Speech</button>
    </div>
    <div v-if="isLoading">
      <p>Converting...</p>
    </div>
    <div v-else-if="responseData">
      <h2>API Response:</h2>
      <pre>{{ JSON.stringify(responseData, null, 2) }}</pre>
      <div v-if="audioSrc">
        <h3>Generated Audio:</h3>
        <audio controls :src="audioSrc"></audio>
      </div>
    </div>
  </main>
</template>

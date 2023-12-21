<template>
  <!-- Loading -->
  <div v-if="isLoading" 
    class="tw-inset tw-w-full tw-h-[100vh] tw-fixed tw-flex tw-items-center tw-justify-center tw-opacity-50 tw-bg-gray-600 tw-z-50">
    <q-circular-progress
      indeterminate
      size="80px"
      color="orange-1"
      class="q-ma-md"
    />
  </div>
  <!-- PopUp -->
  <div class="q-pa-xs q-gutter-sm">
    <q-dialog v-model="alertSwitch" persistent transition-show="scale" transition-hide="scale">
      <q-card class="tw-bg-orange-300 text-white tw-w-1/2">
        <q-card-section>
          <div class="text-h6">Hello! This is alert.</div>
        </q-card-section>
        <q-card-section class="q-pt-none">
          {{alertMsg}}
        </q-card-section>
        <q-card-actions align="right" class="bg-white text-teal">
          <q-btn flat label="OK" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
  <!--  -->
  <q-page>
    <div class="md:tw-w-[40vw] tw-mx-auto tw-p-2 tw-mt-4">
      <q-file color="orange" accept="image/jpeg, image/png, image/gif, image/webp"
        filled v-model="file" label="Upload Image" loading="true"
      >
        <template v-slot:prepend>
          <q-icon name="cloud_upload" />
        </template>
      </q-file>
    </div>
    <div class="md:tw-flex tw-items-center tw-justify-center tw-py-6 xs:tw-py-2">
      <div class="md:tw-w-[40vw] tw-mx-auto tw-p-2" >
        <p class="tw-text-gray-600 tw-mb-2">
          Provide me with a photo, and I will return the content to you through AI.
        </p>
        <div v-if="imgLoading">
          <q-card flat class="md:tw-max-w-[40vw]">
            <q-skeleton class="md:tw-h-[20rem] tw-h-[15rem]" square />
            <q-card-section>
              <q-skeleton type="text" class="text-subtitle1" />
              <q-skeleton type="text" width="50%" class="text-subtitle1" />
              <q-skeleton type="text" class="text-caption" />
            </q-card-section>
          </q-card>
        </div>
        <div v-if="isAnalyze" class="tw-mb-4">
          <img :src="imgBase64Src" alt="" class="rounded">
          <q-input v-model="aiDescription"
            filled color="orange-5"
            label="AI generated description"
            type="textarea" autogrow
          />
          <div class="tw-flex tw-justify-end">
            <q-btn push color="orange-5" text-color="white" label="Copy description" 
              class="tw-my-1 tw-p-1" @click="copyDescription()"
            />
          </div>
        </div>
      </div>
      <div class="md:tw-w-[40vw] tw-mx-auto tw-p-2">
        <p class="tw-text-gray-600 tw-mb-2">
          You can either modify the image description based on the one you uploaded or generate an image according to your own imagination
        </p>
        <img :src="generationImgSrc" alt="" class="rounded" @load="imgLoadingFinish()">
        <q-input v-model="generationPrompt"
          filled color="orange-5"
          label="Input your prompt"
          type="textarea" autogrow
        />
        <div class="tw-flex tw-justify-end tw-gap-4">
          <q-btn v-if="downloadBtn" push color="deep-orange-6" 
            text-color="white" label="Download Image"
            class="tw-my-1 tw-p-1" @click="downloadImage()"
          />
          <q-btn push color="deep-orange-6" text-color="white" 
            label="Generate Your Ideas" 
            class="tw-my-1 tw-p-1" @click="generateImage()"
          />
        </div>
      </div>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import type { Ref } from 'vue'
import { ref , computed ,onMounted , watch} from 'vue'
import Axios from 'axios'

const isLoading : Ref<boolean> = ref(false)
// 給 AI 生成圖片的提示
const generationPrompt : Ref<string> = ref("")
const generationImgSrc : Ref<string> = ref("")
// AI 生成的描述
const aiDescription : Ref<string> = ref("")
// 控制是否顯示結果
const isAnalyze : Ref<boolean> = ref(true)
// 控制 alert
const alertMsg : Ref<string> = ref("")
const alertSwitch : Ref<boolean>  = ref(false)
// 控制 skeleton loading
const imgLoading : Ref<boolean> = ref(false)
// 圖片base64
const imgBase64Src : Ref<string | ArrayBuffer | null> = ref(null)
const downloadBtn : Ref<boolean> = ref(false)
// 上傳檔案
const file : Ref<File | null> = ref(null) 
watch(file ,()=>{
  let size = (file.value as File).size
  if(size > 1024*1024*20){
    alertMsg.value = "File size is too large"
    return
  }
  imgLoading.value = true
  isAnalyze.value = false
  const reader = new FileReader();
  // 當圖片上傳完成後，打後端API取得分析結果
  reader.onload = (e) => {
    if (e.target) {
      imgBase64Src.value = e.target.result as string;
      analyzeImage()
    }
  };
  reader.readAsDataURL(file.value as File);
})

const analyzeImage = () : void => {
  if(file.value === null){
    alertMsg.value = "Please upload a photo"
    return
  }
  // http://localhost:5000/node_ai/analyze_images
  Axios.post('https://brief-url.link/node_ai/analyze_images',{imgBase64:imgBase64Src.value}
  ).then(res => {
    aiDescription.value = res.data.result
    imgLoading.value = false
    isAnalyze.value = true
  })
  .catch(err => {
    console.log(err)
    imgLoading.value = false
  }) 
}

const copyDescription = ()=>{
  if(navigator.clipboard && navigator.clipboard.writeText){
    navigator.clipboard.writeText(aiDescription.value).then(()=>{
      alertMsg.value = 'Copied to clipboard'
      alertSwitch.value = true
    }).catch((err)=>{
      console.log(err)
      alertMsg.value = 'Copy failed，Http does not support'
      alertSwitch.value = true
    })
  }else{
    alertMsg.value = 'Copy failed，Http does not support'
    alertSwitch.value = true
  }
}

const generateImage = ()=>{
  if(generationPrompt.value == ""){
    alertMsg.value = "Please input your prompt"
    alertSwitch.value = true
    return
  }
  isLoading.value = true
  // http://localhost:5000/node_ai/create_images
  Axios.post('https://brief-url.link/node_ai/create_images',{prompt:generationPrompt.value}
  ).then(res => {
    generationImgSrc.value = res.data.result
  })
  .catch(err => {
    console.log(err)
    isLoading.value = false
  }) 
}

const imgLoadingFinish = ()=>{
  isLoading.value = false
  downloadBtn.value = true
}

const downloadImage = async ()=>{
  isLoading.value = true
  // http://localhost:5000/node_ai/download_images
  Axios.post('https://brief-url.link/node_ai/download_images',{url:generationImgSrc.value}
  ).then(res => {
    const responseData = res.data.result
    const encodedBase64 = encodeURIComponent(responseData);
    const downloadLink = document.createElement("a");
    downloadLink.href = `data:image/png;base64,${encodedBase64}`;
    downloadLink.download = `download${new Date().toLocaleString()}.png`;
    document.body.appendChild(downloadLink);
    downloadLink.click();
    document.body.removeChild(downloadLink);
    isLoading.value = false
  })
  .catch(err => {
    console.log(err)
    isLoading.value = false
  })
}

</script>

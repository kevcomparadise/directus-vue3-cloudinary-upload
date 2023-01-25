<template>
  <div id="cloudinary-upload-dropzone">
    <div class="loader" v-if="uploadProgress">
      <div>Please wait, upload in progress</div>
    </div>
    <div v-bind="getRootProps()" class="draggable-zone" :class="{ 'is-dragged': isDragActive }" :style="{
      backgroundImage:`url(${previewUrl})`
    }">
      <input v-bind="getInputProps()"/>
    </div>
  </div>
</template>

<script>
import {useDropzone} from "vue3-dropzone";
import {inject, onMounted, ref, watchEffect} from "vue";
import * as sha from "js-sha1";
import axios from 'axios'

export default {
  name: "UseDropzoneDemo",
  props: {
    value: {
      type: String,
      default: null,
    },
    collection: {
      type: String,
      default: null,
    },
  },
  setup(props, {emit}) {
    const apiUrl = ref('__CDN_API_URL__');
    const apiSecret = ref('__CDN_API_SECRET__');
    const apiKey = ref('__CDN_API_KEY__');
    const apiFolder = ref('__CDN_API_FOLDER__');
    const previewUrl = ref('')
    const uploadProgress = ref(false)
    const values = inject('values')

    const toggleProgress = () => uploadProgress.value = !uploadProgress.value;
    /**
     * @param url
     * @returns {Promise<Blob>}
     */
    const fetchImage = async (url) => await fetch(url).then(r => r.blob());

    /**
     * @param blob
     * @returns {Promise<unknown>}
     */
    const urlToBase64 = async function (blob) {
      return new Promise(resolve => {
        let reader = new FileReader();
        reader.onload = () => resolve(reader.result);
        reader.readAsDataURL(blob);
      });
    }

    const setPreview = async (value) => {
      const blob = await fetchImage(value)
      previewUrl.value = await urlToBase64(blob)
    }

    /**
     * @param publicId
     * @param timestamp
     * @returns {*}
     */
    const generateSignature = function (publicId, timestamp) {
      const params = `public_id=${publicId}&timestamp=${timestamp}${apiSecret.value}`
      return sha.default.hex(params)
    }

    /**
     * @param fileName
     * @returns {`${string}/${string}`}
     */
    const generateFileName = (fileName) => {
      const name = String(fileName).toLowerCase().replace(/ /g, '_')
      const folder = apiFolder.value !== "undefined" ? `${apiFolder.value}/` : ''
      return `${folder}${props.collection}/${name}`
    }

    const {getRootProps, getInputProps, isDragAccept, ...rest} = useDropzone({
      onDrop,
      maxFiles: 1,
      accept: 'image/*'
    });

    /**
     * @param formData
     * @returns {Promise<void>}
     */
    async function send(formData) {
      toggleProgress()
      const response = await axios.post(`${apiUrl.value}/image/upload`, formData);
      toggleProgress()
      return response.data.secure_url
    }

    /***
     * @param acceptFiles
     * @param rejectReasons
     * @returns {Promise<void>}
     */
    async function onDrop(acceptFiles, rejectReasons) {
      if (isDragAccept) {
        let dataUrl = await urlToBase64(acceptFiles[0])
        let fileName = generateFileName(acceptFiles[0].name)
        // Remove extension
        fileName = fileName.replace(/\.[^/.]+$/, "");
        const timestamp = (new Date()).getTime().toString()
        const formData = new FormData()
        formData.append('file', dataUrl)
        formData.append('api_key', apiKey.value)
        formData.append('public_id', fileName)
        formData.append('timestamp', timestamp)
        formData.append('signature', generateSignature(fileName, timestamp))
        const url = await send(formData)
        emit('input', url)
      }
    }

    watchEffect(() => {
      setPreview(props.value)
    })
    onMounted(async () => {
      await setPreview(props.value);
    })

    return {
      getRootProps,
      getInputProps,
      previewUrl,
      values,
      uploadProgress,
      ...rest,
    };
  },
};
</script>

<style>
#cloudinary-upload-dropzone {
  width: 100%;
  padding: 20px;
  background: #f0f4f9;
  border-radius: 10px;
  position: relative;
}

#cloudinary-upload-dropzone .draggable-zone {
  border-radius: 10px;
  min-height: 200px;
  background-repeat: no-repeat;
  background-size: cover;
}

#cloudinary-upload-dropzone .draggable-zone.is-dragged {
  border: 5px dashed slategrey;
}

#cloudinary-upload-dropzone img {
  max-width: 370px;
}

#cloudinary-upload-dropzone .loader {
  background: rgba(0, 0, 0, 0.8);
  text-align: center;
  position: absolute;
  border-radius: 10px;
  width: 100%;
  height: 100%;
  top:0;
  left: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
}
</style>
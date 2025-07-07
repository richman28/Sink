<script setup>
import { ref, onMounted, watch } from 'vue'
import { Download, Image as ImageIcon, XCircle } from 'lucide-vue-next'
import QRCodeStyling from 'qr-code-styling'

const props = defineProps({
  data: {
    type: String,
    required: true,
  },
})

const color = ref('#000000')
const qrCodeEl = ref(null)
const fileInput = ref(null)
const uploadedLogoName = ref('')
const uploadedLogoPreview = ref('')
const shape = ref('square')  // 👉 default bentuk QR

const options = {
  width: 256,
  height: 256,
  data: props.data,
  margin: 10,
  qrOptions: { typeNumber: '0', mode: 'Byte', errorCorrectionLevel: 'Q' },
  image: '',
  imageOptions: {
    hideBackgroundDots: true,
    imageSize: 0.3,
    margin: 2,
  },
  dotsOptions: { type: shape.value, color: '#000000' },
  backgroundOptions: { color: '#ffffff' },
  cornersSquareOptions: { type: 'square', color: '#000000' },
  cornersDotOptions: { type: 'square', color: '#000000' },
}

const qrCode = new QRCodeStyling(options)

function updateQRCode() {
  qrCode.update({
    dotsOptions: { type: shape.value, color: color.value },
    cornersSquareOptions: { type: shape.value, color: color.value },
    cornersDotOptions: { type: shape.value, color: color.value },
    image: uploadedLogoPreview.value || '',
  })
}

watch([color, shape], updateQRCode)

function handleImageUpload(event) {
  const file = event.target.files[0]
  if (!file) return

  const allowedTypes = ['image/png', 'image/jpeg', 'image/jpg', 'image/svg+xml']
  const maxSize = 500 * 1024

  if (!allowedTypes.includes(file.type)) {
    alert('Hanya mendukung file PNG, JPG, JPEG, atau SVG.')
    return
  }
  if (file.size > maxSize) {
    alert('Ukuran logo maksimal 500KB.')
    return
  }

  const reader = new FileReader()
  reader.onload = (e) => {
    uploadedLogoName.value = file.name
    uploadedLogoPreview.value = e.target.result
    updateQRCode()
  }
  reader.readAsDataURL(file)
}

function resetLogo() {
  uploadedLogoName.value = ''
  uploadedLogoPreview.value = ''
  updateQRCode()
}

function downloadQRCode() {
  const slug = props.data.split('/').pop()
  qrCode.download({
    extension: 'png',
    name: `qr_${slug}`,
  })
}

onMounted(() => {
  qrCode.append(qrCodeEl.value)
})
</script>

<template>
  <div class="flex flex-col items-center gap-6">
    <div ref="qrCodeEl" class="bg-white p-2 rounded-2xl shadow-lg" />

    <div class="flex flex-wrap gap-4 items-center justify-center">
      <div class="relative flex items-center">
        <div
          class="w-10 h-10 rounded-full border border-gray-300 dark:border-gray-600 cursor-pointer overflow-hidden"
          :style="{ backgroundColor: color }"
          title="Ganti warna QR"
        >
          <input
            v-model="color"
            type="color"
            class="absolute inset-0 w-full h-full opacity-0 cursor-pointer"
          />
        </div>
      </div>

      <select v-model="shape" class="border rounded-lg px-2 py-1 text-sm focus:outline-none">
        <option value="square">Kotak</option>
        <option value="dots">Bulat</option>
        <option value="rounded">Rounded</option>
        <option value="classy">Classy</option>
        <option value="classy-rounded">Classy Rounded</option>
      </select>

      <Button variant="outline" size="sm" @click="downloadQRCode">
        <Download class="w-4 h-4 mr-2" />
        {{ $t('links.download_qr_code') }}
      </Button>

      <input type="file" accept="image/*" @change="handleImageUpload" class="hidden" ref="fileInput" />
      <Button variant="outline" size="sm" @click="fileInput.click()">
        <ImageIcon class="w-4 h-4 mr-2" />
        Upload Logo
      </Button>
      <Button v-if="uploadedLogoName" variant="destructive" size="sm" @click="resetLogo">
        <XCircle class="w-4 h-4 mr-2" />
        Hapus Logo
      </Button>
    </div>

    <div v-if="uploadedLogoName" class="flex flex-col items-center gap-2 text-xs text-gray-500 dark:text-gray-400 mt-2">
      <span>Logo: {{ uploadedLogoName }}</span>
      <img :src="uploadedLogoPreview" alt="Logo Preview" class="w-12 h-12 rounded-lg border shadow" />
    </div>
  </div>
</template>

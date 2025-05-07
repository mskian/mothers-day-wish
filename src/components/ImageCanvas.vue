<template>
    <div class="canvas-container mt-4 mb-6">
      <canvas ref="canvasRef"></canvas>
      <br />
      <div class="has-text-centered">
        <button class="button is-success is-small" @click="downloadImage">
          <DownloadIcon size="14" class="mr-1" /> Download Image
        </button>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, watch } from 'vue'
  import { DownloadIcon } from 'lucide-vue-next'
  
  const props = defineProps({
    image: {
      type: String,
      required: true
    },
    text: {
      type: String,
      required: true
    }
  })
  
  const canvasRef = ref(null)
  const context = ref(null)
  
  const font = '600 28pt Anek Tamil'
  const color = '#f7f1e3'
  const textX = 516
  const textY = 980
  
  onMounted(() => {
    const canvas = canvasRef.value
    if (canvas) {
      context.value = canvas.getContext('2d')
    }
  })
  
watch(
  () => [props.image, props.text, context.value],
  async () => {
    if (!context.value || !props.image) return

    const img = new Image()
    img.crossOrigin = 'anonymous'
    img.src = props.image

    img.onload = async () => {
      const canvas = canvasRef.value
      if (!canvas || !context.value) return

      await document.fonts.load('28pt "Anek Tamil"')
      await document.fonts.ready

      canvas.width = 1080
      canvas.height = 1080

      const ctx = context.value
      ctx.clearRect(0, 0, canvas.width, canvas.height)
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height)

      const gradient = ctx.createLinearGradient(0, 0, canvas.width, 0);
      gradient.addColorStop(0.6, '#e056fd');
      gradient.addColorStop(0.2, '#e056fd');
      gradient.addColorStop(1, '#e056fd');
      ctx.fillStyle = gradient;
      ctx.shadowColor = '#e056fd';
      ctx.shadowBlur = 4;

      ctx.font = '600 32pt "Anek Tamil"'
      ctx.fillStyle = '#f6e58d'
      ctx.textAlign = 'center'
      ctx.fillText(props.text, 516, 995)
    }
  },
  { immediate: true }
)

  const generateRandomFilename = () => {
    const randomString = Math.random().toString(36).substring(2, 15)
    return `greeting-wish-${randomString}.png`
  }

  const downloadImage = () => {
    const canvas = canvasRef.value
    if (!canvas) return
  
    const dataUrl = canvas.toDataURL('image/png')
    const link = document.createElement('a')
    link.href = dataUrl
    link.download = generateRandomFilename()
    link.click()
  }
  </script>
  
<style scoped>
.canvas-container {
  font-family: "Anek Tamil", sans-serif;
  width: 100%;
  max-width: 540px;
  text-align: center;
  border: 1px solid #f37c45;
  padding: 18px;
}
canvas {
  font-family: "Anek Tamil", sans-serif;
  display: block;
  width: 100%;
  height: auto;
  margin: auto;
  image-rendering: crisp-edges;
}
</style>
<template>
  <div>
    <button @click="change()">切换</button>
    <PdfLocal v-if="isLocal"/>
    <PdfView  v-else />

  </div>
</template>

<script>
import PdfView from '@/components/PDF.vue';
import PdfLocal from '@/components/PDFLocal.vue'
export default {
  name: 'App',
  components: {
    PdfView,
    PdfLocal
  },
  data() {
    return {
      isLocal:false,
    }
  },
  methods:{
    change(){
      this.isLocal = !this.isLocal;
    }
  }
}
</script>

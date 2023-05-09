<template>
  <div>
    <PdfView/>

  </div>
</template>

<script>
import PdfView from '@/components/PDF.vue';
export default {
  name: 'App',
  components: {
    PdfView,
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

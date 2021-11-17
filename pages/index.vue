<template>
  <div>
    <HomeBackground/>
    <div class="docSlider">
      <Section1/>
      <Section2/>
      <Section3/>
    </div>
  </div>
</template>

<script>
  // import docslider from 'docslider/docSlider'

  export default {
    mounted() {
      window.docSlider.init({
        scrollReset: true,
        pager: true,
        beforeChange: (index, page, toIndex) => {
          this.emitSlideChange(toIndex)
        },
        speed : 1500,
        easing : 'ease-in-out'
      })
    },
    methods: {
      emitSlideChange(index) {
        this.$nuxt.$emit('slide-change', index)
      }
    }
  }
</script>
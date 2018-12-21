<template>
<v-content>
    <v-carousel hide-controls width="5" delimiter-icon="cloud">
    <v-carousel-item
      v-for="(item,i) in items"
      :key="i"
      :src="item.src"
    ></v-carousel-item>
  </v-carousel>
</v-content>
</template>
<script>
export default {
  name: 'Banner',
  components: {

  },
  mounted () {

  },
  props: {
    msg: String
  },
  data () {
    return {
      items: [
        {
          src: 'http://cpm.kmutt.ac.th/assets/media/slideshow/project/slide_project_2.jpg'
        },
        {
          src: 'http://admission.kmutt.ac.th/images/shared/kmutt.jpeg'
        },
        {
          src: 'https://www.bhubeth.com/wp-content/uploads/2013/04/sit-kmutt.jpg'
        },
        {
          src: 'https://www.camphub.in.th/wp-content/uploads/2016/08/110642007.jpg'
        }
      ]

    }
  },
  methods: {

  }
}
</script>

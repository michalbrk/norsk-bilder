<template>
  <div>
    <v-container v-if="isGalleryVisible">
      <v-row dense>
        <v-col cols="6" sm="9" md="9" lg="10" xl="11"></v-col>
        <v-col cols="6" sm="3" md="3" lg="2" xl="1">
          <v-card flat color="transparent" href="https://pixabay.com/">
            <v-card-actions>
              <v-img 
                max-height="5vh" 
                contain 
                src="https://pixabay.com/static/img/public/leaderboard_a.png" 
                alt="Pixabay">
              </v-img>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
      <v-row dense>
        <v-col v-for="image in images" :key="image.id">
          <v-card>
            <v-img 
              :src="image.previewURL" 
              :height="itemHeight" 
              @click="showLargeImage(image.largeImageURL)" 
              class="white--text align-end">
            </v-img>
            <v-card-actions>
              <v-spacer></v-spacer>
                <v-rating dense small></v-rating>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
    <v-card v-if="isLargeImageVisible">
      <v-img 
        max-height="81vh"
        :src="largeImageSource">
        <v-icon 
          dark
          class="mt-3 ml-2"
          @click="backToGallery">mdi-arrow-left</v-icon>
      </v-img>
    </v-card>
  </div>
</template>

<script>
import { EventBus } from '@/EventBus.js'

export default {
  data(){
    return {
      isGalleryVisible: false,
      isSearchBarVisible: true,
      isLargeImageVisible: false,
      images: [],
      largeImageSource: '',
      itemHeight: 0,
    }
  },
  mounted(){
    //Listen for the API response from Search
    //component and display it
    EventBus.$on('value-to-display', itemsFromSearch => {
      this.isGalleryVisible = true
        this.galleryLayoutHandler(itemsFromSearch)
    })
  },
  beforeUpdate(){
    //Clean-up after going back to homescreen
    EventBus.$on('back-to-homepage', () => {
      this.isGalleryVisible = false
      this.isLargeImageVisible = false
      this.images = []
    })
  },
  beforeDestroy(){
    EventBus.$off('value-to-display')
    EventBus.$off('back-to-homepage')
  },
  methods: {
    showLargeImage(bigImage){
      this.largeImageSource = bigImage
      EventBus.$emit('large-image-open')
      this.isGalleryVisible = false
      this.isLargeImageVisible = true
    },
    backToGallery(){
      EventBus.$emit('large-image-closed')
      this.isLargeImageVisible = false
      this.isGalleryVisible = true
    },
    //Fixing the unequal height of displayed
    //components, by setting the height according to
    //the highest image from the API response
    //and handling the one-item-only display situation
    galleryLayoutHandler(imagesArray){
      let heightsArray = []
      let maxValue = 0
      imagesArray.forEach(image => {
        heightsArray.push(image.previewHeight)
        this.images.push({
          previewURL: image.previewURL, 
          largeImageURL: image.largeImageURL
        })
      })
      if(heightsArray.length > 1){
        heightsArray.reduce((previous, next) => {
          return maxValue = Math.max(previous, next)
        })
        this.itemHeight = maxValue
      } else {
        this.itemHeight = heightsArray[0]
      } 
    }
  }
}
</script>
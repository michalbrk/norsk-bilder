<template>
  <div>
    <v-container v-if="isVisibleTextField">
      <v-row justify="center">
        <v-col cols="10" sm="6" md="4" class="mb-0 pb-0">
          <v-dialog v-model="dialog" persistent :fullscreen="$vuetify.breakpoint.xsOnly" max-width="30vw">
            <v-card>
              <v-card-title>
                Please provide the authentication key
              </v-card-title>
              <v-card-actions>
                <v-container>
                  <v-row>
                    <v-col>
                      <v-text-field dense :rules="[rules.match]"></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-actions>
              <v-card-text>*If missing, reach out to gudrunn.andersen@yahoo.com</v-card-text>
            </v-card>
          </v-dialog>
          <v-form v-if="isHiddenBeforeAuth" class="mb-0 pb-0">
            <v-text-field
              v-model="searchValue" 
              @input="mainInputHandler"
              @keypress.enter.prevent
              label="Type something..."
              outlined dense dark></v-text-field>
          </v-form>
        </v-col>
      </v-row>
      <v-row v-if="isVisibleTextError" justify="center">
        <v-col cols="10" sm="6" md="4" class="mt-0 pt-0">
          <v-alert type="warning">
            {{ matchingError }}
          </v-alert>
        </v-col>
      </v-row>
      <v-row v-if="isVisibleNoResultError" justify="center">
          <v-col cols="10" sm="6" md="4" class="mt-0 pt-0">
          <v-alert type="info">
            {{ noResultsFound }}
          </v-alert>
        </v-col>
      </v-row>
      <v-row v-if="isVisibleAPIError" justify="center">
          <v-col cols="10" sm="6" md="4" class="mt-0 pt-0">
          <v-alert type="error">
            {{ warningMessage }}
          </v-alert>
        </v-col>
      </v-row>
    </v-container> 
  </div>
</template>

<script>
import axios from 'axios'
import debounce from 'lodash/debounce'
import { EventBus } from '@/EventBus.js'
import { Attractions } from '@/Data.js'

export default {
  data(){
    return {
      dialog: null,
      searchValue: '',
      warningMessage: 'Ops! Our bad, try again later. ;)',
      matchingError: 'Not related to Norway...',
      noResultsFound: 'Ops! No result for this, sorry...',
      APIKey: '12945024-089af957dfe72c50765f6cd0e',
      isHiddenBeforeAuth: false,
      isVisibleTextField: true,
      isVisibleTextError: false,
      isVisibleAPIError: false,
      isVisibleNoResultError: false,
      rules: {
        match: authValue => {
          if(String(authValue) == this.APIKey){
            this.isHiddenBeforeAuth = true
            this.dialog = false
            return true
          } else{
            return 'Invalid API Key'
          }
        }
      }
    }
  },
  mounted(){
    this.dialog = true
  },
  beforeUpdate(){
    //receiving event from the gallery component,
    //when the gallery is being closed
    EventBus.$on('back-to-homepage', () => {
      if(this.isVisibleTextError){
        this.isVisibleTextError = false
      } else if(this.isVisibleNoResultError){
        this.isVisibleNoResultError = false
      }
      this.isVisibleTextField = true
      this.searchValue = ''
    })
  },
  beforeDestroy(){
    EventBus.$off('back-to-homepage')
    localStorage.clear()
  },
  methods: {
    mainInputHandler: debounce(async function bounced(){
      //check if search input matches any term from Attractions file or not,
      //regardless, store the value
      let matchedSearchTerm = this.matching(this.searchValue, Attractions, this.matchingError)
      //handle match situation
      if(matchedSearchTerm !== this.matchingError){
        //if the searched term is different than the error message
        //check the local storage for the given value
        if(!JSON.parse(localStorage.getItem(`${this.searchValue}`))){
          //If the value is not found, perform the API request
          let galleryItems = await this.getImage(matchedSearchTerm)
          if(galleryItems.data.hits.length <= 0){
          //handling the no-items-found from the API situation
          //otherwise pass the result values to gallery component
            this.isVisibleTextError = false
            this.isVisibleNoResultError = true
          } else {
            this.isVisibleTextField = false
            EventBus.$emit('blur-background')
            EventBus.$emit('value-to-display', galleryItems.data.hits)
            //and store the data inside the local storage
            localStorage.setItem(`${this.searchValue}`, JSON.stringify(galleryItems.data.hits))
          }
        } else {
          //get the searched data from the local storage
          this.isVisibleTextField = false
          EventBus.$emit('blur-background')
          EventBus.$emit('value-to-display', JSON.parse(localStorage.getItem(`${this.searchValue}`)))
        }
        //handle no-match situation
      } else if(this.searchValue.length !== 0){
        //toggling error messages depending on
        //the current state of the text field
        this.isVisibleNoResultError = false
        this.isVisibleTextError = true
      } else {
        this.isVisibleTextError = false
        this.isVisibleNoResultError = false
      }
    }, 1000),
    getImage(matchedValue){
      //get requested images from the API or inform about the error
      let URL = `https://pixabay.com/api/?key=${this.APIKey}&q=${matchedValue}&image_type=photo`
      let responsefromAPI = null
      try {
        responsefromAPI = axios.get(URL)
      } catch (error) {
        this.isVisibleAPIError = true
        this.errorFromAPI = error
      }
      return responsefromAPI
    },
    matching(inputValue, dataValue, errorValue){
      //check if searched term matches any value from Attractions file
      return dataValue.includes(inputValue.toLowerCase()) ? inputValue : errorValue
    }
  }
}
</script>

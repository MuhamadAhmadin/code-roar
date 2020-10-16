<template>
  <section class="section__input-kode-baru">
      <Header></Header>
  
  
    <div class="w-full flex flex-col md:flex-row justify-between">
      
      <div class="w-full flex flex-col md:flex-row mb-5">
        <div class="flex flex-col justify-between mb-3 md:mb-0">
          <div class="flex flex-col">
            <label for="file-name" class="text-gray-100">Filename</label>
            <Input class="mb-3" placeholder="somename" id="file-name" :value="data.fileName"
                   @getvalue="(val) => data.fileName = val" 
            />
          </div>
          <div class="flex flex-col ">
            <Select
              full 
              name="Programming Language" 
              :selected="data.lang" 
              :options="option_langs" 
              @getvalue="(val) => data.lang = val"
            />
          </div>
          
        </div>
        
        <div class="flex flex-col ml-0 md:ml-5  justify-between">
          <div class="flex flex-col">
            <label for="file-name" class="text-gray-100">Highlighted Code (Line number)</label>
            <Input class="mb-3" placeholder="eg: 1, 1-3, 2-6" :value="data.highlight"
                   @getvalue="(val) => data.highlight = val"
            />
          </div>

          <div class="flex">
            <Button class="mr-3" color="red" @click="resetInput">Reset</Button>
            <Button class="mr-3" color="green" @click="downloadCode">Download</Button>
            <Button  color="blue" @click="simpanCode">Save to my list</Button>
          </div>
        </div>
      </div>
    </div>
    <div class="w-full flex flex-col md:flex-row justify-between">
      <textarea class=" p-3 text-gray-800 resize-none overflow-auto w-full md:w-1/2 mb-3 md:mb-0 md:mr-3 max-h"
                placeholder="Paste the code here" v-model="data.code" cols="50" rows="12"></textarea>
      <!-- code yang di highlight -->
      <div v-html="codeHighlight"
           class="w-full md:w-1/2 max-h p-3 bg-gray-900 rounded text-left overflow-auto break-all">
      </div>
    </div>
  </section>
</template>

<script>
import Select from '@/components/Select';
import Button from '@/components/Button';
import Input from '@/components/Input';
import Header from '@/views/section/Header';

import debounce from 'debounce-fn';

export default {
  components: {
    Header,
    Select,
    Button,
    Input
  },
  data(){
    return {
      data: {
        code: '',
        lang: '',
        fileName: '',
        highlight: ''
      },
      codeHighlight: '',
      option_langs: []
    }
  },
  async created(){
    try{
      const langs = await this.$store.dispatch('getLangs')
      this.option_langs = langs
    } catch(err){
      this.$store.dispatch('showNotification', {message: err.message, mode: 'danger'})
    }
  },
  computed: {
    userId(){
      return this.$store.state.user.id
    }
  },
  watch: {
    data: {
      handler: debounce(async function () {
           await this.hightlightingCode()
         }, {wait: 300}),
      deep:true
    }
  },
  methods: {
    async hightlightingCode(){
      if(this.data.code && this.data.lang) {
        try{
          this.codeHighlight = await this.$store.dispatch('hasilkanKode', this.data)
        } catch(err) {
          this.$store.dispatch('showNotification', {message: err.message, mode: 'danger'})
        }
      } else {
        this.codeHighlight = ''
      }
    },
    downloadCode(){
      this.$store.dispatch('downloadCode', this.data)
    },
    async simpanCode(){
      let data = {...this.data}
      delete data.download
      if(!data.highlight) {
        delete data.highlight
      }
      if(!data.fileName) {
        data.fileName = 'example_code'
      }
       this.$store.dispatch('simpanKode', {user: this.userId, content: data})
    },
    resetInput() {
      this.data = {
        code: '',
        lang: '',
        fileName: '',
        highlight: ''
      }
    }
  }
}
</script>

<style scoped lang="postcss">
  .max-h{
    max-height: 400px;
    min-height: 400px;
  }

  textarea {
    @apply border rounded
  }
  textarea:focus {
    @apply outline-none border-blue-500
  }

  .section__input-kode-baru{
    @apply bg-gradient-to-br from-yellow-700 via-red-800 to-orange-900 p-5 pt-3 flex flex-col border-b-2 min-h-screen justify-between
  }
</style>


<template>
  <div class="mt-0 flex justify-end  w-full border-b-2 pb-3 px-4 relative mb-4">
    
    <Button class="mr-3 mb-4 mt-5 px-5" color="red" @click="logout">Logout</Button>
  </div>
</template>

<script>
import Button from '@/components/Button';
export default {
  components: {
    Button
  },
  computed: {
    username(){
      return this.$store.state.user.name
    }
  },
  methods: {
    logout(){
      let confirm = window.confirm('Are you sure to logout??')
      if(confirm) {
        this.$store.commit('getUserData', {})
        localStorage.removeItem('user')
      }
    }
  }
}
</script>
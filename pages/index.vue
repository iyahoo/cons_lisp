<template>
   <section class="container">
      <div>
         <v-textarea
            v-model="exp"
            auto-grow
            box
            color="deep-purple"
            label="input"
            rows="1"
            id="inputArea"
         />
         <v-text-field :value="output" label="Output" outline readonly />
      </div>
   </section>
</template>

<script>
export default {
   methods: {
      setCursor: val => {
         const inputArea = document.getElementById('inputArea')
         inputArea.setSelectionRange(val, val)
      },
   },
   computed: {
      output() {
         if (this.exp === '(') {
            this.exp = '()'
         }

         if (this.exp === '()') {
            this.setCursor(1)
         }

         return this.exp
      },
   },
   data: () => ({
      exp: '',
   }),
}
</script>

<style></style>

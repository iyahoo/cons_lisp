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
         <v-text-field :value="output()" label="Output" outline readonly />
      </div>
   </section>
</template>

<script>
export default {
   methods: {
      setCursor(val) {
         const inputArea = document.getElementById('inputArea')
         inputArea.setSelectionRange(val, val)
      },

      checkExp(exp) {
         if (exp === '()') {
            this.setCursor(1)
         }
      },

      output() {
         if (this.exp === '(') {
            this.exp = '()'
         }

         return this.exp
      },
   },

	updated () {
		this.checkExp(this.exp)
	},

   data: () => ({
      exp: '',
   }),
}
</script>

<style></style>

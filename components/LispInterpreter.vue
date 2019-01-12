<template>
   <div>
      <v-textarea
         v-model="inputstr"
         auto-grow
         box
         color="deep-purple"
         label="input"
         rows="1"
         id="inputArea"
      />
      <v-textarea :value="trySynAna" label="Parsed" outline readonly />
   </div>
</template>

<script>
export default {
	methods: {
		syntaxAnalyzer()
	},
   computed: {
      trySynAna() {
         return this.inputstr
      },
   },
   data() {
      return {
         inputstr: '',
      }
   },
}
</script>

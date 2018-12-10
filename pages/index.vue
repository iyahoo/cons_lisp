<template>
   <section class="container">
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
         <v-text-field :value="eval" label="Output" outline readonly />
         <v-text-field
            :value="parsed(parser({ parsed: null, rest: this.inputstr }))"
            label="test"
            outline
            readonly
         />
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

      // perser
      pNumber({ parsed, rest }) {
         const r = /(^[1-9]+)(.*)/
         const result = rest.match(r)
         if (!result) {
            return { parsed: parsed, rest: rest }
         } else {
            return { parsed: parseInt(result[1]), rest: result[2] }
         }
      },
      parser(pd) {
         const result = this.pNumber(pd)
         if (result.parsed === null) {
            return false
         } else {
            return result
         }
      },
      parsed(pd) {
         return pd.parsed
      },
   },
   computed: {
      eval() {
         const result = this.parser({ parsed: null, rest: this.inputstr })
         if (!result) {
            return 'Parser error!'
         }

         const { parsed } = result
         return parsed
      },
   },

   data: () => ({
      inputstr: '',
   }),
}
</script>

<style></style>

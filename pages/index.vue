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
         <v-layout row wrap>
            <v-flex xs12 md6 v-for="(t, i) in testings" :key="i">
               <v-text-field
                  :value="JSON.stringify(t.function(t.input))"
                  :label="testToStr(t)"
                  :background-color="testToColor(t)"
                  outline
                  readonly
               />
            </v-flex>
         </v-layout>
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

      // test
      testToStr(t) {
         return 'expect: ' + JSON.stringify(t.expect)
      },

      testToColor(t) {
         const actual = t.function(t.input)

         return this.objectEquality(actual, t.function(t.input))
            ? 'green'
            : 'red'
      },

      objectEquality(a, b) {
         // only for unnested object
         let r = true
         for (const k in a) {
            r &= a[k] === b[k]
         }
         return r
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

   data() {
      return {
         inputstr: '',

         // test
         testings: [
            {
               input: { parsed: null, rest: '33' },
               function: this.pNumber,
               expect: { parsed: 33, rest: '' },
            },
            {
               input: { parsed: null, rest: 'hoge' },
               function: this.pNumber,
               expect: { parsed: null, rest: 'hoge' },
            },
         ],
      }
   },
}
</script>

<style></style>

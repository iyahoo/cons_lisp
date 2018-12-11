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
         <v-text-field :value="tryParser" label="Parsed" outline readonly />
         <h3>Tests</h3>
         <v-layout row wrap>
            <v-flex xs12 md6 v-for="(t, i) in testings" :key="i">
               <h4>{{ t.name }}</h4>
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
      pSpace({ parsed, rest }) {
         const r = /\s+(.*)/
         const result = rest.match(r)
         if (!result) {
            return { parsed: parsed, rest: rest }
         } else {
            return { parsed: parsed, rest: result[1] }
         }
      },
      pNumber({ parsed, rest }) {
         const r = /(^[1-9]+)(.*)/
         const result = rest.match(r)
         if (!result) {
            return { parsed: parsed, rest: rest }
         } else {
            if (parsed === null) {
               return { parsed: parseInt(result[1]), rest: result[2] }
            } else {
               return parser({
                  parsed: parsed.concat(parseInt(result[1])),
                  rest: result[2],
               })
            }
         }
      },
      pList({ parsed, rest }) {
         const r = /^\((.+)\)/
         const result = rest.match(r)
         if (!result) {
            return { parsed: parsed, rest: rest }
         } else {
            return { parsed: parsed, rest: rest }
         }
      },
      parser(pd) {
         const result = this.pList(this.pNumber(this.pSpace(pd)))
         if (result.parsed === null) {
            throw new Error('parser error')
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

         return this.objectEquality(actual, t.expect) ? 'green' : 'red'
      },

      objectEquality(a, b) {
         // only for unnested object

         if (a === b) {
            return true
         }
         if (typeof a === 'object' && typeof b === 'object') {
            let r = true
            for (const k in a) {
               r &= a[k] === b[k]
            }
            return r
         }
         return false
      },
   },
   computed: {
      tryParser() {
         try {
            const result = this.parser({ parsed: null, rest: this.inputstr })
            const { parsed } = result
            return parsed
         } catch (e) {
            return e
         }
      },
   },

   data() {
      return {
         inputstr: '',

         // test
         testings: [
            {
               name: 'pNumber1',
               input: { parsed: null, rest: '33' },
               function: this.pNumber,
               expect: { parsed: 33, rest: '' },
            },
            {
               name: 'pNumber2',
               input: { parsed: null, rest: 'hoge' },
               function: this.pNumber,
               expect: { parsed: null, rest: 'hoge' },
            },
            {
               name: 'pSpace1',
               input: { parsed: null, rest: '   ' },
               function: this.pSpace,
               expect: { parsed: null, rest: '' },
            },
            {
               name: 'pSpace2',
               input: { parsed: null, rest: '   hoge' },
               function: this.pSpace,
               expect: { parsed: null, rest: 'hoge' },
            },
            {
               name: 'check equality of null and empty string',
               input: null === '',
               function: x => x,
               expect: false,
            },
         ],
      }
   },
}
</script>

<style></style>

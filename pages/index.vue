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
         <v-layout row wrap v-if="runTest">
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
         const r = /^\s+(.*)/
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
               return this.parser({
                  parsed: parsed.concat([parseInt(result[1])]),
                  rest: result[2],
               })
            }
         }
      },
      pList({ parsed, rest }) {
         const r = /^\((.+)\)(.*)/
         const result = rest.match(r)
         if (!result) {
            return { parsed: parsed, rest: rest }
         } else {
            const inList = this.parser({
               parsed: [],
               rest: result[1],
            })
            if (parsed === null) {
               return { parsed: inList.parsed, rest: inList.rest + result[2] }
            } else {
               return {
                  parsed: parsed.concat(inList.parsed),
                  rest: inList.rest + result[2],
               }
            }
         }
      },
      parser(pd) {
         const result = this.pList(this.pNumber(this.pSpace(pd)))
         if (result.parsed === null || result.rest !== '') {
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
         if (Array.isArray(a) && Array.isArray(b)) {
            if (a === b) return true
            if (a == null || b == null) return false
            if (a.length != b.length) return false

            for (var i = 0; i < a.length; ++i) {
               if (a[i] !== b[i]) return false
            }
            return true
         }
         if (typeof a === 'object' && typeof b === 'object') {
            let r = true
            for (const k in a) {
               r &= this.objectEquality(a[k], b[k])
            }
            return r
         }

         return false
      },
   },
   computed: {
      tryParser() {
         try {
            const input = this.inputstr
            const result = this.parser({ parsed: null, rest: input })
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
         runTest: true,
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
            {
               name: 'check equality of arrays',
               input: [1, 2, 3] === [1, 2, 3],
               function: x => x,
               expect: false,
            },
            {
               name: 'check Array.isArray',
               input: Array.isArray([1, 2, 3]) && Array.isArray([1, 2, 3]),
               function: x => x,
               expect: true,
            },

            {
               name: 'pList1',
               input: { parsed: null, rest: '(1 2 3)' },
               function: this.pList,
               expect: { parsed: [1, 2, 3], rest: '' },
            },
         ],
      }
   },
}
</script>

<style></style>

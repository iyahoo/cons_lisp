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
      <v-text-field :value="tryParser" label="Parsed" outline readonly />
   </div>
</template>

<script>
export default {
   methods: {
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
               return {
                  parsed: { type: 'integer', value: result[1] },
                  rest: result[2],
               }
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
                  parsed: parsed.concat([inList.parsed]),
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

      objectEquality(a, b) {
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
            return r ? true : false
         }

         return false
      },
   },
   computed: {
      tryParser() {
         try {
            const input = this.inputstr
            const result = this.parser({ parsed: null, rest: input })
            return result.parsed
         } catch (e) {
            return e
         }
      },
   },
   data() {
      return {
         inputstr: '',
      }
   },
}
</script>

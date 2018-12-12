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
      <v-textarea :value="tryParser" label="Parsed" outline readonly />
   </div>
</template>

<script>
export default {
   methods: {
      pSpace(context) {
         const r = /^\s+(.*)/
         const result = context.rest.match(r)
         if (!result) {
            return context
         } else {
            context.rest = result[1]
            return context
         }
      },
      pNumber(context) {
         const r = /(^[1-9]+)(.*)/
         const result = context.rest.match(r)
         if (!result) {
            return context
         } else {
            const ast = { type: 'integer', value: result[1] }
            if (context.parsed === null) {
               context.parsed = ast
               context.rest = result[2]
               return context
            } else {
               context.parsed = context.parsed.concat([ast])
               context.rest = result[2]
               return this.parser(context)
            }
         }
      },
      pList(context) {
         const r = /^\((.+)\)(.*)/
         //const r = /^\((.+)/
         const result = context.rest.match(r)
         if (!result) {
            return context
         } else {
            let innerContext = { parsed: [], rest: result[1] }
            this.parser(innerContext)
            if (context.parsed === null) {
               context = innerContext
               context.rest = context.rest + result[2]
               return context
            } else {
               context.parsed = context.parsed.concat([innerContext.parsed])
               context.rest = context.rest + result[2]
               return context
            }
         }
      },
      parser(context) {
         const result = this.pList(this.pNumber(this.pSpace(context)))

         if (result.parsed === null || result.rest !== '') {
            throw new Error('parser error')
         } else {
            return result
         }
      },

      parsed(context) {
         return context.parsed
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
               if (!this.objectEquality(a[i], b[i])) return false
            }
            return true
         }
         if (typeof a === 'object' && typeof b === 'object') {
            for (let k in a) {
               if (!this.objectEquality(a[k], b[k])) return false
            }
            return true
         }

         return false
      },
   },
   computed: {
      tryParser() {
         try {
            const input = this.inputstr
            let context = { parsed: null, rest: input }
            const result = this.parser(context)
            return JSON.stringify(result.parsed)
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

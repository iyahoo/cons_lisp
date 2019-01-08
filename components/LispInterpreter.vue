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
            return {
               parsed: context.parsed,
               rest: result[1],
               depth: context.depth,
            }
         }
      },
      pNumber(context) {
         const r = /(^[0-9]+)(.*)/
         const result = context.rest.match(r)
         if (!result) {
            return context
         } else {
            const ast = { type: 'integer', value: result[1] }
            if (context.parsed === null) {
               return this.parser({
                  parsed: ast,
                  rest: result[2],
                  depth: context.depth,
               })
            } else {
               return this.parser({
                  parsed: context.parsed.concat([ast]),
                  rest: result[2],
                  depth: context.depth,
               })
            }
         }
      },
      pList(context) {
         console.log('pList')
         console.table(context)
         const r = /^\((.*)/
         let headResult = context.rest.match(r)
         if (headResult) {
            const innerContext = {
               parsed: [],
               rest: headResult[1],
               depth: context.depth + 1,
            }
            const innerContextParsed = this.parser(innerContext)
            if (context.parsed === null) {
               return this.parser(innerContextParsed)
            } else {
               return this.parser({
                  parsed: context.parsed.concat([innerContextParsed.parsed]),
                  rest: innerContextParsed.rest,
                  depth: innerContextParsed.depth,
               })
            }
         } else if (!headResult && context.depth !== 0) {
            console.log('in else if')
            console.table(context)
            const r = /^\)(.*)/
            let endResult = context.rest.match(r)

            if (!endResult) {
               if (context.rest !== '') {
                  return context
               } else {
                  throw new Error('parser error: no match brackets!!')
               }
            } else {
               console.log('in else if else')
               const resultContext = {
                  parsed: context.parsed,
                  rest: endResult[1],
                  depth: context.depth - 1,
               }
               console.log('resultContext')
               console.table(resultContext)
               return resultContext
            }
         } else {
            return context
         }
      },
      parser(context) {
         if (context.parsed !== null && context.rest === '') {
            return context
         }

         let holder = context
         holder = this.pSpace(holder)
         holder = this.pNumber(holder)
         holder = this.pList(holder)
         const result = holder

         return result
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
            console.log('tryParser')
            const input = this.inputstr
            const context = { parsed: null, rest: input, depth: 0 }
            const result = this.parser(context)
            if (result.parsed === null) {
               throw new Error('parser error')
            } else if (result.rest !== '') {
               throw new Error(
                  'parser error: unexpected token ' + result.rest[0]
               )
            } else if (result.depth !== 0) {
               throw new Error('parser error: no match brackets')
            }
            return JSON.stringify(result.parsed)
         } catch (e) {
            return e
         }
      },
   },
   data() {
      return {
         onDebug: true,
         inputstr: '',
      }
   },
}
</script>

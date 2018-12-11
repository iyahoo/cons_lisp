<template>
   <v-expansion-panel v-model="exp" dark>
      <v-expansion-panel-content>
         <div slot="header">Test</div>
         <v-layout row wrap>
            <v-flex xs12 md12 v-for="(t, i) in testings" :key="i" class="px-2">
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
      </v-expansion-panel-content>
   </v-expansion-panel>
</template>

<script>
import LispInterpreter from '~/components/LispInterpreter'

export default {
   mixins: [LispInterpreter],
   methods: {
      testToStr(t) {
         return 'expect: ' + JSON.stringify(t.expect)
      },

      testResult(t) {
         const actual = t.function(t.input)
         const result = this.objectEquality(actual, t.expect)
         if (!result) {
            console.log(t.name)
            console.log('actual')
            console.log(actual)
            console.log('expect')
            console.log(JSON.stringify(t.expect))
         }
         return result
      },

      testToColor(t) {
         return this.testResult(t) ? 'green' : 'red'
      },
   },
   data() {
      return {
         exp: [true],
         testings: [
            {
               name: 'objectEquality',
               input: { parsed: { hoge: 1, fuga: 2 }, rest: '33' },
               function: x =>
                  this.objectEquality(x, {
                     parsed: { hoge: 1, fuga: 2 },
                     rest: '33',
                  }),
               expect: true,
            },
            {
               name: 'objectEquality2',
               input: { parsed: { hoge: 1, fuga: 2 }, rest: '33' },
               function: x =>
                  this.objectEquality(x, {
                     parsed: { hoge: 1, fuga: 2 },
                     rest: '33',
                  }),
               expect: true,
            },
            {
               name: 'pNumber1',
               input: { parsed: null, rest: '33' },
               function: this.pNumber,
               expect: { parsed: { type: 'integer', value: '33' }, rest: '' },
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
               expect: {
                  parsed: [
                     { type: 'integer', value: '1' },
                     { type: 'integer', value: '2' },
                     { type: 'integer', value: '3' },
                  ],
                  rest: '',
               },
            },
            {
               name: 'pList2',
               input: { parsed: null, rest: '(2)' },
               function: this.pList,
               expect: {
                  parsed: [{ type: 'integer', value: '2' }],
                  rest: '',
               },
            },
         ],
      }
   },
}
</script>

<style></style>

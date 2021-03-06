<template>
   <v-expansion-panel v-model="exp" dark>
      <v-expansion-panel-content>
         <div slot="header">Test</div>
         <v-layout row wrap>
            <v-flex xs12 md12 v-for="(t, i) in testings" :key="i" class="px-2">
               <v-textarea
                  v-if="!testResult(t)"
                  :value="testToStr(t)"
                  :label="t.name"
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
         let expect = t.expect
         let input = JSON.stringify(t.input)
         if (!(expect instanceof Error) || typeof t.expect === 'object') {
            expect = JSON.stringify(t.expect)
         }
         try {
            return (
               'input: ' +
               input +
               '\n' +
               'expect: ' +
               expect +
               '\n' +
               'actual: ' +
               JSON.stringify(t.function(t.input))
            )
         } catch (e) {
            return 'input: ' + input + '\nexpect: ' + expect + '\nactual: ' + e
         }
      },

      testResult(t) {
         try {
            const actual = t.function(t.input)
            const result = this.objectEquality(actual, t.expect)
            return result
         } catch (e) {
            return false
         }
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
               input: { parsed: null, rest: '(1 2 3)', depth: 0 },
               function: this.pList,
               expect: {
                  parsed: [
                     { type: 'integer', value: '1' },
                     { type: 'integer', value: '2' },
                     { type: 'integer', value: '3' },
                  ],
                  rest: '',
                  depth: 0,
               },
            },
            {
               name: 'pList2',
               input: { parsed: null, rest: '(2)', depth: 0 },
               function: this.pList,
               expect: {
                  parsed: [{ type: 'integer', value: '2' }],
                  rest: '',
                  depth: 0,
               },
            },
            {
               name: 'pList3',
               input: { parsed: null, rest: '(2 (3) (3))', depth: 0 },
               function: this.pList,
               expect: {
                  parsed: [
                     { type: 'integer', value: '2' },
                     [{ type: 'integer', value: '3' }],
                     [{ type: 'integer', value: '3' }],
                  ],
                  rest: '',
                  depth: 0,
               },
            },
            {
               name: 'pList4',
               input: { parsed: null, rest: '(1 (2 (3)) (4))', depth: 0 },
               function: this.pList,
               expect: {
                  parsed: [
                     { type: 'integer', value: '1' },
                     [
                        { type: 'integer', value: '2' },
                        [{ type: 'integer', value: '3' }],
                     ],
                     [{ type: 'integer', value: '4' }],
                  ],
                  rest: '',
                  depth: 0,
               },
            },
            {
               name: 'pList5',
               input: { parsed: null, rest: '(1 (2)())', depth: 0 },
               function: this.pList,
               expect: {
                  parsed: [
                     { type: 'integer', value: '1' },
                     [{ type: 'integer', value: '2' }],
                     [],
                  ],
                  rest: '',
                  depth: 0,
               },
            },
            {
               name: 'pList4 Error',
               input: { parsed: null, rest: '(2', depth: 0 },
               function: this.pList,
               expect: Error('perser error: no match brackets'),
            },
         ],
      }
   },
}
</script>

<style></style>

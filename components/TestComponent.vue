<template>
   <v-expansion-panel dark>
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
      // test
      testToStr(t) {
         return 'expect: ' + JSON.stringify(t.expect)
      },

      testToColor(t) {
         const actual = t.function(t.input)
         return this.objectEquality(actual, t.expect) ? 'green' : 'red'
      },
   },
   data() {
      return {
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

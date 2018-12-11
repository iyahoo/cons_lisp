<template>
   <section class="container">
      <div>
         <lisp-interpreter />
         <test-component v-if="runTest" />
      </div>
   </section>
</template>

<script>
import LispInterpreter from '~/components/LispInterpreter'
import TestComponent from '~/components/TestComponent'

export default {
   components: {
      LispInterpreter,
      TestComponent,
   },
   data() {
      return {
         runTest: true,
      }
   },
}
</script>

<style></style>

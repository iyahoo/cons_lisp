<template>
   <section class="container">
      <div>
         <v-textarea
            v-model="exp"
            auto-grow
            box
            color="deep-purple"
            label="input"
            rows="1"
         />
      </div>
   </section>
</template>

<script>
export default {
   data: () => ({
      exp: '(+ 1 1)',
   }),
}
</script>

<style></style>

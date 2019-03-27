<template>
  <div class="message-box alert alert-info">
    {{ message }}
  </div>
</template>

<script>
export default {
  name: 'MessageBox',
  props: {
    message: String
  }
}
</script>

<style scoped>
  /* .message-box {
    margin-left: 2rem;
    margin-right: 2rem;
  } */
</style>



<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>
<script>
  import NavMenu from '../components/NavMenu'
  export default {
    name: 'app',
    components: {
      NavMenu
    }
  }

</script>>
<style>
  body {
    margin: 0;
    font-size: 2rem;
    font-family: -apple-system, BlinkMacSystemFont,
    'avenir next', avenir,
    helvetica, 'helvetica neue',
    Ubuntu,
    'segoe ui', arial,
    sans-serif;
  }

  .page {
    /* nesting for the need to test postcss */
    code {
      background-color: #f0f0f0;
      padding: 3px 5px;
      border-radius: 2px;
    }
  }

</style>

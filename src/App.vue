<template>
  <div id="app">
    <Header />
    <Main />
  </div>
</template>

<script>
import Header from './components/Header.vue'
import Main from './components/Main.vue'
import './styles/application.scss'

export default {
  name: 'App',
  components: {
    Header,
    Main
  }
}
</script>


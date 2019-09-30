<template>
<div id="footer">
  <v-app id="inspire">
    <v-footer height="auto" color="green">
      <v-layout justify-center row wrap>
        <v-btn to="/" color="white" flat round exact>Home</v-btn>
        <v-btn to="/upload" color="white" flat round exact>Upload</v-btn>
        <v-btn to="/research" color="white" flat round exact>Research</v-btn>
        <v-btn to="/employees" color="white" flat round exact>Employees</v-btn>
        <v-btn to="/samples" color="white" flat round exact>Samples</v-btn>
      </v-layout>
    </v-footer>
    <router-view></router-view>
  </v-app>
</div>
</template>

<script>
import Vue from 'vue'
import VueRouter from 'vue-router'

import Home from './components/Home'
import Upload from './components/Upload'
import Research from './components/Research'
import Employees from './components/Employees'
import Samples from './components/Samples'

Vue.use(VueRouter)

const router = new VueRouter({
  routes: [{
      path: '/',
      component: Home
    },
    {
      path: '/upload',
      component: Upload
    },
    {
      path: '/research',
      component: Research
    },
    {
      path: '/employees',
      component: Employees
    },
    {
      path: '/samples',
      component: Samples
    }
  ]
})

export default {
  router: router
}
</script>

<template>
  <v-row justify="center" align="center" :class="['text-h1']"
    >
  <v-card
    :loading="loading"
    class="mx-auto my-12"
    max-width="1000"
  >
    <template slot="progress">
      <v-progress-linear
        color="deep-purple"
        height="10"
        indeterminate
      ></v-progress-linear>
    </template>

    <v-img
      height="250"
      :src="item.openGraphImageUrl"
    ></v-img>

    <v-card-title>{{item.name}}</v-card-title>

    <v-card-text>



      <div>{{item.description}}</div>
    </v-card-text>

    <v-divider class="mx-4"></v-divider>

    <v-card-title>$ {{item.amount}}</v-card-title>



    <v-card-actions>
      <v-btn
        color="deep-purple lighten-2"
        text
        :disabled="disabled"
        @click="buyNow(item.repo_id,item.username,item.name)"
      >
        BUY NOW
      </v-btn>
    </v-card-actions>
  </v-card>
</v-row>
</template>
<script lang="ts">
import Cookies from 'js-cookie'
import { Component, Vue } from 'nuxt-property-decorator'
import {configs} from '@/utils/configs.js'
import axios from 'axios'
@Component
export default class MyStore extends Vue {
public disabled:boolean=false
mounted(){
  Cookies.remove("redirect")
}
  async asyncData({ params }:any) {

      const username = params.username
      const name = params.name
      const url=configs.repo_detail_url
      const {data:item}=await axios.post(`${url}`,{username,name})
      return { item }
    }
async buyNow(id:any,username:string,name:string) {

    this.disabled=true;
    const token = Cookies.get('token')
    if(!token){
      window.location.href="/login"
       Cookies.set('redirect', `/${username}/${name}`)
      return
    }
    Cookies.remove("redirect")
    const url=<string>configs.buy_paypal_url;
    const res = await this.$axios.post(url, { id, token })
    if (res.data) {
      const link=res.data.result.links.filter((a:any)=>a.rel==="approve")[0].href
      window.location.href=link;
    }else{
      this.disabled=false;
      alert("error")
    }
  }
}
</script>


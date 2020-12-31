<template>
  <v-container>
    <v-row justify="space-between">
      <v-col cols="4" class="text-h4 font-weight-bold">Dashboard</v-col>
      <v-col cols="4" class="d-flex justify-end"
        ><v-btn @click="logout">LOGOUT</v-btn></v-col
      >
    </v-row>
    <DashboardInfo
    :disabled="disabled"
      :username="username"
      :profilePhoto="profilePhoto"
      :paypalToken="paypalToken"
      :paypalBalance="paypalBalance"
      @disconnect="disconnectPaypal"
      @connect="connectPaypal"
    />
    <v-row justify="space-between">
      <v-col cols="4" class="text-h4 font-weight-bold">For Sell Repo List</v-col>
    </v-row>
    <DashboardForSell
    :disabled="disabled"
    :ownedRepo="false"
    :amountRefresh="amountRefresh"
      @get-all-repo="getAllRepo"
      :listRepo="privateRepo"
      :paypalToken="paypalToken"
      @sell-repo="sellRepo($event._id, $event.amount)"
      @unlist-repo="unlistRepo($event._id)"
    />
    <v-row justify="space-between">
      <v-col cols="4" class="text-h4 font-weight-bold">Owned Repo</v-col>
    </v-row>
    <DashboardOwned :amountRefresh="amountRefresh" :disabled="disabled" :ownedRepo="true" :paypalToken="paypalToken" :listRepo="ownedRepo" />
    <DashboardRepoAll :disabled="disabled" @get-all-repo="getAllRepo($event.after, $event.before)" :pageInfo="pageInfo" :dialog="dialog" :allRepo="allRepo" />
    <v-snackbar v-model="snackbarAmount">
      the amount shouldn't be negative or zero
    </v-snackbar>
  </v-container>
</template>

<script lang="ts">
import Cookies from 'js-cookie'
import DashboardInfo from '@/components/dashboard_components/DashboardInfo.vue'
import DashboardForSell from '@/components/dashboard_components/DashboardForSell.vue'
import DashboardOwned from '@/components/dashboard_components/DashboardOwned.vue'
import DashboardRepoAll from '@/components/dashboard_components/DashboardRepoAll.vue'
import { Component, Vue, Prop } from 'nuxt-property-decorator'
import {configs} from '@/utils/configs.js'
@Component({
  components: {
    DashboardInfo,
    DashboardForSell,
    DashboardOwned,
        DashboardRepoAll,
  },
})
export default class MyStore extends Vue {
  @Prop({ required: true }) readonly login!: boolean
  public dialog: boolean=false
  public username: string = ''
  public profilePhoto: string = ''
  public snackbarAmount: boolean = false
  public paypalToken: boolean = false
  public privateRepo: Array<object> = []
  public allRepo: Array<object> = []
  public ownedRepo: Array<object> = []
  public paypalBalance: number = 0
  public amountRefresh:number=0
  public disabled:boolean=false;
  public pageInfo:object={};
  logout() {
    Cookies.remove('token')
    window.location.href = '/'
  }
  async created() {
    // if (!this.login) window.location.href = '/'
    this.disabled=true
    const token = Cookies.get('token')
    const url = configs.get_profile_url
    const privateRepoUrl = configs.get_for_sell_url
    const ownedRepoUrl = configs.get_owned_repo_url
    const getPaypalUrl = configs.get_paypal_url
    const profile = await this.$axios.get(`${url}?token=${token}`)
    this.username = profile.data.data.login
    this.profilePhoto = profile.data.data.avatarUrl
    const privateRepo = await this.$axios.get(
      `${privateRepoUrl}?token=${token}`
    )
    this.privateRepo = privateRepo.data.data
    const ownedRepo = await this.$axios.get(`${ownedRepoUrl}?token=${token}`)
    this.ownedRepo = ownedRepo.data.data
    const paypalToken = await this.$axios.get(`${getPaypalUrl}?token=${token}`)

    if (paypalToken.data.status&&!paypalToken.data.data.disconnect) {
      this.paypalToken = true
      this.paypalBalance = paypalToken.data.data.amount
    }
this.disabled=false




  }
  async getAllRepo(after:string,before:string):Promise<void> {
    this.disabled=true
    const token = Cookies.get('token')
    const url = configs.get_all_repo_url

    if(this.allRepo.length===0){
const { data } = await this.$axios.get(`${url}?token=${token}`)
this.allRepo = data.data.nodes
this.pageInfo= data.data.pageInfo
    }else{
const { data } = await this.$axios.get(`${url}?token=${token}${before?`&before=${before}`:""}${after?`&after=${after}`:""}`)
this.allRepo = data.data.nodes
this.pageInfo= data.data.pageInfo
    }



    this.dialog=true
    this.disabled=false
  }
  async sellRepo(_id: string, amount: number) {
    // console.log(amount)
    this.disabled=true
    if (Number(amount) === 0 || Number(amount) < 0 || !amount) {
      console.log(amount)
      this.snackbarAmount = true
      this.disabled=false
      return
    } else {
      const token = Cookies.get('token')
      const url = configs.sell_repo_url

      const { data } = await this.$axios.post(`${url}?token=${token}`, {
        _id,
        amount,
      })
      this.amountRefresh=0;
      this.privateRepo = data.data
    }
    this.disabled=false
  }
  async unlistRepo(_id: string) {
    this.disabled=true
    const token = Cookies.get('token')
    const url = configs.unlist_repo_url
    const { data } = await this.$axios.post(`${url}?token=${token}`, { _id })
    this.privateRepo = data.data
    this.disabled=false
  }
  async connectPaypal() {
    if (this.username.trim() !== '') {
      window.location.href = `https://www.sandbox.paypal.com/connect/?flowEntry=static&client_id=${configs.paypal_client_id}&response_type=code&scope=email&redirect_uri=${configs.redirect_uri_paypal}`
    } else {
      alert('please wait until username shown')
    }
  }
  async disconnectPaypal() {
    this.disabled=true
    if (this.username.trim() !== '') {
      const url = configs.disconnect_paypal_url
      const token = Cookies.get('token')
      this.$axios.post(`${url}?token=${token}`)
      this.paypalToken = false
    } else {
      alert('please wait until username shown')
    }
    this.disabled=false
//     if(this.privateRepo.length>0){
// this.getPrivateRepo()
//     }

  }
}
</script>

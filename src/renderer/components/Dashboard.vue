<template>
  <div>
    <AppHeader>
      <b-col class="py-1 justify-content-center align-self-center" cols="auto">
        <b-button variant="icon-lg" to="/add" v-b-popover.hover.bottom="$t('create.heading')"><PlusIcon /></b-button>
      </b-col>
    </AppHeader>
    <b-container>
      <b-card class="bs-24-40 my-5">
        <b-row>
          <b-col md="6">
            <h4 class="text-center pb-2">
              {{$t('public.balances')}}
            </h4>
            <vc-donut
              background="#00235B"
              :sections="amounts_chart"
              :total="(total_unspent/100000000)+0.01"
              :size="200"
              :thickness="10"
              :hasLegend="true"
            >
            {{$t('public.usable')}}<br />
            <i class="nuls-green"></i> {{((total_unspent || 0)/100000000).toFixed(2)}}<br />
            {{$t('public.staked')}}<br />
            <i class="nuls-green"></i> {{((total_consensus_locked || 0)/100000000).toFixed(2)}}<br />
            {{$t('public.time_locked')}}<br />
            <i class="nuls-green"></i> {{((total_time_locked || 0)/100000000).toFixed(2)}}
          </vc-donut>
          </b-col>
          <b-col md="6">
            <h4 class="text-center pb-2">
              {{$t('public.accounts')}}
            </h4>
            <vc-donut
              background="#00235B"
              :sections="accounts_chart"
              :total="total_unspent/100000000"
              :size="200"
              :thickness="10"
              :hasLegend="true"
            >
            <h6 class="head-900 pb-0 mb-0">{{accounts.length}}</h6>{{$t('public.wallets')}}
            </vc-donut>
          </b-col>
          <b-col>
          </b-col>
        </b-row>
      </b-card>
    </b-container>

    <b-container>
      <b-row>
        <b-col>
          <b-card class="mb-4">
            <h4 slot="header"><i class="nuls-green"></i> {{$t('public.balance')}} <span class="text-muted">(incl. locked)</span></h4>
            <p class="card-price"><i class="nuls-green"></i> {{((total_unspent || 0)/100000000).toFixed(2)}}</p>
          </b-card>
        </b-col>
        <b-col>
          <b-card class="mb-4">
            <h4 slot="header">{{$t('public.total_balance')}}</h4>
            <p class="card-price">
              {{price_info.DISPLAY.NULS[to_symbol].TOSYMBOL}}
              {{(((total_unspent || 0)/100000000) * price_info.RAW.NULS[to_symbol].PRICE).toFixed(2)}}
            </p>
          </b-card>
        </b-col>
        <b-col>
          <b-card class="mb-4">
            <h4 slot="header">{{$t('public.price')}}</h4>
            <p class="card-price">{{price_info.DISPLAY.NULS[to_symbol].PRICE}}</p>
          </b-card>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-card class="mb-4">
            <h4 slot="header"><i class="nuls-green"></i> {{$t('public.available_balance')}}</h4>
            <p class="card-price"><i class="nuls-green"></i> {{((total_available || 0)/100000000).toFixed(2)}}</p>
          </b-card>
        </b-col>
        <b-col>
          <b-card class="mb-4">
            <h4 slot="header"><i class="nuls-green"></i> {{$t('public.consensus_locked_balance')}}</h4>
            <p class="card-price"><i class="nuls-green"></i> {{((total_consensus_locked || 0)/100000000).toFixed(2)}}</p>
          </b-card>
        </b-col>
        <b-col>
          <b-card class="mb-4">
            <h4 slot="header"><i class="nuls-green"></i> {{$t('public.time_locked_balance')}}</h4>
            <p class="card-price"><i class="nuls-green"></i> {{((total_time_locked || 0)/100000000).toFixed(2)}}</p>
          </b-card>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import axios from 'axios'
import AppHeader from './AppHeader.vue'
import {
  PlusIcon, LogInIcon, MoreVerticalIcon, Edit3Icon, DeleteIcon
} from 'vue-feather-icons'
import { mapState } from 'vuex'
import store from '../store'



export default {
  name: 'dashboard',
  data () {
    return {
      total_unspent: 0,
      total_consensus_locked: 0,
      total_time_locked: 0,
      total_available: 0,
      amounts_chart: [],
      accounts_chart:  []
    }
  },
  components: {
    AppHeader,
    PlusIcon, LogInIcon, MoreVerticalIcon, Edit3Icon, DeleteIcon
  },
  computed: {
    // map this.count to store.state.count
    accounts() {
      return this.$store.getters.chain_accounts
    },
    unspent_info() {
      return this.$store.state.unspent_info
    },
    settings() {
      return this.$store.state.settings
    },
    last_height() {
      return this.$store.state.last_height
    },
    to_symbol() {
      return this.$store.state.to_symbol
    },
    price_info() {
      return this.$store.state.price_info
    }
  },
  watch: {
    accounts() { this.update(); },
    unspent_info() { this.update(); }
  },
  methods: {
    async update () {
      this.total_unspent = Object.values(this.unspent_info).map((u) => u.unspent_value).reduce((e, i) => e + i)
      this.total_available = Object.values(this.unspent_info).map((u) => u.available_value).reduce((e, i) => e + i)
      this.total_consensus_locked = Object.values(this.unspent_info).map((u) => u.consensus_locked_value).reduce((e, i) => e + i)
      this.total_time_locked = Object.values(this.unspent_info).map((u) => u.time_locked_value).reduce((e, i) => e + i)
      this.update_charts()
    },
    update_charts () {
      this.amounts_chart = [{
          label: 'Available Balance',
          value: this.total_available / 100000000,
          color: "#FFFFFF"
        }, {
          label: 'Time Locked',
          value: this.total_time_locked / 100000000,
          color: "#0A3B89"
        }, {
          label: 'Consensus locked',
          value: this.total_consensus_locked / 100000000,
          color: "#002E78"
        }]
      this.accounts_chart = this.accounts.map((a) => ({
        label: a.name,
        value: (Object.keys(this.unspent_info).includes(a.address) ? this.unspent_info[a.address].unspent_value / 100000000 : 0)
      }))

    },
    rename_account (account) {
      store.commit('start_rename', account)
    },
    delete_account (account) {
      if (confirm(`Delete account ${account.name} ?\n\nPlease backup your private key before doing this!`)) {
        if (confirm(`The address is ${account.address}\n\nAre you really sure? There is no way to going back!`)) {
          alert(`Ok, deleting address ${account.address}`)
          store.commit('remove_account', account);
        }
      }
    }
  },
  async created () {
    // We may not have a correct account list yet... So wait a bit.
    this.$nextTick(this.update.bind(this))
    // setTimeout(this.update.bind(this), 500)
    setInterval(this.update.bind(this), 60000)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.chart-middle-caption {
  padding-top: 2rem;
}
.card-body.vchart {
  z-index: 10;
}
.menu-btn .btn {
  padding: 0 !important;
}
.card-table td, .card-table tr {
  vertical-align: middle;
}
</style>

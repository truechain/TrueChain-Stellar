<template>
  <div class="set-tx-config">
    <div>
      <span class="label">From</span>
      <div class="input">
        <div class="greenbelt-from" v-if="useGreenBelt">{{ greenbeltAddr }}</div>
        <selector
          v-else ref="account"
          :defaultSelect="'0x...'"
          :options="allAccounts"
          :defaultOptions="deployFromDefaultOptions"
          canCopy
          @change="changeDeployFrom">
        </selector>
      </div>
      <div class="clear"></div>
    </div>
    <div>
      <span class="label">Gas Price</span>
      <input v-model="options.gasPrice" type="text">
      <span class="notice">{{gasPriceGwei}}</span>
      <div class="clear"></div>
    </div>
    <div>
      <span class="label">Gas Limit</span>
      <input v-model="options.gas" type="text">
      <span class="notice">{{computedGas ? `${$t('SetTxConfig.expected')} ${computedGas}` : $t('SetTxConfig.noSugs')}}</span>
      <div class="clear"></div>
    </div>
    <div @click="next" :class="{'next-active': gotReady}" class="next-contract">
      {{ $t('Common.next') }}
    </div>
  </div>
</template>

<script>
import { mapState, mapGetters, mapMutations } from 'vuex'

import Selector from './Selector'

export default {
  name: 'SetTxConfig',
  props: ['computedGas', 'active'],
  data () {
    return {
      options: {
        from: '',
        gas: Number(this.computedGas) || 100000,
        gasPrice: '20000000000'
      }
    }
  },
  computed: {
    ...mapState({
      useGreenBelt: state => state.useGreenBelt,
      greenbeltAddr: state => state.greenbeltAddr,
      'gasPrice': state => state.eth.gasPrice,
      'utils': state => state.web3.utils,
      'latestAccounts': state => state.accounts.latestAccounts
    }),
    ...mapGetters([
      'allAccounts'
    ]),
    deployFromDefaultOptions () {
      return [this.$t('SetTxConfig.newAccount')]
    },
    gotReady () {
      const validFrom = this.useGreenBelt && this.greenbeltAddr || this.utils.isAddress(this.options.from)
      return validFrom && this.active
    },
    gasPriceGwei () {
      let error = false
      let gwei = ''
      try {
        gwei = this.utils.fromWei(this.options.gasPrice, 'Gwei')
      } catch (err) {
        error = true
      }
      return error ? 'invalid' : `= ${gwei} Gwei`
    }
  },
  watch: {
    latestAccounts (newValue) {
      if (this.utils.isAddress(newValue)) {
        this.$nextTick(() => {
          this.$refs.account.changeSelect(newValue)
        })
      }
    },
    computedGas (newValue) {
      this.options.gas = newValue
    },
    gasPrice (newValue) {
      this.options.gasPrice = newValue
    }
  },
  created () {
    this.options.gasPrice = this.gasPrice || '1000000'
    if (this.useGreenBelt) {
      this.$emit('selectFrom', this.greenbeltAddr)
    }
  },
  methods: {
    ...mapMutations([
      'toAddAccounts'
    ]),
    changeDeployFrom (key, isDefaultOption) {
      if (isDefaultOption) {
        if (key === this.deployFromDefaultOptions[0]) {
          this.toAddAccounts()
        }
      } else {
        this.$emit('selectFrom', key)
        this.options.from = key
      }
    },
    next () {
      if (!this.gotReady) {
        return
      }
      if (this.useGreenBelt && this.greenbeltAddr) {
        this.options.from = this.greenbeltAddr
      }
      this.$emit('next', this.options)
    }
  },
  components: {
    Selector
  }
}
</script>

<style lang="stylus" scoped>
.set-tx-config
  width 100%
  font-size 16px
  &>div
    margin 20px 0
  .label
    display block
    float left
    width 100px
    line-height 40px
  .input
    float left
    width calc(100% - 100px)
    line-height 38px
    // font-family "ubuntu"
  input
    width 120px
  .notice
    position absolute
    line-height 40px
    margin-left 6px
    opacity .6
.deploy-confirm
  margin-top -20px
  width 50%
  padding-left 60px
  box-sizing border-box
  float left
  &>div
    margin 20px 0
  .deploytx-info
    font-size 12px
    border solid 1px #ddd
    background-color #eee
    color #666
    border-radius 3px
    height 160px
    padding 9px
    box-sizing border-box

.next-contract
  background-color #bbb
  cursor default
  line-height 40px
  border-radius 3px
  color #fff
  text-align center
  transition background-color .4s
.next-active
  background-color #0d85da
  cursor pointer

.greenbelt-from
  overflow hidden
  text-overflow ellipsis
  position relative
  &:before
    content 'GreenBelt account'
    color #0fa9a2
    font-size 12px
    position absolute
    top 0
    left 0
    line-height 12px
</style>

<template>
  <div class="content">
    <div class="flex-container center fill">
      <el-table
        :data="historyList"
        style="width: 100%"
        :border="true"
        :empty-text="$t('HISTORY_PAGE.EMPTY_LIST')">
        <el-table-column
          width="40"
          class-name="icon-column">
          <template scope="scope">
            <img :src="scope.row.icon">
          </template>
        </el-table-column>
        <el-table-column
          prop="value"
          :label="$t('COMMON.VALUE')"
          width="100">
        </el-table-column>
        <el-table-column
          prop="relativeTime"
          :label="$t('COMMON.DATE')"
          width="150">
        </el-table-column>
        <el-table-column
          prop="tx.Destination"
          :label="$t('COMMON.ADDRESS')">
          <template scope="scope">
            <span class="small-text" v-if="scope.row.state == 'receive'">{{ scope.row.tx.Account }}</span>
            <span class="small-text" v-else>{{ scope.row.tx.Destination }}</span>
          </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import { mapGetters } from 'vuex'
export default {
  name: 'history-page',
  data () {
    return {
      historyList: []
    }
  },
  computed: {
    ...mapGetters([
      'getWallet'
    ])
  },
  mounted () {
    this.Loading(true)
    let url = `https://data.ripple.com/v2/accounts/${this.getWallet.address}/transactions`
    let params = {
      limit: 15,
      descending: true
    }
    this.$http.get(url, {params: params}).then(res => {
      this.Loading(false)
      if (res.data.result === 'success') {
        console.log(res.data.transactions)
        res.data.transactions.forEach(item => {
          if (item.tx.Destination === this.getWallet.address) {
            item.icon = '/static/receive.svg'
            item.state = 'receive'
          } else if (item.meta.TransactionResult === 'tecUNFUNDED_PAYMENT') {
            item.icon = '/static/sendfailed.svg'
            item.state = 'sendfailed'
          } else {
            item.icon = '/static/send.svg'
            item.state = 'send'
          }
          item.value = item.tx.Amount / 1000000
          item.relativeTime = moment(item.date).startOf('hour').fromNow()
        })
        this.historyList = res.data.transactions
      }
    }).catch(err => {
      this.Loading(false)
      console.log(err)
    })
  }
}
</script>

<style lang="scss" scoped>
.content {
  padding-top:60px;
  height:100%;

  .fill {
    width:100%;
    height:100%;
  }

  span.small-text {
    font-size:12px;
  }
}
</style>

<template>
  <va-card :title="$t('tables.searchTrendsBadges')">
    <div class="row align--center">
      <div class="flex xs12 md6">
        <va-input
          :value="term"
          :placeholder="$t('tables.searchByName')"
          @input="search"
        >
          <va-icon name="fa fa-search" slot="prepend" />
        </va-input>
      </div>

      <div class="flex xs12 md3 offset--md3">
        <va-select
          v-model="perPage"
          :label="$t('tables.perPage')"
          :options="perPageOptions"
        />
      </div>
    </div>
    <va-data-table
      :fields="fields"
      :data="filteredData"
      :per-page="parseInt(perPage)"
    >
      <template slot="pKey" slot-scope="props">
        <div>
          {{props.rowData.meta.serial_number}}
        </div>
      </template>
      <template slot="productName" slot-scope="props">
        <div>
          {{props.rowData.meta.name}}
        </div>
      </template>
      <template slot="status" slot-scope="props">
        <va-div>
          {{ props.rowData.transition_type }}
        </va-div>
      </template>
      <template slot="createdAt" slot-scope="props">
        <div>
          {{ props.rowData.created_at.split('T')[0] }}
          {{ props.rowData.created_at.split('T')[1].split('.')[0]}}
        </div>
      </template>
      <template slot="memo" slot-scope="props">
        <div>
          {{props.rowData.note}}
          <button @click="showStaticModal = true,
          modalTitle='메모 변경', inputValue=props.rowData.note,
          targetTransitionId=props.rowData.id">수정</button>
        </div>
      </template>
    </va-data-table>

    <va-modal
      v-model="showStaticModal"
      size="large"
      :title="modalTitle"
      cancelClass="none"
      :okText=" $t('modal.confirm') "
      noOutsideDismiss
      noEscDismiss
      v-on:ok="editTransitionMemo(targetTransitionId, inputValue)"
    >
      <textarea name="" id="" cols="50" rows="5" v-model="inputValue"></textarea>
    </va-modal>
  </va-card>
</template>

<script>
import { debounce } from 'lodash'

export default {
  data () {
    return {
      term: null,
      perPage: '6',
      perPageOptions: ['4', '6', '10', '20'],
      users: [],
      showStaticModal: false,
      modalTitle: '',
      inputValue: '',
      targetTransitionId: '',
    }
  },
  computed: {
    fields () {
      return [ {
        name: '__slot:pKey',
        title: this.$t('tables.headings.pKey'),
        width: '5%',
      }, {
        name: '__slot:productName',
        title: this.$t('tables.headings.productName'),
        width: '10%',
      }, {
        name: '__slot:status',
        title: this.$t('tables.headings.nowStatus'),
        width: '3%',
      }, {
        name: '__slot:createdAt',
        title: this.$t('tables.headings.createdAt'),
        width: '3%',
      }, {
        name: '__slot:memo',
        title: this.$t('tables.headings.memo'),
        width: '40%',
      }]
    },
    filteredData () {
      if (!this.term || this.term.length < 1) {
        return this.users
      }
      return this.users.filter(item => {
        return item.productName.toLowerCase().startsWith(this.term.toLowerCase())
      })
    },
  },
  mounted () {
    this.fetchDatas(57039)
  },
  methods: {
    async fetchDatas (customerId) {
      let url = process.env.VUE_APP_URL_CHECKER_SERVICE + '/v1/transitions/host/' + customerId + '/'
      let next = ''
      while (url !== null) {
        await this.$http.get(url)
          .then((result) => {
            // console.log(result.data)
            for (var i = 0; i < result.data.results.length; i++) {
              result.data.results[i].transition_type = this.convertType(result.data.results[i].transition_type)
              this.users.push(result.data.results[i])
            }
            next = result.data.next
          })
        url = next
      }
    },
    async editTransitionMemo (transitionId, note) {
      let urlHead = '/v1/transitions/'
      let urlTail = '/note/'
      let payload = new FormData()
      payload.append('content', note)
      await this.$http.put(process.env.VUE_APP_URL_CHECKER_SERVICE + urlHead + transitionId + urlTail, payload)
        .then((result) => {
          if (result.status === 201) {
            alert('수정하였습니다.')
            // this.fetchDatas(57039)
            this.$nextTick(function () {
              this.users = []
              this.fetchDatas(57039)
              // do something cool
            })
          }
        })
    },
    convertType (type) {
      if (type === 'TYPE_OPEN') {
        return '열림'
      }
      if (type === 'TYPE_CLOSE') {
        return '닫힘'
      }
    },
    getTrendIcon (user) {
      if (user.trend === 'up') {
        return 'fa fa-caret-up'
      }

      if (user.trend === 'down') {
        return 'fa fa-caret-down'
      }

      return 'fa fa-minus'
    },
    getTrendColor (user) {
      if (user.trend === 'up') {
        return 'primary'
      }

      if (user.trend === 'down') {
        return 'danger'
      }

      return 'grey'
    },
    showUser (user) {
      alert(JSON.stringify(user))
    },
    search: debounce(function (term) {
      this.term = term
    }, 400),
  },
}
</script>

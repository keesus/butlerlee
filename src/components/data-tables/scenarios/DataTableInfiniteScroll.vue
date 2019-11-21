<template>
  <va-card :title="$t('tables.infiniteScroll')">
    <div class="data-table-infinite-scroll--container" ref="scrollable" @scroll="onScroll">
      <va-data-table
        :fields="fields"
        :data="users"
        api-mode
        no-pagination
      >
        <template slot="name" slot-scope="props">
          <div>
            {{props.rowData.meta.name}}
            <button @click="showStaticModal = true, modalTitle='이름 변경',
            inputValue=props.rowData.meta.name,
            targetCheckerId=props.rowData.id">수정</button>
          </div>
        </template>

      </va-data-table>
      <div class="flex-center ma-3">
        <spring-spinner
          v-if="loading"
          :animation-duration="2000"
          :size="60"
          :color="$themes.primary"
        />
      </div>
    </div>
    <va-modal
      v-model="showStaticModal"
      size="large"
      :title="modalTitle"
      cancelClass="none"
      :okText=" $t('modal.confirm') "
      noOutsideDismiss
      noEscDismiss
      v-on:ok="editCheckerName(targetCheckerId, inputValue)"
    >
      <textarea name="" id="" cols="50" rows="5" v-model="inputValue"></textarea>
    </va-modal>

  </va-card>
</template>

<script>
import { SpringSpinner } from 'epic-spinners'

export default {
  components: {
    SpringSpinner,
  },
  data () {
    return {
      users: [],
      loading: false,
      offset: 0,
      showStaticModal: false,
      targetCheckerId: '',
    }
  },
  computed: {
    fields () {
      return [{
        name: '__slot:name',
        title: this.$t('tables.headings.name'),
      }, {
        name: 'meta.serial_number',
        title: this.$t('tables.headings.serialNumber'),
      }, {
        name: 'meta.battery_level',
        title: this.$t('tables.headings.batteryStatus'),
      }, {
        name: 'connection_status',
        title: this.$t('tables.headings.connectionStatus'),
      }]
    },
  },
  mounted () {
    this.fetchData(57039)
  },
  methods: {
    fetchData (customerId) {
      this.loading = true
      let url = '/v1/ownerships/host/'
      this.$http.get(process.env.VUE_APP_URL_CHECKER_SERVICE + url + customerId + '/')
        .then((result) => {
          this.users = result.data
          this.loading = false
        })
    },
    editCheckerName (checkerId, input) {
      this.loading = true
      let url = '/v1/meta/'
      let payload = new FormData()
      payload.append('checker', checkerId)
      payload.append('key', 'name')
      payload.append('value', input)

      this.$http.post(process.env.VUE_APP_URL_CHECKER_SERVICE + url, payload)
        .then((result) => {
          if (result.status === 201) {
            alert('수정하였습니다.')
            this.fetchData(57039)
          }
        })
    },
    onScroll (e) {
      if (this.loading) {
        return
      }
      const { target } = e

      if (target.offsetHeight + target.scrollTop === target.scrollHeight) {
        this.loadMore()
      }
    },
  },
}
</script>

<style lang="scss">
  .data-table-infinite-scroll--container {
    height: 300px;
    overflow-y: auto;
  }
</style>

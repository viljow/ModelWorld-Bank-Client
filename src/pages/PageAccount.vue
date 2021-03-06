<template>
  <div id="PageLoginSuccess">
    <div class="page-header">
      <h1>{{account.name}}
        <small>{{account.description}}</small>
      </h1>
    </div>
    <div class="row">
      <div class="col-md-12">
        <div class="panel panel-primary">
          <div class="panel-heading">
            Account Summary
          </div>
          <div class="panel-body">
            Unique Account ID: <strong>{{account._id}}</strong>
            <br/>
            Creation Date: {{account.created}}
            <br/>
            Public: {{account.public}}
            <br/>
          </div>
          <table class="table table-striped table-bordered">
            <thead>
              <tr>
                <th>Currency</th>
                <th>Balance</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(amount, currency) in balances">
                <td>{{currency}}</td>
                <td>{{amount | currency}}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      <div class="col-md-12">
        <nav class="navbar navbar-default">
          <div class="container-fluid">
            <ul class="nav navbar-nav">
              <li><router-link :to="'/account/' + this.$route.params.id + '/transactions'">Transactions</router-link></li>
              <li><router-link :to="'/account/' + this.$route.params.id + '/wages'">Wages</router-link></li>
              <li><router-link :to="'/account/' + this.$route.params.id + '/bets'">Bets</router-link></li>
              <!--<li><router-link :to="'/account/' + this.$route.params.id + '/property'">Property</router-link></li>-->
              <li><router-link :to="'/account/' + this.$route.params.id + '/settings'">Settings</router-link></li>
            </ul>
          </div>
        </nav>
      </div>
      <div class="col-md-12" v-if="(this.$route.params.sub == undefined) || (this.$route.params.sub == 'transactions')">
        <div class="panel panel-primary">
          <div class="panel-heading">
            Transactions
          </div>
          <vue-good-table
          :columns="tables.transactions"
          :rows="transactions"
          :search-options="{
            enabled: true
          }"
          :pagination-options="{
            enabled: true
          }"
          :sort-options="{
            enabled: true,
            initialSortBy: {field: 'created', type: 'desc'}
          }"
          >
            <template slot="table-row" scope="props">
              <span v-if="props.column.field === 'created'">{{ props.row.created | dateString}}</span>
              <span v-if="props.column.field === 'amount'">{{ props.row.amount | currency }} {{ props.row.currency }}</span>
              <span :title="props.row.other._id" v-if="props.column.field === 'other.name'"><strong>{{ props.row.other.name }}</strong></span>
              <span :title="props.row._id" v-if="props.column.field === 'description'">{{ props.row.description }}</span>
            </template>
          </vue-good-table>
          <div class="panel-footer">
            <transaction-dialogue v-on:updateTransactions="fetchTransactions"></transaction-dialogue>
          </div>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-md-12" v-if="this.$route.params.sub == 'wages'">
        <div class="panel panel-primary">
          <div class="panel-heading">
            Wages
          </div>
          <vue-good-table
          :columns="tables.wages"
          :rows="account.wages"
          :search-options="{
            enabled: true
          }"
          :pagination-options="{
            enabled: true
          }"
          :sort-options="{
            enabled: true,
            initialSortBy: {field: 'value', type: 'desc'}
          }"
          >
            <template slot="table-row" scope="props">
              <span v-if="props.column.field === '_id'"><strong>{{ props.row._id }}</strong></span>
              <span v-if="props.column.field === 'name'">{{ props.row.name }}</span>
              <span v-if="props.column.field === 'description'">{{ props.row.description }}</span>
              <span v-if="props.column.field === 'value'">{{ props.row.value | currency}} {{props.row.currency}}</span>
              <span v-if="props.column.field === 'delete'"><button class="btn btn-danger" v-on:click="deleteWage(props.row)">Remove</button></span>
            </template>
          </vue-good-table>
          <table class="table table-striped table-bordered">
            <thead>
              <tr>
                <th>Currency</th>
                <th>Total Wage</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(amount, currency) in totalWages">
                <td>{{currency}}</td>
                <td>{{amount | currency}}</td>
              </tr>
            </tbody>
          </table>
          <div class="panel-footer">
            <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#wageRequestModal">Request Wage</button>
          </div>
        </div>
      </div>
      <div class="col-md-12" v-if="this.$route.params.sub == 'settings'">
        <div class="panel panel-primary">
          <div class="panel-heading">
            Users
          </div>
          <vue-good-table
          :columns="tables.users"
          :rows="account.users"
          >
            <template slot="table-row" scope="props">
              <span v-if="props.column.field === 'name'">{{ props.row.name }}</span>
              <span v-if="props.column.field === 'level'">{{ props.row.level | accessLevel }}</span>
            </template>
          </vue-good-table>
          <div class="panel-footer">
            <div class="row">
              <div class="col-md-6">
                <label for="wage-name">Username:</label>
                <div class="input-group">
                  <span class="input-group-addon">Username:</span>
                  <input v-model="userToAdd.name" type="text" id="wage-name" class="form-control">
                </div>
                <br>
                <div class="input-group">
                  <label for="sel1">Access Level:</label>
                  <select v-model="userToAdd.level" class="form-control" id="sel1">
                    <option v-for='(name, level) in accessLevels' :value="level">{{name}}</option>
                  </select>
                </div>
                <br>
                <button type="button" v-on:click="addUser" class="btn btn-primary">Save User</button>
              </div>
            </div>
          </div>
        </div>
      </div>
      <account-description-dialogue v-if="this.$route.params.sub == 'settings'" v-on:updatedAccountDescription="fetchAccount" :description="account.description" :public="account.public"></account-description-dialogue>
      <wager-list v-if="this.$route.params.sub == 'bets'"></wager-list>
    </div>
    <div v-if="user.admin && this.$route.params.sub == 'settings'" class="row">
      <div class="col-md-12">
        <div class="panel panel-primary">
          <div class="panel-heading">
            Admin Options
          </div>
          <div class="panel-body">
            <button v-on:click="deleteAccount()" type="button" class="btn btn-danger">Delete Account</button>
            <button type="button" v-on:click="payWages()" class="btn btn-primary">Pay Wages</button>
            <button type="button" class="btn btn-primary disabled">Place Hold</button>
          </div>
        </div>
      </div>
    </div>
    <div class="modal fade" id="wageRequestModal" tabindex="-1" role="dialog" >
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span>&times;</span></button>
            <h4 class="modal-title">Wage Requests</h4>
          </div>
          <div class="modal-body">
            <h4>Currently Requested</h4>
            <p>Your requests will be validated as soon as possible. Please be patient.</p>
            <vue-good-table
            :columns="tables.wageRequests"
            :rows="wageRequests"
            :filterable="true"
            :globalSearch="true"
            :paginate="true"
            styleClass="table table-bordered condensed"/>
            </vue-good-table>
            <hr />
            <h4>Request New</h4>
            <p>Once requested, a site operator will validate and accept your request to have this wage added.</p>
            <vue-good-table
            :columns="tables.wagesList"
            :rows="possibleWages"
            :filterable="true"
            :globalSearch="true"
            :paginate="true"
            styleClass="table table-bordered condensed">
              <template slot="table-row" scope="props">
                <span v-if="props.column.field === 'name'">{{ props.row.name }}</span>
                <span v-if="props.column.field === 'description'">{{ props.row.description }}</span>
                <span v-if="props.column.field === 'value'">{{ props.row.value | currency }}</span>
                <span v-if="props.column.field === 'request'"><button type="button" v-on:click="requestWage(props.row._id)" class="btn btn-primary">Request</button></span>
              </template>
            </vue-good-table>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import api from '@/api'
import errorHandler from '@/errorHandler'
import swal from 'sweetalert2'
import {accessLevels} from '@/globalValues'
import TransactionDialogue from '@/pages/account/TransactionDialogue'
import AccountDescriptionDialogue from '@/pages/account/AccountDescriptionDialogue'
import WagerList from '@/pages/account/WagerList'

export default {
  name: 'PageAccount',
  store: ['user', 'jwt', 'currencies'],
  components: {TransactionDialogue, AccountDescriptionDialogue, WagerList},
  data: function () {
    return {
      account: {},
      accessLevels: accessLevels,
      userToAdd: {name: '', level: ''},
      wageToRequest: '',
      wageRequests: [],
      possibleWages: [],
      transactions: [],
      balances: {},
      totalWages: {},
      tables: {
        users: [
          {
            label: 'Username',
            field: 'name'
          },
          {
            label: 'Access Level',
            field: 'level'
          }
        ],
        wages: [
          {
            label: 'ID',
            field: '_id'
          },
          {
            label: 'Name',
            field: 'name'
          },
          {
            label: 'Description',
            field: 'description'
          },
          {
            label: 'Value',
            field: 'value',
            type: 'decimal'
          },
          {
            label: 'Delete',
            field: 'delete',
            sortable: false
          }
        ],
        transactions: [
          {
            label: 'Date',
            field: 'created',
            type: 'decimal'
          },
          {
            label: 'Amount',
            field: 'amount',
            type: 'decimal'
          },
          {
            label: 'Other Account',
            field: 'other.name',
            sortable: false
          },
          {
            label: 'Description',
            field: 'description',
            sortable: false
          }
        ],
        wagesList: [
          {
            label: 'Name',
            field: 'name'
          },
          {
            label: 'Description',
            field: 'description'
          },
          {
            label: 'Value',
            field: 'value',
            type: 'decimal'
          },
          {
            label: 'Request',
            field: 'request',
            sortable: false
          }
        ],
        wageRequests: [
          {
            label: 'Request ID',
            field: '_id'
          },
          {
            label: 'Wage ID',
            field: 'wage._id'
          },
          {
            label: 'Name',
            field: 'wage.name'
          },
          {
            label: 'Requested',
            field: 'created'
          }
        ]
      }
    }
  },
  methods: {
    fetchAccount: function () {
      let $this = this
      api.request({
        url: '/api/account/id/' + $this.$route.params.id,
        method: 'get',
        headers: {jwt: $this.$store.jwt}
      }).then(function (response) {
        $this.processAccountData(response.data)
      }).catch(errorHandler)
    },
    fetchPossibleWages: function () {
      let $this = this
      api.request({
        url: '/api/wage',
        method: 'get',
        headers: {jwt: $this.$store.jwt}
      }).then(function (response) {
        $this.processWageData(response.data)
      }).catch(errorHandler)
    },
    fetchWageRequests: function () {
      let $this = this
      api.request({
        url: '/api/account/id/' + $this.$route.params.id + '/wage',
        method: 'get',
        headers: {jwt: $this.$store.jwt}
      }).then(function (response) {
        response.data = response.data.filter(function (val) { // Remove broken data entries. Will eventually be cleaned server-side
          return !(val.wage == null)
        })

        $this.wageRequests = response.data
        $this.fetchPossibleWages()
      }).catch(errorHandler)
    },
    fetchTransactions: function () {
      let $this = this
      api.request({
        url: '/api/account/id/' + $this.$route.params.id + '/transaction',
        method: 'get',
        headers: {jwt: $this.$store.jwt}
      }).then(function (response) {
        let processedTransactions = []

        response.data.transactions.forEach(function (transaction) {
          let accountID = $this.$route.params.id

          if (transaction.from === accountID) {
            transaction.other = transaction.to
            transaction.sign = '-'
          } else if (transaction.to === accountID) {
            transaction.other = transaction.from
            transaction.sign = '+'
          }

          transaction.created = Date.parse(transaction.created)

          if (transaction.other == null) {
            transaction.other = {_id: 'deleted', name: 'Deleted Account'}
          }

          let amount = Math.abs(transaction.amount)

          if (transaction.sign === '-') {
            amount = amount * (-1)
          }

          transaction.amount = amount

          processedTransactions.push(transaction)
        })

        $this.transactions = processedTransactions
        $this.balances = response.data.balance
      }).catch(errorHandler)
    },
    processAccountData: function (responseData) {
      let userData = []
      let $this = this

      this.totalWages = {}

      for (let key in responseData.users) {
        if (responseData.users.hasOwnProperty(key)) {
          userData.push({name: key, level: responseData.users[key]})
        }
      }
      responseData.users = userData

      $this.account = responseData

      responseData.wages.forEach(function (wage) {
        if ($this.totalWages[wage.currency]) {
          $this.totalWages[wage.currency] += wage.value
        } else {
          $this.totalWages[wage.currency] = wage.value
        }
      })

      $this.fetchWageRequests()
    },
    processWageData: function (responseData) {
      let $this = this
      responseData = responseData.filter(function (val) {
        let existingWages = $this.account.wages.find(function (wage) { // Remove elements that exist already in the wage data.
          return wage['_id'] === val['_id']
        })

        let existingRequests = $this.wageRequests.find(function (wageRequest) { // Remove elements that exist already in the requests
          if (wageRequest.wage) {
            return wageRequest.wage['_id'] === val['_id']
          } else {
            return false
          }
        })
        return (existingWages === undefined) && (existingRequests === undefined)
      })

      $this.possibleWages = responseData
    },
    addUser: function () {
      let $this = this
      api.request({
        url: '/api/account/id/' + $this.$route.params.id + '/user',
        method: 'post',
        headers: {jwt: $this.$store.jwt},
        data: {newDocument: $this.userToAdd}
      }).then(function (response) {
        $this.processAccountData(response.data)
      }).catch(errorHandler)
    },
    deleteWage: function (wage) {
      let $this = this
      swal({
        title: 'ARE YOU SURE?',
        type: 'warning',
        text: 'Clicking \'ok\' will remove this wage!',
        showCancelButton: true
      }).then((result) => {
        if (result.value) {
          api.request({
            url: '/api/account/id/' + $this.$route.params.id + '/wage/' + wage._id,
            method: 'delete',
            headers: {jwt: $this.$store.jwt}
          }).then(function (response) {
            $this.fetchAccount()
          }).catch(errorHandler)
        }
      })
    },
    requestWage: function (wageID) {
      let $this = this
      api.request({
        url: '/api/account/id/' + $this.$route.params.id + '/wage',
        method: 'post',
        headers: {jwt: $this.$store.jwt},
        data: {wageID: wageID}
      }).then(function (response) {
        $this.wageRequests = response.data
        $this.fetchPossibleWages()
      }).catch(errorHandler)
    },
    deleteAccount: function () {
      let $this = this
      swal({
        title: 'ARE YOU SURE?',
        type: 'warning',
        text: 'Clicking \'ok\' will permenantly delete this account!',
        confirmButtonText: 'Delete It!',
        showCancelButton: true
      }).then((result) => {
        if (result.value) {
          api.request({
            url: '/api/account/id/' + $this.account._id,
            method: 'delete',
            headers: {jwt: $this.$store.jwt}
          }).then(function (response) {
            $this.$router.push('/admin/accounts')
          }).catch(errorHandler)
        }
      })
    },
    payWages: function () {
      let $this = this
      api.request({
        url: '/api/account/id/' + $this.$route.params.id + '/pay',
        method: 'get',
        headers: {jwt: $this.$store.jwt}
      }).then(function (response) {
        $this.fetchTransactions()
      }).catch(errorHandler)
    }
  },
  mounted: function () {
    this.fetchAccount()
    this.fetchTransactions()
  },
  filters: {
    currency: function (value) {
      return value.toFixed(2).replace(/(\d)(?=(\d{3})+\.)/g, '$1,')
    },
    accessLevel: function (level) {
      return accessLevels[level]
    },
    dateString: function (dateIn) {
      let date = new Date(dateIn)
      let string = date.toLocaleString('en-GB')
      return string
    }
  }
}
</script>

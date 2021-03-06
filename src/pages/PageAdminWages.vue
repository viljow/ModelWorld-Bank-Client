<template>
  <div id="PageLoginSuccess">
    <div class="page-header">
      <h1>Wages Management
        <small>Create and update existing wages</small>
      </h1>
    </div>
    <div class="row">
      <div class="col-md-12">
        <div class="panel panel-primary">
          <div class="panel-heading">
            Wages
          </div>
          <vue-good-table
          :columns="tables.wages"
          :rows="wages"
          :search-options="{
            enabled: true
          }"
          :pagination-options="{
            enabled: true
          }"
          >
            <template slot="table-row" scope="props">
              <span v-if="props.column.field === '_id'"><strong>{{ props.row._id }}</strong></span>
              <span v-if="props.column.field === 'name'">{{ props.row.name }}</span>
              <span v-if="props.column.field === 'description'">{{ props.row.description }}</span>
              <span v-if="props.column.field === 'value'">{{ props.row.value | currency}} {{ props.row.currency}}</span>
              <span v-if="props.column.field === 'buttons'"><button class="btn btn-primary" v-on:click="selectWage(props.row)" data-toggle="modal" data-target="#wageModal">Modify Wage</button></span>
            </template>
          </vue-good-table>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-md-12">
        <div class="panel panel-primary">
          <div class="panel-heading">
            Add Wage
          </div>
          <form>
            <div class="panel-body">
              <div class="input-group">
                <span class="input-group-addon">Wage Name:</span>
                <input v-model="newWage.name" type="text" id="wage-name" class="form-control">
              </div>
              <br>
              <div class="input-group">
                <span class="input-group-addon">Wage Description:</span>
                <input v-model="newWage.description" type="text" id="wage-description" class="form-control">
              </div>
              <br>
              <div class="input-group">
                <span class="input-group-addon">Wage Amount:</span>
                <input v-model="newWage.value" type="text" id="wage-amount" class="form-control">
              </div>
              <br>
              <div class="input-group">
                <label for="sel1">Currency:</label>
                <select v-model="newWage.currency" class="form-control" id="sel1">
                  <option v-for='currency in currencies'>{{currency}}</option>
                </select>
              </div>
            </div>
            <div class="panel-footer">
              <button v-on:click="createWage()" type="button" class="btn btn-primary">Submit Form</button>
            </div>
          </form>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-md-12">
        <div class="panel panel-primary">
          <div class="panel-heading">
            Wage Requests
          </div>
          <vue-good-table
          :columns="tables.wageRequests"
          :rows="wageRequests"
          :search-options="{
            enabled: true
          }"
          :pagination-options="{
            enabled: true
          }"
          >
            <template slot="table-row" scope="props">
              <span v-if="props.column.field === '_id'"><strong>{{ props.row._id }}</strong></span>
              <span v-if="props.column.field === 'wage._id'">{{ props.row.wage._id }}</span>
              <span v-if="props.column.field === 'account.name'">{{ props.row.account.name}}</span>
              <span v-if="props.column.field === 'wage.name'">{{ props.row.wage.name}}</span>
              <span v-if="props.column.field === 'user'">{{ props.row.user}}</span>
              <span v-if="props.column.field === 'created'">{{ props.row.created}}</span>
              <span v-if="props.column.field === 'buttons'">
                <button class="btn btn-success" v-on:click="decideRequest(props.row, true)">Accept</button>
                <button class="btn btn-danger" v-on:click="decideRequest(props.row, false)">Deny</button>
              </span>
            </template>
          </vue-good-table>
          <div class="panel-footer">
            <button type="button" class="btn btn-success" v-on:click="decideAllRequests(true)">Accept All</button>
            <button type="button" class="btn btn-danger" v-on:click="decideAllRequests(false)">Deny All</button>
            <button type="button" class="btn btn-danger" v-on:click="purgeWages()">Purge All Wages</button>
          </div>
        </div>
      </div>
    </div>

    <div class="modal fade" id="wageModal" tabindex="-1" role="dialog" >
      <div class="modal-dialog" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span>&times;</span></button>
            <h4 class="modal-title" id="myModalLabel">{{selectedWage._id}}</h4>
          </div>
          <div class="modal-body">
            <form>
              <div class="input-group">
                <span class="input-group-addon">Wage Name:</span>
                <input v-model="selectedWage.name" type="text" id="wage-name" class="form-control">
              </div>
              <br>
              <div class="input-group">
                <span class="input-group-addon">Wage Description:</span>
                <input v-model="selectedWage.description" type="text" id="wage-description" class="form-control">
              </div>
              <br>
              <div class="input-group">
                <span class="input-group-addon">Wage Amount:</span>
                <input v-model="selectedWage.value" type="text" id="wage-amount" class="form-control">
              </div>
              <br>
              <div class="input-group">
                <label for="sel1">Currency:</label>
                <select v-model="selectedWage.currency" class="form-control" id="sel1">
                  <option v-for='currency in currencies'>{{currency}}</option>
                </select>
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-default" data-dismiss="modal">Close</button>
            <button type="button" v-on:click="deleteWage()" class="btn btn-danger" data-dismiss="modal">Delete Wage</button>
            <button type="button" v-on:click="updateWage()" class="btn btn-primary" data-dismiss="modal">Save changes</button>
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

export default {
  name: 'PageAdminWages',
  store: ['user', 'jwt', 'currencies'],
  data: function () {
    return {
      wages: [],
      wageRequests: [],
      tables: {
        wages: [
          {
            label: 'ID',
            field: '_id',
            sortable: false
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
            label: 'Buttons',
            field: 'buttons',
            sortable: false
          }
        ],
        wageRequests: [
          {
            label: 'Request ID',
            field: '_id',
            sortable: false
          },
          {
            label: 'Wage ID',
            sortable: false,
            field: 'wage._id'
          },
          {
            label: 'Account Name',
            field: 'account.name'
          },
          {
            label: 'Wage Name',
            field: 'wage.name'
          },
          {
            label: 'User',
            field: 'user'
          },
          {
            label: 'Requested Time',
            field: 'created',
            type: 'decimal'
          },
          {
            label: 'Buttons',
            field: 'buttons',
            sortable: false
          },
        ]
      },
      wagesQueue: [],
      selectedWage: {},
      newWage: {
        name: '',
        description: '',
        value: '',
        currency: ''
      }
    }
  },
  mounted: function () {
    this.fetchWages()
    this.fetchRequests()
  },
  methods: {
    createWage: function (event) {
      let $this = this
      api.request({
        url: '/api/wage',
        method: 'post',
        headers: {jwt: this.$store.jwt},
        data: {newDocument: $this.newWage}
      }).then(function (response) {
        $this.wages = response.data
      }).catch(errorHandler)
    },
    purgeWages: function () {
      swal({
        title: 'ARE YOU SURE?',
        type: 'warning',
        text: 'Clicking \'ok\' will clear all wages from all accounts!',
        showCancelButton: true
      }).then((result) => {
        if (result.value) {
          api.request({
            url: '/api/wage/purge',
            method: 'post',
            headers: {jwt: this.$store.jwt},
            data: {}
          }).then(function (response) {
            swal('All wages have been purged...')
          }).catch(errorHandler)
        }
      })
    },
    decideRequest: function (wageRequest, decision) {
      // Decision - True = Accept, Decision - False = Deny
      let $this = this
      swal({
        title: 'ARE YOU SURE?',
        type: 'warning',
        text: 'Clicking \'ok\' will complete this action!',
        showCancelButton: true
      }).then((result) => {
        if (result.value) {
          api.request({
            url: '/api/request/id/' + wageRequest._id,
            method: 'post',
            headers: {jwt: this.$store.jwt},
            data: {accept: decision}
          }).then(function (response) {
            $this.fetchRequests()
          }).catch(errorHandler)
        }
      })
    },
    decideAllRequests: function (decision) {
      let $this = this
      swal({
        title: 'ARE YOU SURE?',
        type: 'warning',
        text: 'Clicking \'ok\' will ' + (decision ? 'accept' : 'deny') + ' all requests!',
        showCancelButton: true
      }).then((result) => {
        if (result.value) {
          api.request({
            url: '/api/request/all',
            method: 'post',
            headers: {jwt: this.$store.jwt},
            data: {decision}
          }).then(function (response) {
            $this.fetchRequests()
          }).catch(errorHandler)
        }
      })
    },
    selectWage: function (row) {
      this.selectedWage = Object.assign({}, row) // Performs a copy of the object
    },
    updateWage: function () {
      let $this = this
      api.request({
        url: '/api/wage/id/' + $this.selectedWage._id,
        method: 'put',
        headers: {jwt: this.$store.jwt},
        data: {updatedDocument: this.selectedWage}
      }).then(function (response) {
        $this.wages = response.data
      }).catch(errorHandler)
    },
    deleteWage: function () {
      let $this = this
      swal({
        title: 'ARE YOU SURE?',
        type: 'warning',
        text: 'Clicking \'ok\' will permenantly delete this wage!',
        showCancelButton: true
      }).then((result) => {
        if (result.value) {
          api.request({
            url: '/api/wage/id/' + $this.selectedWage._id,
            method: 'delete',
            headers: {jwt: this.$store.jwt}
          }).then(function (response) {
            $this.wages = response.data
          }).catch(errorHandler)
        }
      })
    },
    fetchWages: function () {
      let $this = this
      api.request({
        url: '/api/wage',
        method: 'get',
        headers: {jwt: this.$store.jwt}
      }).then(function (response) {
        $this.wages = response.data
      }).catch(errorHandler)
    },
    fetchRequests: function () {
      let $this = this
      api.request({
        url: '/api/request',
        method: 'get',
        headers: {jwt: this.$store.jwt}
      }).then(function (response) {
        response.data = response.data.filter(function (val) { // Remove broken data entries. Will eventually be cleaned server-side
          return !(val.wage == null || val.account == null)
        })
        $this.wageRequests = response.data
      }).catch(errorHandler)
    }
  },
  filters: {
    currency: function (value) {
      return value.toFixed(2).replace(/(\d)(?=(\d{3})+\.)/g, '$1,')
    }
  }
}
</script>

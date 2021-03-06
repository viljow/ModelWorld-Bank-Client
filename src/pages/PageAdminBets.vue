<template>
  <div id="PageAdminAccounts">
    <div class="page-header">
      <h1>Bet Management
        <small>Create and access bets</small>
      </h1>
    </div>
    <div class="row">
      <div class="col-md-12">
        <div class="panel panel-primary">
          <div class="panel-heading">
            Add Bet
          </div>
          <form>
            <div class="panel-body">
              <div class="input-group">
                <span class="input-group-addon">Bet Name:</span>
                <input v-model="newBet.name" type="text" class="form-control">
              </div>
              <br>
              <div class="input-group">
                <span class="input-group-addon">Bet Description:</span>
                <input v-model="newBet.description" type="text" class="form-control">
              </div>
              <br>
              <div class="input-group">
                <span class="input-group-addon">Initial Status:</span>
                <select v-model="newBet.status" class="form-control">
                  <option v-for='status in betStatus'>{{status}}</option>
                </select>
              </div>
              <br/>
              <strong>Bet Options:</strong>
              <div class="row">
                <div class="col-md-4" v-for="option in newBet.options">
                  <div class="panel panel-primary">
                    <div class="panel-heading">
                      {{option.name}}
                    </div>
                    <div class="panel-body">
                      <div class="input-group">
                        <span class="input-group-addon">Name:</span>
                        <input v-model="option.name" type="text" class="form-control">
                      </div>
                      <br/>
                      <div class="input-group">
                        <span class="input-group-addon">Description:</span>
                        <textarea v-model="option.description" type="text" class="form-control" style="height:100px; resize: none;"/>
                      </div>
                      <br>
                      <div class="input-group">
                        <span class="input-group-addon">Odds:</span>
                        <input v-model="option.currentOdds" type="text" class="form-control">
                      </div>
                      <br/>
                    </div>
                    <div class="panel-footer">
                      <button v-on:click="removeOption(option)" type="button" class="btn btn-danger">Remove</button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <div class="panel-footer">
              <button v-on:click="submitNew" type="button" class="btn btn-primary">Submit Form</button>
              <button v-on:click="addOption" type="button" class="btn btn-success">Add Option</button>
            </div>
          </form>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-md-12">
        <div class="panel panel-primary">
          <div class="panel-heading">
            Bet Management
          </div>
          <vue-good-table
            :columns="allBets.columns"
            :rows="allBets.rows"
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
              <span v-if="props.column.field === 'status'">{{ props.row.status | statusConversion}}</span>
              <span v-if="props.column.field === 'created'">{{ props.row.created | dateString}}</span>
              <span v-if="props.column.field === 'buttons'">
                <button type="button" class="btn btn-primary">Edit Bet</button>

                <button v-if="props.row.status == 1" v-on:click="updateStatus(props.row, 'Closed')" type="button" class="btn btn-primary">Close</button>
                <button v-if="props.row.status == 0" v-on:click="updateStatus(props.row, 'Open')" type="button" class="btn btn-primary">Open</button>
                <button v-if="props.row.status == 0" v-on:click="updateStatus(props.row, 'Paid')" type="button" class="btn btn-primary">Pay Out</button>
              </span>
            </template>
          </vue-good-table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import api from '@/api'
  import errorHandler from '@/errorHandler'
  import swal from 'sweetalert2'
  import {betStatus} from '@/globalValues'

  export default {
    name: 'PageAdminBets',
    store: ['user', 'jwt'],
    data: function () {
      return {
        betStatus,
        newBet: {
          name: 'XYZ BET',
          description: 'Bet decided upon XYZ based on X criterion by Z',
          status: 'Open',
          options: []
        },
        allBets: {
          columns: [
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
              label: 'Status',
              field: 'status'
            },
            {
              label: 'Created Date',
              field: 'created'
            },
            {
              label: 'Buttons',
              field: 'buttons',
              sortable: false
            }
          ],
          rows: []
        }
      }
    },
    methods: {
      submitNew: function (event) {
        api.request({
          url: '/api/bet',
          method: 'post',
          headers: {jwt: this.$store.jwt},
          data: this.newBet
        }).then((response) => {
          this.allBets.rows.push(response.data) // Add new account to table onscreen

          swal({
            title: 'Creation Success!',
            icon: 'success'
          })
        }).catch(errorHandler)
      },
      addOption: function () {
        this.newBet.options.push({
          name: 'Example Option',
          description: 'Johnson Johnson wins the election.',
          currentOdds: 3.76
        })
      },
      removeOption: function (option) {
        let index = this.newBet.options.indexOf(option)
        this.newBet.options.splice(index, 1)
      },
      fetchBets: function () {
        api.request({
          url: '/api/bet',
          method: 'get',
          headers: {jwt: this.$store.jwt}
        }).then((response) => {
          this.allBets.rows = response.data
        }).catch(errorHandler)
      },
      updateStatus: function (bet, status) {
        if (status === 'Paid') {
          let inputOptions = {} // Generate K-V dict of option id to option name

          bet.options.forEach((item) => {
            inputOptions[item._id] = item.name
          })

          swal({
            title: 'Select Winner!',
            text: 'Select the winning choice',
            input: 'select',
            inputOptions
          }).then((result) => {
            if (result.value) {
              api.request({
                url: '/api/bet/id/' + bet._id + '/status',
                method: 'put',
                headers: {jwt: this.$store.jwt},
                data: {status, winner: result.value}
              }).then((response) => { // Handles resolution of either promise.
                this.fetchBets()
              }).catch(errorHandler)
            }
          })
        } else {
          api.request({
            url: '/api/bet/id/' + bet._id + '/status',
            method: 'put',
            headers: {jwt: this.$store.jwt},
            data: {status}
          }).then((response) => { // Handles resolution of either promise.
            this.fetchBets()
          }).catch(errorHandler)
        }
      }
    },
    mounted: function () {
      this.fetchBets()
      this.addOption()
    },
    filters: {
      statusConversion: function (number) {
        return betStatus[number]
      },
      dateString: function (dateIn) {
        let date = new Date(dateIn)
        let string = date.toLocaleString('en-GB')
        return string
      }
    }
  }
</script>

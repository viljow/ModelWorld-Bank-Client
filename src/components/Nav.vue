<template>
  <div id="navbar">
    <nav class="navbar navbar-default">
      <div class="container-fluid">
        <!-- Brand and toggle get grouped for better mobile display -->
        <div class="navbar-header">
          <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <a class="navbar-brand" href="#"><img height="100%"  src="../assets/nubbank.png"></a>
        </div>

        <!-- Collect the nav links, forms, and other content for toggling -->
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
          <ul class="nav navbar-nav">
            <li>
              <router-link to="/">Home</router-link>
            </li>
            <li>
              <router-link to="/account">Banking</router-link>
            </li>
            <li>
              <router-link to="/betting">Betting</router-link>
            </li>
            <li>
              <router-link to="/leaderboard">Leaderboard</router-link>
            </li>
            <!--
            <li>
              <router-link to="/property">Broperty Market</router-link>
            </li>
            -->
          </ul>
          <ul class="nav navbar-nav navbar-right">
            <li v-if="user.name" class="dropdown">
              <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">{{user.name}}
                <span class="caret"></span>
              </a>
              <ul class="dropdown-menu">
                <li>
                  <a href="#">Preferences</a>
                </li>
                <li>
                  <router-link to="/logout">Log Out</router-link>
                </li>
              </ul>
            </li>
            <li v-else>
              <router-link to="/login">Login</router-link>
            </li>
            <li>
              <router-link to="/help">Help</router-link>
            </li>
            <li>
              <router-link to="/staff">Staff</router-link>
            </li>
            <li>
              <a href="https://discord.gg/MXkYHyB">Discord</a>
            </li>
            <li>
              <a>C:{{version}} | S:{{serverVersion}}</a>
            </li>
            <li v-if="user.admin" class="dropdown">
              <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">Admin Menu
                <span class="caret"></span>
              </a>
              <ul class="dropdown-menu">
                <li>
                  <router-link to="/admin/wages">Wage Management</router-link>
                </li>
                <li>
                  <router-link to="/admin/accounts">Account Management</router-link>
                </li>
                <li>
                  <router-link to="/admin/transactions">Transaction Management</router-link>
                </li>
                <li>
                  <router-link to="/admin/bets">Bet Management</router-link>
                </li>
                <li role="separator" class="divider"></li>
                <li>
                  <router-link to="/admin/economy">Economy Settings</router-link>
                </li>
              </ul>
            </li>
          </ul>
        </div>
        <!-- /.navbar-collapse -->
      </div>
      <!-- /.container-fluid -->
    </nav>
  </div>
</template>

<script>
import { version } from '../../package'
import api from '../api'

export default {
  name: 'navbar',
  store: ['user'],
  data: function () {
    return {
      version: process.env.NODE_ENV === 'production' ? version : 'DEV',
      serverVersion: '...'
    }
  },
  mounted: function () {
    api.get('/api/health').then((res) => {
      this.serverVersion = res.data.version
    })
  }
}
</script>

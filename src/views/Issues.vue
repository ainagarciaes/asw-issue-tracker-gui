<template>
  <div class="container mt-5">
    <div class="row">
      <div class=col-xl-3>
     <h1>Issue Tracker</h1>
      </div>
      <div class="col-xl-4 offset-xl-9">
        <b-button variant="primary" v-if="haveToken" v-on:click="authenticate('google')">Log In</b-button>
        <div v-else>
        <b-button variant="primary" v-on:click="logout()">Log Out</b-button>
        <b-button href='#/issues/new' > Nova Issue</b-button>
        </div>
      </div> 
    </div> 
        <b-button-toolbar>
          Filtrar per:
          <b-button-group class="mx-1">
            <b-button v-on:click="getIssues">Totes</b-button>
            <b-button v-on:click="getFilter('Nou')">Obertes</b-button>
            <b-button v-if="haveToken == false" v-on:click="getMyIssues()">Les meves issues</b-button>
            <b-button v-if="haveToken == false" v-on:click="getMyWatching()">Observant</b-button>
          </b-button-group>
        </b-button-toolbar>
      <h5>Issues:</h5>
      <b-table ref="table" striped hover :items=issues :fields="fields">
        <template v-slot:cell(titol)="data">
          <!-- `data.value` is the value after formatted by the Formatter -->
          <a class="link" :href="`#/issues/${data.item.id}`" target="_self" >{{ data.value }}</a>
        </template>
        <template v-slot:cell(tipus)="data">
          <!-- `data.value` is the value after formatted by the Formatter -->
          <a :href="`#/issues/`" @click="getFilter(data.item.tipus)" >{{ data.value }}</a>
        </template>
        <template v-slot:cell(prioritat)="data">
          <!-- `data.value` is the value after formatted by the Formatter -->
          <a :href="`#/issues/`" @click="getFilter(data.item.prioritat)" >{{ data.value }}</a>
        </template>
        <template v-slot:cell(status)="data">
          <!-- `data.value` is the value after formatted by the Formatter -->
          <a :href="`#/issues/`" @click="getFilter(data.item.status)" >{{ data.value }}</a>
        </template>
        <template v-slot:cell(prioritat)="data">
          <!-- `data.value` is the value after formatted by the Formatter -->
          <a :href="`#/issues/`" @click="getFilter(data.item.prioritat)" >{{ data.value }}</a>
        </template>
        <template v-slot:cell(data_creacio)="data">
          <!-- `data.value` is the value after formatted by the Formatter -->
          <a :href="`#/issues/`" @click="getFilter(data.item.data_creacio)" >{{ data.value }}</a>
        </template>
        <template v-slot:cell(assignee)="data">
          <!-- `data.value` is the value after formatted by the Formatter -->
          <a :href="`#/issues/`" @click="getAssignee(data.item.assignee)" >{{ data.value }}</a>
        </template>
      </b-table>
  </div>
</template>

<script>
import VueJwtDecode from 'vue-jwt-decode';
import axios from "axios";
import { authMixin } from '../mixins/authMixin';

let token = localStorage.getItem('vue-authenticate.vueauth_token');
   
  export default {
    mixins: [ authMixin ],
    computed: {
      haveToken() {
        return token == null;
      }
    },

    data() {
      return {
        issues: null,
        users: null,
        mytoken: localStorage.getItem('vue-authenticate.vueauth_token'),
        // Note 'isActive' is left out and will not appear in the rendered table
        fields: [
          {
            key: 'titol',
            label: 'Títol',
            sortable: true
          },
          {
            key: 'tipus',
            label: 'Tipus',
            sortable: true
          },
          {
            key: 'prioritat',
            label: 'Prioritat',
            sortable: true,
          },
          {
            key: 'status',
            label: 'Status',
            sortable: true,
          },
          {
            key: 'assignee',
            label: 'Assignat a',
            sortable: true,
            formatter: 'getUsername',
          },
          {
            key: 'vote_set',
            label: 'Vots',
            sortable: true,
            formatter: 'countVotes'
          },
          {
            key: 'data_creacio',
            label: 'Data de creació',
            sortable: true,
          },
          {
            key: 'watch_set',
            label: 'Watching',
            sortable: true,
            formatter: 'iswatching'
          }
        ]
      };
    },
    methods: {
    getFilter: async function(f) {
      await axios
        .get(
          "https://asw-issue-tracker-2019.herokuapp.com/api/issues/?search="+f
        )
        .then(response => {
          this.issues = response.data;
          this.$$refs.table.refresh();
          return response.data;
        });
    },
    getIssues: async function() {
      // hauria de posar aqui les credencials i tal
      await axios
        .get(
          "https://asw-issue-tracker-2019.herokuapp.com/api/issues/"
        )
        .then(response => {
          this.issues = response.data;
          return response.data;
        });
    },
    getUsers: async function() {
      // hauria de posar aqui les credencials i tal
      await axios
        .get(
          "https://asw-issue-tracker-2019.herokuapp.com/api/user/"
        )
        .then(response => {
          this.users = response.data;
          return response.data;
        });
    },
    logout(){
      localStorage.removeItem('vue-authenticate.vueauth_token');
      window.location.href = '/';
    },
    countVotes(value) {
      return `${Object.keys(value).length}`
    },
    iswatching(value) {
      var watchers = `${Object.keys(value).map(function(e){
        return value[e].watcher;
      })}`;
      if (this.mytoken != null && watchers.includes(VueJwtDecode.decode(this.mytoken).user_id)) {return "Sí"}
      else {return "No"}
    },
    getUsername(value) {
      var size =  Object.keys(this.users).length;
      var i = 0;
      while (i < size) {
        if (this.users[i].id == value) return this.users[i].username;
        i++;
      }
      return ""
    },
    getMyIssues: async function() {
      await this.getIssues();
      var myid = VueJwtDecode.decode(this.mytoken).user_id;
      var size = Object.keys(this.issues).length;
      var i = 0;
      var issues2 = [];
      while (i < size) {
        var index = this.issues.findIndex(x => x.creator == myid);
        if (index !== -1) {
          issues2.push(this.issues[index])
          this.issues.splice(index, 1);
        }
        i++;
      }
      this.issues = issues2;
      this.$$refs.table.refresh();
    },
    getMyWatching: async function() {
      await this.getIssues();
      var myid = VueJwtDecode.decode(this.mytoken).user_id;
      var size = Object.keys(this.issues).length;
      var i = 0;
      var issues2 = [];
      while(i < size) {
        var sizewatchers = Object.keys(this.issues[i].watch_set).length;
        var j = 0;
        while (j < sizewatchers) {
          if (this.issues[i].watch_set[j].watcher == myid) issues2.push(this.issues[i])
          j++;
        }
        i++;
      }
      this.issues = issues2;
      this.$$refs.table.refresh();
    },
    getAssignee: async function(value) {
      await this.getIssues();
      var size = Object.keys(this.issues).length;
      var i = 0;
      var issues2 = [];
      while(i < size) {
        if (this.issues[i].assignee == value) issues2.push(this.issues[i])
        i++;
      }
      this.issues = issues2;
      this.$$refs.table.refresh();
    }
  },
  mounted() {
    this.getIssues();
    this.getUsers();
    this.countVotes();
    this.iswatching();
    this.getUsername();
  }
  }
</script>

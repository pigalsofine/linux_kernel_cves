<template>
  <div class="hello">
    <h1>CVEs in Stream {{this.stream}}</h1>
    <div id="content">
      <v-list>
        <v-list-group
          v-for="(data, id) in contents"
          :key="id"
        >
          <template slot="activator">
            <v-list-tile>
              <v-list-tile-content>
                <v-list-tile-title>Fixed in {{ id }}</v-list-tile-title>
              </v-list-tile-content>
            </v-list-tile>
          </template>
          <v-container fluid>
            <v-layout
              flexbox
              row
              fill-height
              wrap
            >
              <v-flex 
                xs12 sm6 m6 l4
                v-for="(details, cveid) in data"
                :key="cveid"
              >
                <v-card class="cve-card">
                  <v-card-title>
                    <span class="headline">{{cveid}}</span>
                  </v-card-title>
                  <v-card-text>
                    <span>{{details.cmt_msg}}</span>
                    <span>{{details.cmt_id}}</span>
                  </v-card-text>
                  <v-card-actions v-if="details.cmt_id">
                    <v-spacer></v-spacer>
                    <v-btn icon flat color="green"
                      :to="'/cves/' + cveid"
                    >
                    <v-icon>link</v-icon>
                    </v-btn>
                    <v-btn flat icon color="green"
                      v-clipboard:copy="details.cmt_id"
                      v-clipboard:success="onCopy"
                      v-clipboard:error="onError"
                    >
                      <v-icon>file_copy</v-icon>
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-flex>
            </v-layout>
          </v-container>
        </v-list-group>
      </v-list>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import Vue from 'vue'
import VueClipboard from 'vue-clipboard2'

Vue.use(VueClipboard)

export default {
  data () {
    return {
      stream: [],
      contents: [],
      errors: [],
      sorting: 'most recent',
      message: 'copy'
    }
  },
  created () {
    this.load()
  },
  watch: {
    '$route' () {
      this.load()
    }
  },
  methods: {
    onCopy: function (e) {
      alert('Commit ID copied to clipboard: ' + e.text)
    },
    onError: function () {
      alert('Failed to copy text.')
    },
    load: function () {
      this.stream = this.$route.path.split('/')[2]
      document.title = 'Linux Kernel CVEs | ' + 'CVEs in Stream ' + this.stream
      var url = this.$apiBaseUrl + 'data/stream_data.json'
      axios.get(url)
        .then(response => {
          // JSON responses are automatically parsed.
          var stream = response.data[this.stream]
          if (stream == null) {
            this.$router.push('/404')
          }
          this.contents = stream
        })
        .catch(e => {
          this.errors.push(e)
        })
    },
  },
  filters: {
    trim: function (value) {
      if (!value) return ''
      return value.substring(0, 7)
    }
  },
  computed: {
    orderedStreams: function ()  {
      var sortedStreams = this.contents
      delete sortedStreams['outstanding']
      if (this.sorting === 'most recent') {
        var reversed = {}
        var keys = Object.keys(this.contents).reverse()
        keys.forEach((key) => {
          reversed[key] = this.contents[key]
        })
        return reversed
      }
      return this.contents
    },
    outstanding: function () {
      return this.contents['outstanding']
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.cve-card {
  margin: 10px;
}
</style>

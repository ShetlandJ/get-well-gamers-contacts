<template>
  <div class="hello">

    <form class="searchForm" v-on:submit.prevent="submitSearch">
      <input class="search-box" type="text" v-model="searchQuery" placeholder="Search hospital or person" @keyup="submitSearch">
    </form>

    <div class="results-panel">

      <div class="panel-item" v-for="item in searchResults">
        <p><strong>Hospital:</strong> {{item.hospitalName}}</p>
        <p><strong>Ward:</strong> {{item.hospitalWard}}</p>
        <p><strong>Play Specialist:</strong> {{item.hps}}
          <a v-bind:href="'mailto:' + item.hpsEmail">(email)</a>
        </p>
        <p><strong>Phone Number:</strong> {{item.hospitalNumber}}</p>
      </div>

    </div>
  </div>
</template>


<script>

export default {
  name: 'Search',
  data () {
    return {
      contactData: [],
      isResult: false,
      searchQuery: '',
      searchResults: []
    }
  },
  methods: {
    async fetchAsync () {
      let response = await fetch('https://spreadsheets.google.com/feeds/list/1j6Aa0FAU3-AKpCmAlzM0YatTxKSfCOfuJey5kzt5tpU/od6/public/basic?alt=json');
      let data = await response.json();

      var contactObject = this.renderContactObject(data['feed']['entry']);
    },

    submitSearch() {
      var results = []
      var searchTerm = this.searchQuery.toLowerCase();
      this.contactData.filter(function(item) {
        var hospitalName = item.hospitalName.toLowerCase().includes(searchTerm);
        var hospitalHPS = item.hps.toLowerCase().includes(searchTerm);

        if (hospitalName || hospitalHPS) {
          results.push(item);
        }
      });
      if (this.searchQuery === "") {
        this.searchResults = ""
      } else {
        this.searchResults = results;
      }
    },

    renderContactObject(contactObject) {
      for (var contact of contactObject) {

        var hospitalDetails = contact.content['$t'].split(',');
        var hps = contact.title['$t'];
        for (var item of hospitalDetails) {
          if (item.includes('hospitalname')) {
            var index = hospitalDetails.indexOf(item);
            var hospitalName = hospitalDetails[index].slice(14);
          } else if (item.includes('ward:')) {
            var index = hospitalDetails.indexOf(item);
            var hospitalWard = hospitalDetails[index].slice(7);
          } else if (item.includes('number')) {
            var index = hospitalDetails.indexOf(item);
            var hospitalNumber = hospitalDetails[index].slice(9, 22);
          } else if (item.includes('email')) {
            var index = hospitalDetails.indexOf(item);
            var hpsEmail = hospitalDetails[index].slice(8)
          }
        }
        var contactItem = {
          "hps": hps,
          "hpsEmail": hpsEmail,
          "hospitalName": hospitalName,
          "hospitalWard": hospitalWard,
          "hospitalNumber": hospitalNumber
        }
        this.contactData.push(contactItem);
      }

    }
  },
  mounted() {
    this.fetchAsync()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.results-panel {
  display: flex;
  flex-wrap: wrap;
}

.search-box {
  font-size: 20px;
  width: 220px;
}

.panel-item {
  margin: 5px;
  padding: 10px;
  background-color: #F0F3FE;
  border-radius: 10px;
  border: 3px solid transparent;
  text-align: left;
}

.panel-item:hover {
  border: 3px inset #4C71F3;
}

h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>

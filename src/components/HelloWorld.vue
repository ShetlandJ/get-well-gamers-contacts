<template>
  <div class="hello">

    <form class="searchForm" v-on:submit.prevent="submitSearch">
      <input type="text" v-model="searchQuery" placeholder="Type here" @keyup="submitSearch">
    </form>

    <div class="" v-for="item in searchResults">
      <p>Hospital: {{item.hospitalName}}</p>
      <p>Ward: {{item.hospitalWard}}</p>
      <p>Play Specialist: {{item.hps}}</p>
      <p>Play Specialist: {{item.hpsEmail}}</p>
      <p>Phone Number: {{item.hospitalNumber}}</p>
      <hr/>
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
      var searchTerm = this.searchQuery.toLowerCase()
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

    }
  },
  mounted() {
    this.fetchAsync()
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
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

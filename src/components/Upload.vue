<template>
    <form id="app" @submit.prevent="handleSubmit()">
        <div class="container">
            <div class="panel panel-sm">
                <div class="panel-heading"> 
                    <h4>CSV Import</h4>
                </div>
            <div class="panel-body">
                <div class="form-group">
                <label for="csv_file" class="control-label col-sm-3 text-right">CSV file to import</label>
                <div class="col-sm-9">
                    <input type="file" id="csv_file" name="csv_file" class="form-control" ref="file" @change="loadCSV($event)">
                </div>
                </div>
                <div class="col-sm-offset-3 col-sm-9">
                <div class="checkbox-inline">
                    <label for="header_rows">
                    <input type="checkbox" id="header_rows"> File contains header row?</label>
                </div>
                </div>
                <div class="col-sm-offset-3 col-sm-9" id="buttons">
                    <!-- <button class="btn btn-primary" id="parsing-button">Parse CSV</button> -->
                    <button type="submit" class="btn btn-primary" id="parsing-button">Submit</button>
                </div>
                <table v-if="parse_csv">
                    <thead>
                        <tr>
                        <th colspan="{{parse_header.length}}" class="table-title">Imported Data</th>
                        </tr>
                        <tr>
                        <th v-for="key in parse_header"
                            @click="sortBy(key)"
                            :class="{ active: sortKey == key }">
                            {{ key | capitalize }}
                            <span class="arrow" :class="sortOrders[key] > 0 ? 'asc' : 'dsc'">
                            </span>
                        </th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="csv in parse_csv">
                        <td v-for="key in parse_header">
                            {{csv[key]}}
                        </td>
                        </tr>
                    </tbody>
                </table>
            </div>
            </div>
        </div>
        <!-- <button type="submit" class="btn btn-primary" id="parsing-button">Submit</button> -->
    </form>
</template>

<script>
import axios from 'axios'


export default {
  name: 'App',
  data() {
    return {
      channel_name: '',
      channel_fields: [],
      channel_entries: [],
      parse_header: [],
      parse_csv: [],
      sortOrders:{},
      sortKey: '',
      'api': 'http://localhost://3000/api',
      files: [],
      upload_status: '',
      filename: ''
    };
  },
  filters: {
    capitalize: function (str) {
      return str.charAt(0).toUpperCase() + str.slice(1)
    }
  },
  methods: {
    sortBy: function (key) {
      let vm = this
      vm.sortKey = key
      vm.sortOrders[key] = vm.sortOrders[key] * -1
    },
    csvJSON(csv){
      let vm = this
      let lines = csv.split("\n")
      let result = []
      let headers = lines[0].split(",")
      vm.parse_header = lines[0].split(",") 
      lines[0].split(",").forEach(function (key) {
        vm.sortOrders[key] = 1
      })
      
      lines.map(function(line, indexLine){
        if (indexLine < 1) return // Jump header line
        
        let obj = {}
        let currentline = line.split(",")
        
        headers.map(function(header, indexHeader){
          obj[header] = currentline[indexHeader]
        })
        
        result.push(obj)
      })
      
      result.pop() // remove the last item because undefined values
      
      return result // JavaScript object
    },
    loadCSV(e) {
      let vm = this
      this.filename = this.$refs.file.files[0]
      if (window.FileReader) {
        let reader = new FileReader();
        reader.readAsText(e.target.files[0]);
        // Handle errors load
        reader.onload = function(event) {
          let csv = event.target.result;
          vm.parse_csv = vm.csvJSON(csv)
        };
        reader.onerror = function(evt) {
          if(evt.target.error.name == "NotReadableError") {
            alert("Cannot read file!");
          }
        };
      } else {
        alert('FileReader are not supported in this browser.');
      }
    },
    // uploadedFile(){
    //     this.filename = this.$refs.file.files[0]
    // },
    handleSubmit(){
        console.log("Submitted")
        let formData = new FormData()
        formData.append("csv", this.filename)

        axiosConfig = {
            headers: {
                'Content-Type': 'multipart/form-data'
            }
        }

        axios.post(api + '/files/', formData, axiosConfig )
            .then( response => {
              console.log(response)
              this.upload_status('File Uploaded Successfully')
            }).catch(error => {
              this.upload_status('File Uploaded Successfully')
            })

    }
  }
}

</script>

<style scoped>
    #app, .container {
        margin: 0;
        padding: 0;
    }
    .container {
        margin: 32px auto;
        background-color: #161bae86;
    }
    .panel {
        border: 2px solid #dfdfdf;
        box-shadow: rgba(0, 0, 0, 0.15) 0 1px 0 0;
        margin: 10px;
    }
    .panel.panel-sm {
        max-width: 700px;
        margin: 10px auto;
    }
    .panel-heading {
        border-bottom: 2px solid #dfdfdf;
    }
    .panel-heading h4{
        font-size: 20px;
        text-align: center;
        color: #ffffff;
    }
    .panel-heading h1, .panel-heading h2, .panel-heading h3, .panel-heading h4, .panel-heading h5, .panel-heading h6 {
        margin: 0;
        padding: 0;
    }
    .panel-body .checkbox-inline {
        padding: 20px 30px;
    }
    
    input{
        padding: 20px;
        background-color: rgba(100, 135, 73, 0.696);

    }

    table {
        font-family: arial, sans-serif;
        border-collapse: collapse;
        width: 100%;
    }

    td, th {
    border: 1px solid #dddddd;
    text-align: left;
    padding: 8px;
    }

    tr:nth-child(even) {
    background-color: #dddddd;
    }
    #buttons{
        margin-right: 100px;
        display: flex;
        justify-content: space-evenly;
    }
    #parsing-button{
        cursor: pointer;
        text-decoration: none;
        font-size: 1.4rem;
        padding: 6px;
        border: none;
        border-radius: 10px;
        background-color: rgb(100, 135, 73);
        color: #dddddd;
        font-weight: 600;
    }
</style>
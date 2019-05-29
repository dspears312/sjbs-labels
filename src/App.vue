<template>
  <v-app id="inspire" v-if="!printMode">
    <v-content>
      <v-container fluid fill-height>
        <v-layout align-center justify-center>
          <v-flex xs12 sm10 md8>
            <v-card class="elevation-12">
              <div v-if="e1 == 0 && !printMode">
                <v-card-title>
                  <h2 class="headline">Avery 5160 Label Setup</h2>
                </v-card-title>
                <v-card-text>
                  <p class="mb-4">Enter the information for the books that need a label.</p>
                  <input type="file" @change="parseCSV" ref="fileUpload" style="display: none;">
                  <div style="font-family: 'Libre Barcode 39 Text'; position: absolute; opacity: 0.01;">Hello</div>
                  <table style="width: 100%">
                    <thead>
                      <tr>
                        <th style="text-align: left;">Title</th>
                        <th style="text-align: left;">Author</th>
                        <th style="text-align: left;">Barcode</th>
                        <th style="text-align: left;">Actions</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr v-for="(label, index) in labels" :key="index">
                        <td>
                          <v-text-field v-model="label.title" placeholder="Title"></v-text-field>
                        </td>
                        <td>
                          <v-text-field v-model="label.author" placeholder="Author"></v-text-field>
                        </td>
                        <td>
                          <v-text-field v-model="label.barcode" placeholder="12345"></v-text-field>
                        </td>
                        <td>
                          <v-btn icon @click="labels.splice(index, 1)">
                            <v-icon>delete</v-icon>
                          </v-btn>
                        </td>
                      </tr>
                    </tbody>
                  </table>
                  <v-btn class="mt-3" color="primary" block @click="labels.push({title: '', author: '', barcode: ''})">
                    Add Book</v-btn>
                </v-card-text>
                <v-card-actions class="ma-2">
                  <v-btn color="primary" @click="uploadCSV">Upload Koha CSV</v-btn>
                  <v-spacer></v-spacer>
                  <v-btn color="primary" @click="e1=1">Continue to Print Setup</v-btn>
                </v-card-actions>
              </div>
              <div v-if="e1 == 1 && !printMode">
                <v-card-title>
                  <h2 class="headline">Print Setup</h2>
                </v-card-title>
                <v-card-text>
                  <p class="mb-4">Enter the row and column the first label should be printed on.</p>
                  <v-text-field block label="Row" v-model="row"></v-text-field>
                  <v-text-field block label="Column" v-model="column"></v-text-field>
                </v-card-text>
                <v-card-actions class="ma-2">
                  <v-spacer></v-spacer>
                  <v-btn @click="printLabels" color="primary">Print Labels</v-btn>
                </v-card-actions>
              </div>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
  </v-app>
  <div v-else>
    <v-btn class="startOverButton" color="primary" v-if="printMode" @click="startOver">Start Over</v-btn>
    <div id="page">
      <div v-for="label in labels" class="print-label">
        <p class="label-text">{{label.title}}</p>
        <p class="label-text">{{label.author}}</p>
        <p class="label-barcode" v-if="label.barcode !== ''">*{{label.barcode}}*</p>
      </div>
    </div>
  </div>
</template>

<script>
  import Papa from 'papaparse'
  export default {
    data() {
      return {
        labels: [{
          title: '',
          author: '',
          barcode: ''
        }],
        e1: 0,
        row: 1,
        column: 1,
        printMode: false
      }
    },
    methods: {
      printLabels() {
        window.print()
      },
      uploadCSV() {
        this.$refs.fileUpload.click()
      },
      parseCSV() {
        this.labels = []
        Papa.parse(this.$refs.fileUpload.files[0], {
          complete: (results) => {
            for (let label of results.data) {
              if (label[0] && label[1] && label[3]) {
                this.labels.push({
                  title: label[0],
                  author: label[1],
                  barcode: label[3]
                })
              }
            }
          }
        })
      },
      printLabels() {
        let labelsToAdd = ((this.row - 1) * 3) + (this.column - 1)
        for (let i = 0; i < labelsToAdd; i++) {
          this.labels.unshift({ title: '', author: '', barcode: '' })
        }
        this.printMode = true;
        this.$nextTick(() => { window.print() })
      },
      startOver() {
        this.labels = [{ title: '', author: '', barcode: '' }]
        this.e1 = 0;
        this.printMode = false
        this.row = 1;
        this.column = 1;
      }
    }
  }
</script>
<style>
  .print-label {
    width: 2.625in;
    height: 1in;
    overflow: hidden;
    display: inline-block;
    background: #fff;
    box-sizing: border-box;
    margin-left: 0.07in;
    margin-right: 0.07in;
    line-height: 16px;
    border: solid #000 0;
    font-family: Arial, Helvetica, sans-serif;
    padding: 0.075in;
    text-align: center;
  }

  .label-text {
    text-overflow: ellipsis;
    line-height: 20px;
    font-size: 16px;
    margin: 0;
    padding: 0;
    overflow: hidden;
    width: 2.5in;
    white-space: nowrap;
    margin: 0 auto;
    font-family: 'Roboto', sans-serif;
  }

  .label-barcode {
    font-family: 'Libre Barcode 39 Text';
    font-size: 40px;
    line-height: 40px;
    margin-top: 2px;
  }

  #page .label:nth-child(30n + 30) {
    page-break-after: always;
  }

  #page {
    width: 8.5in;
    padding-top: 0.5in;
    padding-left: 0.07975in;
    padding-right: 0.07975in;
    background: #fff;
    box-sizing: border-box;
    margin: 0 auto;
    display: block;
    line-height: 0;
  }

  @media print {
    #page {
      padding: 0;
      width: auto;
      box-sizing: content-box;
    }

    body {
      background: #fff;
    }
    .startOverButton {
      display: none;
    }
  }

  @page {
    size: 8.5in 11in;
    margin-top: 0.5in;
    margin-left: 0.07975in;
    margin-right: 0.07975in;
  }
</style>
<template>
  <div class="container">
    <v-layout
      column
      justify-center
      align-center
    >
      <v-switch v-model="checkbox1" />
      <v-simple-table>
        <template v-slot:default>
          <thead>
            <tr>
              <th class="text-left">
                なまえ  
              </th>
              <th class="text-left">
                年齢
              </th>
              <th class="text-left">
                status
              </th>
              <th class="text-left">
                detail
              </th>
            </tr>
          </thead>
          <tbody>
            <template v-for="row in rows">
              <tr v-if="!checkbox1||!(row.status)" :key="row.rowNumber">
                <td>{{ row.name }}</td>
                <td>
                  {{ row.age }}
                </td>
                <td>
                  {{ row.status }}
                  <div>
                    <v-btn
                      v-if="!row.status"
                      small
                      color="green"
                      @click.stop="openDialog({
                        title: row.name+'さん',
                        message: '面接に変更しますか',
                        callbackAgree:setInterview,
                        arg: row,
                      })"
                    >
                      面接
                    </v-btn>
                    <v-btn
                      v-if="row.status === '面接'"
                      small
                      color="red"
                      @click.stop="openDialog({
                        title: row.name+'さん',
                        message: '合格に変更しますか',
                        callbackAgree:setPass,
                        arg: row,
                      })"
                    >
                      合格
                    </v-btn>

                    <v-btn
                      v-if="row.status !== '不合格'&&row.status !== '合格'"
                      small
                      color="primary"
                      @click.stop="openDialog({
                        title: row.name+'さん',
                        message: '不合格に変更しますか',
                        callbackAgree:setFailure,
                        arg: row,
                        status:'不合格'
                      })"
                    >
                      不合格
                    </v-btn>
                  </div>
                </td>
                <td>
                  <v-btn
                    small
                    color="red"
                    @click.stop="openDialogDetail(row)"
                  >
                    learn more
                  </v-btn>
                </td>
              </tr>
            </template>
          </tbody>
        </template>
      </v-simple-table>
      <!-- dialogDetailここから -->
      <v-row justify="center">
        <v-dialog v-model="dialogDetail" persistent max-width="600">
          <v-card>
            <v-card-title>
              <span class="headline">User Profile</span>
            </v-card-title>
            <v-card-text>
              <v-container>
                <v-row>
                  <v-col md="4">
                    <h3>{{ dialogInfo.name }}</h3>
                  </v-col>
                  <v-col md="4">
                    <h3>{{ dialogInfo.age }}歳</h3>
                  </v-col>
                  <v-col md="4">
                    <h3>{{ dialogInfo.experience }}</h3>
                  </v-col>
                  <v-col cols="12">
                    <p>＜志望動機＞</p>
                    <v-text>{{ dialogInfo.passion }}</v-text>
                  </v-col>
                  <v-btn color="blue darken-1" text @click.stop=" dialogDetail= false">
                    Close
                  </v-btn>
                  <v-container />
                </v-row>
              </v-container>
            </v-card-text>
          </v-card>
        </v-dialog>
      </v-row>
    </v-layout>
    <v-dialog
      v-model="dialog"
      max-width="290"
    >
      <v-card>
        <v-card-title class="headline">
          {{ dialogInfo.title }}
        </v-card-title>
        <v-card-text>
          {{ dialogInfo.message }}
        </v-card-text>

        <v-card-actions>
          <v-btn
            color="green darken-1"
            text
            @click="onDisagree()"
          >
            Disagree
          </v-btn>

          <v-btn
            color="green darken-1"
            text
            @click="onAgree()"
          >
            Agree
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>

export default {
  data () {
    return {
      row: [],
      rows: [],
      select: ['合格', '不合格', '面接'],
      sheet: undefined,
      headerValues: [],
      dialog: false,
      dialogInfo: {
        callbackAgree: () => {},
        callbackDisagree: () => {},
        arg: {},
        message: '',
        title: '',
        status: '面接'

      },
      dialogDetail: false,
      checkbox1: false
    }
  },

  async mounted () {
    const { GoogleSpreadsheet } = require('google-spreadsheet')

    // spreadsheet key is the long id in the sheets URL
    const doc = new GoogleSpreadsheet(process.env.GOOGLE_PRIVATE_KEY)

    // OR load directly from json file if not in secure environment
    await doc.useServiceAccountAuth(require('../assets/gspread-test-272603-f40ecae24590.json'))

    await doc.loadInfo() // loads document properties and worksheets

    this.sheet = doc.sheetsByIndex[0] // or use doc.sheetsById[id]
    this.rows = await this.sheet.getRows() // can pass in { limit, offset }
    const str = this.rows[0].a1Range
    console.log(str)
    // read/write row values
    const result = str.substr(str.length - 2, 1)
    console.log(this.sheet.headerValues)
    this.headerValues = this.sheet.headerValues
    await this.sheet.loadCells('A1:' + result + (this.rows.length + 1))
    console.log('A1:' + result + (this.rows.length + 1))
  },
  methods: {

    async setCellValue (row, column, value) {
      console.log(row, column, value)
      try {
        const a1 = await this.sheet.getCell(row, column)
        a1.value = value
        await this.sheet.saveUpdatedCells()
      } catch (err) {
        console.error(err)
      }
    },
    openDialog (dialogInfo) {
      this.dialog = true
      this.dialogInfo = dialogInfo
      console.log(this.rows)
    },

    onAgree () {
      this.dialog = false
      this.dialogInfo.callbackAgree(this.dialogInfo.arg)
    },

    onDisagree () {
      this.dialog = false
      if (this.dialogInfo.callbackDisagree) {
        this.dialogInfo.callbackDisagree(this.dialogInfo.arg)
      }
    },

    async setRowValue (row, key, value) {
      await this.setCellValue(row.rowNumber - 1, this.headerValues.indexOf(key), value)
      const index = this.rows.indexOf(row)
      row[key] = value
      // this.rows.splice(index, 1, row)
      this.$set(this.rows, index, row)
    },

    async setPass (row) {
      await this.setRowValue(row, 'status', '合格')
    },
    async setFailure (row) {
      await this.setRowValue(row, 'status', '不合格')
    },

    async setInterview (row) {
      await this.setRowValue(row, 'status', '面接')
    },

    openDialogDetail (dialogInfo) {
      this.dialogDetail = true
      this.dialogInfo = dialogInfo
      console.log(dialogInfo.age)
    }
  }
}
</script>

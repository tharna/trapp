<template>
  <div>
    <div style="text-align: center;">
      <el-button type="primary" @click="addQuestVisible = true" round>Lisää viikkotehtävä</el-button>
    </div>
    <hr>
  <el-table
    :data="quests"
    :default-sort = "{prop: 'questActive', order: 'descending'}"
    style="width: 100%">
    <el-table-column
      prop="name"
      label="Tehtävä"
      sortable>
    </el-table-column>
    <el-table-column
      prop="groupID"
      label="Heimo"
      width=120
      :filter-method="filterGroup"
      :filters="[{ text: 'Yhteinen', value: 'Yhteinen'}, { text: 'Karhut', value: 'Karhut' }, { text: 'Ketut', value: 'Ketut' }, { text: 'Joutsenet', value: 'Joutsenet' }, { text: 'Ilvekset', value: 'Ilvekset' }, { text: 'Saukot', value: 'Saukot' }, { text: 'Sudet', value: 'Sudet' }, { text: 'Hirvet', value: 'Hirvet' }, { text: 'Jalopeurat', value: 'Jalopeurat' }, { text: 'NPC', value: 'NPC' }, { text: 'PJ', value: 99}]"
      filter-placement="bottom-end"
      sortable>
    </el-table-column>
    <el-table-column
      prop="questActive"
      label="Aloituspäivä"
      :formatter="formatDate"
      width=140
      sortable>
    </el-table-column>
    <el-table-column width=200>
      <template slot-scope="scope">
            <el-progress :percentage="scope.row.progress" :stroke-width="16"></el-progress>
      </template>
    </el-table-column>
    <el-table-column>
      <template slot-scope="scope">
        <el-button type="primary" round @click="editQuest(scope.row.questID)">Muokkaa</el-button> 
        <el-button type="primary" round @click="showQuest(scope.row.questID)">Näytä</el-button> 
      </template>
    </el-table-column>
  </el-table>

    <el-dialog
        title="Lisää viikkotehtävä"
        :visible.sync="addQuestVisible"
        :append-to-body="true"
        width="80%">
        Tehtävän nimi
        <el-input v-model="questName" placeholder="Tehtävän nimi"></el-input>
        Tehtävän kuvaus
        <el-input v-model="questDesc" placeholder="Kuvaus" type="textarea"></el-input>
        Tarina
        <el-input v-model="questStory" placeholder="Tarina" type="textarea"></el-input>
        Onnistumisviesti
        <el-input v-model="questSuccess" placeholder="Onnistumisviesti" type="textarea"></el-input>
        Epäonnistumisviesti
        <el-input v-model="questFailure" placeholder="Epäonnistumisviesti" type="textarea"></el-input>
        <el-row>
          <el-col :xs=24 :sm=12>
            <el-radio v-model="questScope" label="1">Yksilö</el-radio>
            <el-radio v-model="questScope" label="2">Heimo</el-radio>
          </el-col>
          <el-col :xs=24 :sm=12>
            <el-select v-model="questGroup" placeholder="Heimo">
              <el-option
                  v-for="item in groups"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value">
              </el-option>
            </el-select>
 
         </el-col>
        </el-row>
        <el-row>
          <el-col :xs=24 :sm=12>
            <el-radio v-model="questRepeat" label="1">Viikossa</el-radio>
            <el-radio v-model="questRepeat" label="2">
            <el-input-number v-model="questDays" placeholder="Päivää"></el-input-number> päivänä</el-radio>
          </el-col>
          <el-col :xs=24 :sm=12>
            <el-radio v-model="questType" label="1">Suoritus</el-radio>
            <el-radio v-model="questType" label="2">Määrä</el-radio>
            <br/>Vaadittu suoritus:<br/>
              <el-input-number  v-model="amount" :min="1" :step="1"></el-input-number><br/>
            <el-input v-show="questType == 2" v-model="questMeasure" placeholder="Yksikkö"></el-input>
 
 
         </el-col>
        </el-row>
        <el-row>
        <el-col :span=24>
          <el-checkbox v-model="grandQuest">Suurtehtävä</el-checkbox>
         </el-col>
        </el-row>
        <el-row>
        <el-col :xs=24 :sm=12 :lg=6>
        Tehtävän aloituspäivä<br>
        <el-date-picker
                  v-model="questActive"
                  format="d.M.yyyy"
                  :picker-options="datePickerOpts"
                  placeholder="Tehtäväviikko">
        </el-date-picker>
         </el-col>
        <el-col :xs=24 :sm=12 :lg=6>
        Tehtävän julkaisupäivä<br>
        <el-date-picker
                  v-model="questPublish"
                  format="d.M.yyyy"
                  type="date"
                  :picker-options="datePickerOpts"
                  placeholder="Tehtävän julkaisu">
        </el-date-picker>
         </el-col>
        </el-row>
        <el-row>
          <el-col :span="24">
         </el-col>
        </el-row>
        <span slot="footer" class="dialog-footer">
          <el-button @click.native="addQuestVisible = false" round>Peruuta</el-button>
          <el-button type="primary"  :loading="loadingSend" round @click.prevent="postAddQuest">Lähetä</el-button>
        </span>
    </el-dialog>

    <el-dialog
        title="Muokkaa viikkotehtävää"
        :visible.sync="editQuestVisible"
        :append-to-body="true"
        width="80%">
        Tehtävän nimi
        <el-input v-model="EquestName" placeholder="Tehtävän nimi"></el-input>
        Tehtävän kuvaus
        <el-input v-model="EquestDesc" placeholder="Kuvaus" type="textarea"></el-input>
        Tarina
        <el-input v-model="EquestStory" placeholder="Tarina" type="textarea"></el-input>
        Onnistumisviesti
        <el-input v-model="EquestSuccess" placeholder="Onnistumisviesti" type="textarea"></el-input>
        Epäonnistumisviesti
        <el-input v-model="EquestFailure" placeholder="Epäonnistumisviesti" type="textarea"></el-input>
        <el-row>
          <el-col :xs=24 :sm=12>
            <el-radio v-model="EquestScope" label="1">Yksilö</el-radio>
            <el-radio v-model="EquestScope" label="2">Heimo</el-radio>
          </el-col>
          <el-col :xs=24 :sm=12>
            <el-select v-model="EquestGroup" placeholder="Heimo">
              <el-option
                  v-for="item in groups"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value">
              </el-option>
            </el-select>
 
         </el-col>
        </el-row>
        <el-row>
          <el-col :xs=24 :sm=12>
            <el-radio v-model="EquestRepeat" label="1">Viikossa</el-radio>
            <el-radio v-model="EquestRepeat" label="2">
            <el-input-number v-model="EquestDays" placeholder="Päivää"></el-input-number> päivänä</el-radio>
          </el-col>
          <el-col :xs=24 :sm=12>
            <el-radio v-model="EquestType" label="1">Suoritus</el-radio>
            <el-radio v-model="EquestType" label="2">Määrä</el-radio>
            <br/>Vaadittu suoritus:<br/>
            <el-input-number  v-model="Eamount" :min="1" :step="1"></el-input-number>
            <el-input v-show="EquestType == 2" v-model="EquestMeasure" placeholder="Yksikkö"></el-input>
 
 
         </el-col>
        </el-row>
        <el-row>
        <el-col :xs=24 :sm=12 :lg=6>
            Tehtävän aloituspäivä<br>
            <el-date-picker
                                              v-model="EquestActive"
                                              format="d.M.yyyy"
                                              :picker-options="datePickerOpts"
                                              placeholder="Tehtäväviikko">
            </el-date-picker>
          </el-col>
        <el-col :xs=24 :sm=12 :lg=6>
            Tehtävän julkaisupäivä<br>
            <el-date-picker
                                              v-model="EquestPublish"
                                              format="d.M.yyyy"
                                              type="date"
                                              :picker-options="datePickerOpts"
                                              placeholder="Tehtävän julkaisu">
            </el-date-picker>
          </el-col>
          </el-row>
        <el-row>
        <el-col :span=24>
          <el-checkbox v-model="EgrandQuest">Suurtehtävä</el-checkbox>
         </el-col>
        </el-row>
          <el-row>
            <el-col :span="24">
            </el-col>
          </el-row>
          <span slot="footer" class="dialog-footer">
          <el-button @click.native="editQuestVisible = false" round>Peruuta</el-button>
          <el-button type="danger" :loading="loadingDel" @click.native="postDelQuest" round>Poista</el-button>
          <el-button type="primary" :loading="loadingSend" round @click.prevent="postEditQuest">Tallenna</el-button>
        </span>
    </el-dialog>
    <el-dialog
        :title="currentQuest.name"
        :visible.sync="showQuestActivityVisible"
        :append-to-body="true"
        width="80%">
      <h2>Tehtävän kuvaus</h2>
      <div v-html="currentQuest.questDesc"></div>
      <h2>Tarina</h2>
      <div v-html="currentQuest.questStory"></div>
      <div v-if="currentQuest.status == 'success'" v-html="currentQuest.questSuccess"></div>
        <div v-if="currentQuest.status == 'failure'" v-html="currentQuest.questFailure"></div>
          <span slot="footer" class="dialog-footer">
      <el-button @click.native="showQuestActivityVisible = false" round>OK</el-button>
          </span>

    </el-dialog>




  </div>
</template>
<script>
import Vue from 'vue'
import axios from 'axios'
import moment from 'moment'
Vue.use(axios)
export default {
  data () {
    return {
      quests: [
      ],
      loadingSend: false,
      loadingDel: false,
      addQuestVisible: false,
      editQuestVisible: false,
      questFilter: 'Kaikki',
      questName: '',
      questType: '',
      questDesc: '',
      questGroup: '',
      questStory: '',
      questSuccess: '',
      questFailure: '',
      questDays: 0,
      amount: 0,
      questMeasure: '',
      questRepeat: '',
      questActive: '',
      questPublish: '',
      grandQuest: false,
      datePickerOpts: {
        firstDayOfWeek: 1
      },
      questScope: '1',
      groupFilter: [{
        value: 'Kaikki',
        label: 'Kaikki'
      }, {
        value: 'Ketut',
        label: 'Ketut'
      }, {
        value: 'Karhut',
        label: 'Karhut'
      }, {
        value: 'Joutsenet',
        label: 'Joutsenet'
      }, {
        value: 'Ilvekset',
        label: 'Ilvekset'
      }, {
        value: 'Saukot',
        label: 'Saukot'
      }, {
        value: 'Hirvet',
        label: 'Hirvet'
      }, {
        value: 'Sudet',
        label: 'Sudet'
      }, {
        value: 'Jalopeurat',
        label: 'Jalopeurat'
      }, {
        value: 'Yhteinen',
        label: 'Yhteinen'
      }, {
        value: 'NPC',
        label: 'NPC'
      }],
      groups: [{
        value: 'Ketut',
        label: 'Ketut'
      }, {
        value: 'Karhut',
        label: 'Karhut'
      }, {
        value: 'Joutsenet',
        label: 'Joutsenet'
      }, {
        value: 'Ilvekset',
        label: 'Ilvekset'
      }, {
        value: 'Saukot',
        label: 'Saukot'
      }, {
        value: 'Hirvet',
        label: 'Hirvet'
      }, {
        value: 'Sudet',
        label: 'Sudet'
      }, {
        value: 'Jalopeurat',
        label: 'Jalopeurat'
      }, {
        value: 'Yhteinen',
        label: 'Yhteinen'
      }, {
        value: 'NPC',
        label: 'NPC'
      }],
      EquestName: ' ',
      EquestType: ' ',
      EquestDesc: ' ',
      EquestGroup: '',
      EquestStory: ' ',
      EquestSuccess: ' ',
      EquestFailure: ' ',
      EquestDays: ' ',
      Eamount: 0,
      EquestMeasure: ' ',
      EquestRepeat: ' ',
      EquestActive: ' ',
      EquestPublish: ' ',
      EquestScope: '1',
      EgrandQuest: false,
      EquestID: '',
      showQuestActivityVisible: false,
      currentQuest: {}
    }
  },
  methods: {
    getQuests: function () {
      axios.get('/admin/quest')
        .then(response => { this.quests = response.data.quests })
    },
    postAddQuest: function () {
      this.loadingSend = true
      axios.post('/admin/quest/add', {
        questName: this.questName,
        questType: this.questType,
        questGroup: this.questGroup,
        amount: parseInt(this.amount),
        questDesc: this.questDesc,
        questStory: this.questStory,
        questSuccess: this.questSuccess,
        questFailure: this.questFailure,
        questMeasure: this.questMeasure,
        questActive: this.questActive,
        questPublish: this.questPublish,
        questDays: this.questDays,
        questRepeat: this.questRepeat,
        questScope: this.questScope,
        grandQuest: this.grandQuest
      }).then(reponse => {
        Object.assign(this.$data, this.$options.data())
        this.$notify({
          title: 'Success',
          message: 'Viikkotehtävä lisätty',
          type: 'success'
        })
        this.addQuestVisible = false
        this.loadingSend = false
        this.getQuests()
      }).catch(err => {
        console.log(err)
        this.loadingSend = false
        this.$notify({
          title: 'Virhe',
          message: 'Tallennus ei onnistunut',
          type: 'error'
        })
      })
    },
    postEditQuest: function () {
      this.loadingSend = true
      axios.post('/admin/quest/update', {
        questID: this.EquestID,
        questName: this.EquestName,
        questType: this.EquestType,
        questGroup: this.EquestGroup,
        questGroupOld: this.EquestGroupOld,
        amount: parseInt(this.Eamount),
        questDesc: this.EquestDesc,
        questStory: this.EquestStory,
        questSuccess: this.EquestSuccess,
        questFailure: this.EquestFailure,
        questMeasure: this.EquestMeasure,
        questActive: this.EquestActive,
        questPublish: this.EquestPublish,
        questDays: this.EquestDays,
        questScope: this.EquestScope,
        questRepeat: this.EquestRepeat,
        grandQuest: this.EgrandQuest
      }).then(reponse => {
        Object.assign(this.$data, this.$options.data())
        this.$notify({
          title: 'Success',
          message: 'Viikkotehtävä tallennettu',
          type: 'success'
        })
        this.editQuestVisible = false
        this.loadingSend = false
        this.getQuests()
      }).catch(err => {
        console.log(err)
        this.loadingSend = false
        this.$notify({
          title: 'Virhe',
          message: 'Tallennus ei onnistunut',
          type: 'error'
        })
      })
    },
    editQuest: function (questID) {
      var quest = this.quests.find(quest => { return quest.questID === questID })
      this.EquestName = quest.name
      this.EquestType = quest.type
      this.EquestDesc = quest.questDesc
      this.EquestGroup = quest.groupID
      this.EquestGroupOld = quest.groupID
      this.EquestStory = quest.questStory
      this.EquestSuccess = quest.questSuccess
      this.EquestFailure = quest.questFailure
      this.EquestDays = quest.questDays
      this.Eamount = quest.amount
      this.EquestMeasure = quest.questMeasure
      this.EquestRepeat = quest.questRepeat
      this.EquestActive = quest.questActive
      this.EquestPublish = quest.questPublish
      this.EquestScope = quest.questScope
      this.EgrandQuest = quest.grandQuest
      this.EquestID = quest.questID
      this.editQuestVisible = true
    },
    postDelQuest: function () {
      this.loadingDel = true
      axios.post('/admin/quest/delete', {
        questID: this.EquestID,
        questGroup: this.EquestGroup
      }).then(reponse => {
        Object.assign(this.$data, this.$options.data())
        this.loadingDel = false
        this.$notify({
          title: 'Success',
          message: 'Viikkotehtävä poistettu',
          type: 'success'
        })
        this.editQuestVisible = false
        this.getQuests()
      }).catch(err => {
        console.log(err)
        this.loadingDel = false
        this.$notify({
          title: 'Virhe',
          message: 'Poisto ei onnistunut',
          type: 'error'
        })
      })
    },
    showQuest: function (questID) {
      this.currentQuest = this.quests.find(quest => { return quest.questID === questID })
      this.showQuestActivityVisible = true
    },
    filterGroup: function (value, row) {
      return row.groupID === value
    },
    formatDate: function (row, column, value, index) {
      return moment(String(value)).format('DD.MM.YYYY')
    }
  },
  filters: {
    date: function (value) {
      return moment(String(value)).format('DD.MM.YYYY')
    }
  },
  created () {
    this.getQuests()
  }

}
</script>
<style>
.el-progress__text {
  margin-left: 8px;
}
</style>

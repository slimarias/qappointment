<template>
  <div id="crudCardAppoinments">
    <!--Card content-->
    <div class="appt-card box" v-if="rowData">
      <!--Title-->
      <div class="appt-card__title row items-center justify-between">
        <!--Label-->
        <div class="box-title">{{ rowData.title }}</div>
        <!--Actions-->
        <btn-menu :actions="fieldActions" :action-data="row"/>
      </div>
      <!--Fields-->
      <div class="appt-card__fields">
        <!--Separator-->
        <q-separator class="q-my-md"/>
        <!--Title-->
        <div class="appt-card__fields-title box-title">{{ $tr('ui.label.information') }}</div>
        <!--Main Information Fields-->
        <div class="appt-card__fields-field q-py-xs" v-for="(field, keyField) in rowData.mainInformation" :key="$uid()">
          <label class="text-grey-7 q-mr-xs">{{ field.label }}:</label><label>{{ field.value }}</label>
        </div>
        <!--Information Fields-->
        <div class="appt-card__fields-field q-py-xs" v-for="(field, keyField) in rowData.informationFields"
             :key="keyField" v-if="showAllInfo && field">
          <label class="text-grey-7 q-mr-xs">{{ field.label }}:</label><label>{{ field.value }}</label>
        </div>
      </div>
      <!--Related user-->
      <div class="appt-card__related-users" v-if="showAllInfo && rowData.relatedUsers.length">
        <!--Separator-->
        <q-separator class="q-my-md"/>
        <!--Assigned to-->
        <div class="row q-gutter-y-sm">
          <div v-for="(user, keyUser) in rowData.relatedUsers" :key="keyUser" class="col-12">
            <q-item dense v-if="user" class="q-px-none">
              <q-item-section side class="q-pr-sm">
                <!--User Image-->
                <q-avatar size="40px">
                  <img :src="user.mediumImage"/>
                </q-avatar>
              </q-item-section>
              <q-item-section>
                <!--User Rol Name (assigneTo/Customer)-->
                <q-item-label caption>{{ user.rolName }}</q-item-label>
                <!--User FullName-->
                <q-item-label caption class="text-blue-grey"><b>{{ user.fullName }}</b></q-item-label>
              </q-item-section>
            </q-item>
          </div>
        </div>
      </div>
      <!--Recommendation info-->
      <div class="appt-card__recommendation" v-if="showAllInfo && rowData.recomendation">
        <!--Separator-->
        <q-separator class="q-my-sm"/>
        <!--Title-->
        <div class="box-title q-mb-sm">{{ $trp('ui.label.recommendation') }}</div>
        <!--Action-->
        <q-btn unelevated color="amber" rounded no-caps icon="fas fa-info"
               @click="$alert.warning({mode : 'modal', message : rowData.recomendation, title : $trp('ui.label.recommendation')})"
               :label="$tr('qappointment.layout.message.readRecomendations')"/>
      </div>
      <!--Conversation info-->
      <div class="appt-card__conversation" v-if="showAllInfo && rowData.conversationId">
        <!--Separator-->
        <q-separator class="q-my-sm"/>
        <!--Title-->
        <div class="box-title q-mb-sm">{{ $tr('ui.label.chat') }}</div>
        <!--Action-->
        <q-btn unelevated color="info" rounded no-caps icon="fas fa-comment-alt" @click="showConversation = true"
               :label="$tr('qappointment.layout.message.readFullConversation')"/>
      </div>
      <!--Rating Fields-->
      <div class="appt-card__fields" v-if="showAllInfo && rowData.ratingFields">
        <!--Separator-->
        <q-separator class="q-my-sm"/>
        <!--Title-->
        <div class="box-title">{{ $tr('ui.label.rating') }}</div>
        <!--Rating-->
        <dynamic-field v-if="rowData.ratingFields.ratingField" :field="rowData.ratingFields.ratingField"/>
        <!--Rating comment-->
        <div class="text-caption" v-if="rowData.ratingFields.rattingComments">
          {{ rowData.ratingFields.rattingComments }}
        </div>
      </div>
      <!--Actions got to appointment-->
      <div class="appt-card__actions text-center"
           v-if="(configApp.mode == 'ipanel') && showAllInfo && rowData.goToAppointment">
        <!--Separator-->
        <q-separator class="q-my-sm"/>
        <!--Action-->
        <q-btn unelevated color="green" rounded no-caps
               :label="$tr('qappointment.layout.message.goToAppointment')"
               :to="{name : 'qappointment.panel.appointments.index', params : {id : rowData.id}}"/>
      </div>
      <!--Actions-->
      <div class="appt-actions text-right">
        <!--Separator-->
        <q-separator class="q-mt-sm q-mb-md"/>
        <!--Buttons-->
        <div class="row justify-between q-gutter-sm">
          <!--Show more info-->
          <q-btn color="info" size="sm" rounded unelevated padding="xs md" @click="showAllInfo = !showAllInfo"
                 :label="$tr(`ui.message.${showAllInfo ? 'showLess' : 'showMore'}`)"
                 :icon="`fas fa-angle-double-${showAllInfo ? 'up' : 'down'}`"/>
        </div>
      </div>
    </div>
    <!--Conversation Content-->
    <q-dialog v-model="showConversation">
      <div style="width: 600px">
        <advance-chat :room-id="rowData.conversationId" :advanced-props="{showFooter : false}"/>
      </div>
    </q-dialog>
  </div>
</template>
<script>
import advanceChat from '@imagina/qchat/_components/advancedChat'

export default {
  beforeDestroy() {
  },
  props: {
    row: {default: false},
    permitAction: {default: false},
    fieldActions: {default: false}
  },
  components: {advanceChat},
  watch: {},
  mounted() {
    this.$nextTick(function () {
      this.init()
    })
  },
  data() {
    return {
      configApp: config('app'),
      loading: false,
      showAllInfo: false,
      showConversation: false
    }
  },
  computed: {
    rowData() {
      if (!this.row) return false//Validate row data
      let row = this.$clone(this.row)//Get row data
      let currentUserId = this.$store.state.quserAuth.userId//Get current user ID
      //Get Information fields
      let informationFields = row.fields ? row.fields.filter(field => field.name.indexOf('rating') == -1) : []
      //Get rating fields
      let rating = row.fields ? row.fields.find(field => field.name == 'rating') : false
      let ratingComments = row.fields ? row.fields.find(field => field.name == 'rating_comments') : false

      //Response
      return {
        id: row.id,
        title: this.$trd(row.createdAt),
        mainInformation: [
          {label: `${this.$tr('ui.label.appointment')} N°`, value: row.id},
          {label: this.$tr('ui.form.status'), value: row.status.title}
        ],
        informationFields: informationFields.map(field => {
          return {label: field.name, value: field.value}
        }),
        relatedUsers: [
          //Assigned to
          ((!row.assigned || row.assigned.id == currentUserId) ? false : {
            ...row.assigned,
            rolName: this.$tr('ui.form.assignedTo')
          }),
          //Customer
          ((!row.customer || row.customer.id == currentUserId) ? false : {
            ...row.customer,
            rolName: this.$tr('ui.label.customer')
          })
        ],
        recomendation: (row.options && row.options.recommendation) ? row.options.recommendation : false,
        conversationId: row.conversation ? row.conversation.id : false,
        ratingFields: (!rating && !ratingComments) ? false : {
          rattingComments: ratingComments ? ratingComments.value : false,
          ratingField: !rating ? false : {
            value: parseInt(rating.value),
            type: 'rating',
            props: {
              readonly: true
            }
          }
        },
        goToAppointment: ([5, 6].indexOf(parseInt(row.statusId)) == -1) ? true : false
      }
    }
  },
  methods: {
    init() {

    }
  }
}
</script>
<style lang="stylus">
</style>

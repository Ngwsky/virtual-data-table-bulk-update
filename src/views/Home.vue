<template>
   <v-container fluid>
      <v-row class="px-4">
         <v-col>
            <v-slider
               v-model="slider"
               class="align-center"
               :max="10000"
               :min="0"
               label="表行数"
               hide-details
            >
               <template v-slot:append>
                  <v-text-field
                     v-model="slider"
                     class="mt-0 pt-0"
                     hide-details
                     single-line
                     type="number"
                     style="width: 60px"
                  ></v-text-field>
               </template>
            </v-slider>
         </v-col>
         <v-col>
            <v-slider
               v-model="bench"
               label="Bench"
               class="align-center"
               :max="99"
               :min="0"
               hide-details
            >
               <template v-slot:append>
                  <v-text-field
                     v-model="bench"
                     class="mt-0 pt-0"
                     hide-details
                     single-line
                     type="number"
                     style="width: 60px"
                  ></v-text-field>
               </template>
            </v-slider>
         </v-col>
         <v-col>
            <v-switch v-model="singleSelect" label="Single select"></v-switch>
         </v-col>
         <v-col>
            <v-switch v-model="dense" label="Dense"></v-switch>
         </v-col>
      </v-row>
      <VirtualScrollTable
         v-model="selected"
         :height="480"
         :bench="bench"
         :headers="headers"
         :items="items"
         :single-select="singleSelect"
         show-select
         multi-sort
         :dense="dense"
         :dark="dark"
         :light="light"
      >
         <template v-slot:[`item.chips`]="{ item }">
            <v-chip :color="getGradeColor(item.chips)">{{ item.chips }}</v-chip>
         </template>
         <template v-slot:[`item.price`]="{ value }">
            {{
               value.toLocaleString('ja', {
                  style: 'currency',
                  currency: 'JPY'
               })
            }}
         </template>
         <template v-slot:[`item.date`]="{ value }">
            {{
               getFormatedDate(value)
            }}
         </template>
         <template v-slot:[`item.time`]="{ value }">
            {{
               getFormatedTime(value)
            }}
         </template>
         <template v-slot:[`item.datetime`]="{ value }">
            {{
               getFormatedDateTime(value)
            }}
         </template>
         <template v-slot:[`item.flag`]="{ item }">
            <v-simple-checkbox
               v-model="item.flag"
               disabled
            ></v-simple-checkbox>
        </template>
         <template v-slot:[`item.remarks`]="{ value }">
           <v-tooltip bottom :open-on-click="false" :open-on-focus="false" :open-on-hover="true">
             <template #activator="{ on, attrs }">
               <span class="to-ellipsis" v-bind="attrs" @mouseenter="showElliptedTooltip($event, on.mouseenter)" v-on="{ ...on, ...{ mouseenter: () => true } }">{{ value }}</span>
             </template>
             <span>{{ value }}</span>
           </v-tooltip>
         </template>
         <template v-slot:[`item.actions`]="{ item }">
           <v-icon
             small
             class="mr-2"
             @click="editItem(item)"
           >mdi-pencil</v-icon>
         </template>
      </VirtualScrollTable>
      <v-row class="pa-4">
        <v-col>
          <v-btn
            color="primary"
            :disabled="selected.length < 1"
            @click="editSelectedItems()"
          >
            一括編集
          </v-btn>
        </v-col>
      </v-row>
      <v-dialog
        v-model="editItemDialog"
        persistent
        scrollable
      >
        <v-card>
          <v-card-title class="text-h6">
             編集
          </v-card-title>
          <v-divider></v-divider>
          <v-card-text>
            <v-container>
              <v-row>
                <v-col cols="2">
                  <v-text-field
                    v-model="eiCode"
                    label="コード"
                    counter
                    max-length="8"
                  ></v-text-field>
                </v-col>
                <v-col cols="10">
                  <v-radio-group v-model="eiPayment" row>
                    <template v-slot:label>
                      支払方法
                    </template>
                    <v-radio
                      v-for="n in types"
                      :key="n"
                      :label="n"
                      :value="n"
                    ></v-radio>
                  </v-radio-group>
                </v-col>
                <v-col cols="4">
                  <v-text-field
                    v-model="eiPrice"
                    label="価格"
                    type="number"
                    suffix="円"
                    counter
                    max-length="8"
                  ></v-text-field>
                </v-col>
                <v-col cols="4">
                  <v-menu
                    ref="eiDateMenu"
                    v-model="eiDateMenu"
                    :close-on-content-click="false"
                    transition="scale-transition"
                    offset-y
                    min-width="auto"
                  >
                    <template v-slot:activator="{ on, attrs }">
                      <v-text-field
                        :value="getFormatedDate(new Date(eiDate))"
                        label="日付"
                        prepend-icon="mdi-calendar"
                        readonly
                        v-bind="attrs"
                        v-on="on"
                      ></v-text-field>
                    </template>
                    <v-date-picker
                      v-model="eiDate"
                      no-title
                      @change="() => this.$refs.eiDateMenu.save(eiDate)"
                    ></v-date-picker>
                  </v-menu>
                </v-col>
                <v-col cols="4">
                  <v-text-field
                    v-model="eiTime"
                    label="時刻"
                    type="time"
                  ></v-text-field>
                </v-col>
                <v-col cols="4">
                  <v-checkbox
                    label="フラグ"
                    :input-value="eiFlag"
                    :indeterminate="eiFlag === null"
                    @click="() => eiFlag = (eiFlag ? null : (eiFlag === null ? false : true))"
                  ></v-checkbox>
                </v-col>
                <v-col cols="12">
                  <v-textarea
                    outlined
                    label="備考"
                    v-model="eiRemarks"
　　　　　　        ></v-textarea>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
          <v-divider></v-divider>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="grey lighten-2"
              @click="editItemDialog = false"
            >
              キャンセル
            </v-btn>
            <v-btn
              color="primary"
              @click="editItemDialog = false"
            >
              設定
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
   </v-container>
</template>

<script>
import VirtualScrollTable from '../components/VirtualScrollTable'

export default {
   name: 'Home',
   components: {
      VirtualScrollTable
   },
   data() {
      return {
         slider: 5000,
         bench: 20,
         dense: false,
         dark: false,
         light: false,
         singleSelect: false,
         headers: [
            {
               text: 'ID',
               value: 'id',
               align: 'end',
            },
            {
               text: 'コード',
               value: 'code',
            },
            {
               text: '支払方法',
               value: 'payment'
            },
            {
               text: '価格',
               value: 'price',
               align: 'end',
            },
            {
               text: '日付',
               value: 'date',
               filterTextFromatter: this.getFormatedDate,
            },
            {
               text: '時刻',
               value: 'time',
               filterTextFromatter: this.getFormatedTime,
            },
            {
               text: '日時',
               value: 'datetime',
               filterTextFromatter: this.getFormatedDateTime,
            },
            {
               text: 'フラグ',
               value: 'flag',
               align: 'center',
               filterTextFromatter: (v) => v ? 'ON' : 'OFF',
            },
            {
               text: 'チップス',
               value: 'chips',
               align: 'center',
            },
            {
               text: '備考',
               value: 'remarks',
            },
            {
               text: '',
               value: 'actions',
               sortable: false,
               filterable: false,
            },
         ],
         items: [],
         types: [
            '現金',
            '振込',
            'クレジットカード',
            'クーポン',
            'コンビニ'
         ],
         grades: ['A', 'B', 'C', 'D', 'E'],
         selected: [],
         editItemDialog: false,
         editItemTargets: [],
         eiCode: '',
         eiPayment: '',
         eiPrice: 0,
         eiDate: '',
         eiTime: '',
         eiDateTime: '',
         eiFlag: false,
         eiChips: '',
         eiRemarks: '',
         eiDateMenu: false,
      }
   },
   methods: {
      getFormatedDate: function (dt) {
         return new Intl.DateTimeFormat('ja', { dateStyle: 'short' }).format(dt) + '(' + new Intl.DateTimeFormat('ja', { weekday: 'narrow' }).format(dt) + ')'
      },
      getFormatedTime: function (dt) {
         return new Intl.DateTimeFormat('ja', { timeStyle: 'short' }).format(dt)
      },
      getFormatedDateTime: function (dt) {
         return this.getFormatedDate(dt) + ' ' + this.getFormatedTime(dt)
      },
      refreshItems: function (val) {
         let temp = []
         let b = Date.now()
         let prfmdt = new Date(Date.now() + Math.floor(Math.random() * 31 * 24 * 60 * 60 * 1000))
         prfmdt.setHours(12)
         prfmdt.setMinutes(34)
         prfmdt.setSeconds(56)
         prfmdt.setMilliseconds(789)
         let kana =
            'あいうえおかきくけこさしすせそたちつてとなにぬねのはひふへほまみむめもやゆよらりるれろわゐうゑをんアイウエオカキクケコサシスセソタチツテトナニヌネノハヒフヘホマミムメモヤユヨラリルレロワヰウヱヲンがぎぐげござじずぜぞだぢづでどばびぶべぼヴガギグゲゴザジズゼゾダヂヅデドバビブベボぱぴぷぺぽパピプペポぁぃぅぇぉっゃゅょゎァィゥェォヵヶッャュョー'
         console.log('start generate data ...')
         for (let i = 1; i <= val; i++) {
            const ja = Array(2 + Math.floor(Math.random() * 100))
                  .fill()
                  .map(() => {
                     let s = Math.floor(Math.random() * kana.length)
                     return kana.substring(s, s + 1)
                  })
                  .join('');
            temp.push({
               id: i,
               code: Math.random().toString(36).slice(-8),
               payment: this.types[Math.floor(Math.random() * this.types.length)],
               chips: this.grades[
                  Math.floor(Math.random() * this.grades.length)
               ],
               remarks: ja,
               date: new Date(prfmdt.getFullYear(), prfmdt.getMonth(), prfmdt.getDate(), 0, 0, 0, 0),
               time: new Date(1970, 0, 1, prfmdt.getHours(), prfmdt.getMinutes(), 0, 0),
               datetime: new Date(prfmdt),
               price: Math.floor(Math.random() * 999 + 1) * 100,
               flag: 0.5 < Math.random(),
               actions: '',
            })
            prfmdt = new Date(prfmdt.getTime() + Math.random() * 24 * 60 * 60 * 1000)
         }
         console.log('complete', (Date.now() - b) / 1000.0, 'sec')
         this.items = temp
      },
      getGradeColor(grade) {
         let c0 = [102, 187, 106]
         let c1 = [255, 238, 88]
         let c2 = [239, 83, 80]
         let i = this.grades.findIndex((g) => g === grade)
         let p = i / (this.grades.length - 1)

         if (p < 0.5)
            return (
               'rgb(' +
               Math.floor((c1[0] - c0[0]) * p + c0[0]) +
               ',' +
               Math.floor((c1[1] - c0[1]) * p + c0[1]) +
               ',' +
               Math.floor((c1[2] - c0[2]) * p + c0[2]) +
               ')'
            )
         else
            return (
               'rgb(' +
               Math.floor((c2[0] - c1[0]) * p + c1[0]) +
               ',' +
               Math.floor((c2[1] - c1[1]) * p + c1[1]) +
               ',' +
               Math.floor((c2[2] - c1[2]) * p + c1[2]) +
               ')'
            )
      },
      isEllipted(element) {
        if (!(element instanceof HTMLElement)) return null;
        return element.offsetWidth < element.scrollWidth;
      },
      showElliptedTooltip(e, f) {
        if (!e || !f) return;
        if (this.isEllipted(e.target)) f(e);
      },
      editItem(item) {
         this.editItemTargets = [item];
         this.eiCode = item.code;
         this.eiPayment = item.payment;
         this.eiPrice = item.price;
         this.eiDate = item.date.toISOString();
         this.eiTime = this.getFormatedTime(item.time);
         this.eiFlag = item.flag;
         this.eiRemarks = item.remarks;
         this.editItemDialog = true;
      },
      editSelectedItems() {
         this.editItemTargets = this.selected;
         this.eiCode = '';
         this.eiPayment = '';
         this.eiPrice = '';
         this.eiDate = '';
         this.eiTime = '';
         this.eiFlag = false;
         this.eiRemarks = '';
         this.editItemDialog = true;
      }
   },
   created() {
      this.refreshItems(this.slider)
   },
   watch: {
      slider: function (val) {
         this.refreshItems(val)
      },
      selected: function (val) {
         console.log('selected', val)
      }
   }
}
</script>

<style scoped>
.to-ellipsis {
  display: inline-block;
  max-width: 320px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
</style>
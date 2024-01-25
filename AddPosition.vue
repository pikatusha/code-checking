<template>
  <v-dialog
      content-class="dialog-simple"
      @click:outside="closse"
      :value="dialogPosition"
      :manuallyDetails = manuallyDetails
      overlay-color="#1B2A61"
      persistent
      max-width="485px"
  >
    <v-card>
      <v-card-text class="dialog-simple__text">
        <div id="addtkp-popup" class="popup-block addtkp" style="display: block;">
          <div class="block">
            <template v-if="sortType === 'addPosition'">
              <div class="header_modal">
                <div class="title">Добавление позиции в ТКП</div>
                <div class="close" @click="closse">
                  <ui_svg icon="popup-close" class="close"/>
                </div>
              </div>

              <v-form ref="form" class="form" @submit.prevent="confirm">
                <div class="line">
                  <ui_input :rules="rules.required" required v-model="create.external_id" label="Номер детали"/>
                </div>

                <div class="line">
                  <ui_input :rules="rules.required" required v-model="create.name_ru" label="Наименование"/>
                </div>

                <div class="specifications">
                  <div class="line_specifications">
                    <ui_input :rules="rules.required" :required="true" type="label-top" label="Ед. измерения:"
                               v-model="create.unit"/>
                  </div>

                  <div class="line_specifications">
                    <ui_number_range label="Количество" v-model="create.count_in_tcp"/>
                  </div>

                  <div class="line_specifications">
                    <ui_input :rules="rules.required" required type="number" v-model="create.price" label="Цена, евро"/>
                  </div>
                </div>

                <div class="line btn-wrapp">
                  <ui_btn class="btn_confirm" type="main" @click="confirmAdd">Добавить</ui_btn>
                </div>
              </v-form>
            </template>

            <template v-if="sortType === 'changePrice'">
              <div>
                <div class="header_modal">
                  <div class="title">Изменение цены</div>
                  <div class="close" @click="closse">
                    <ui_svg icon="popup-close" class="close"/>
                  </div>
                </div>

                <v-form ref="form" class="form" @submit.prevent="confirm">
                  <div class="line">
                    <ui_input :rules="rules.required" required v-model="create.price" label="Процент изменения, %"
                              hint="Введите число в процентах. Например,  «107» — все цены в сервисе
                                            увеличатся на 7% от текущей цены."/>
                  </div>

                  <div class="line btn-wrapp">
                    <ui_btn class="btn_confirm" type="main" @click="changePriceConfirm()">Изменить</ui_btn>
                  </div>

                  <DialogSimple
                      :buttonConfirm="change.buttonConfirm"
                      :buttonCancel="change.buttonCancel"
                      :dialog="change.isShow"
                      :title="change.title"
                      :text="change.text"
                      @cancel="change.isShow = false"
                      @confirm="confirmChange"
                  />

                </v-form>
              </div>
            </template>
          </div>
        </div>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script>
import DialogSimple from "./DialogSimple.vue";
import {mapGetters} from "vuex";
import api from "@/api";

export default {

  name: 'AddPosition',
  data: () => ({
    create: {
      name_ru: '',
      external_id: '',
      price: 0,
      unit: 'шт',
      count_in_tcp: 1
    },
    dialog: {
      isShow: false,
    },
    change: {
      title: 'Внимание все цены будут увеличены на 7%',
      text: `Данное действие необратимо, продолжить?`,
      buttonConfirm: 'Продолжить',
      buttonCancel: 'Отмена',
      isShow: false,
    },
    manuallyDetails: []
  }),

  props: {
    dialogPosition: {
      required: true
    },
    sortType: {
      default: 'main'
    },
  },

  components: {
    DialogSimple
  },
  computed: {
    ...mapGetters(['rules', 'getUser']),
  },

  methods: {
    closse() {
      this.$emit('closse')
    },
    confirm() {
      if (!this.$refs.form.validate()) return
      this.$emit('confirm')
    },
    async confirmAdd() {
      let i = 1
      const formData = {
        id: this.$route.params.id,
        number: i++,
        external_id: this.create.external_id,
        position: "-",
        name_ru: this.create.name_ru,
        unit: this.create.unit,
        count_in_model: 1,
        count_in_tcp: this.create.count_in_tcp,
        price: this.create.price,
        sum:this.create.count_in_tcp * this.create.price,
      }

      try {
        if (!this.$refs.form.validate()) return
        await api.tcpRepository.CreateTcpManually.call(this, {data: {details: [formData]}, id: this.$route.params.id})
        this.detailsManually = formData
        this.$emit('updateDetails', this.detailsManually)
      } catch (e) {
        console.log(e);
      }

    },
    confirmChange() {
      if (!this.$refs.form.validate()) return
      this.$emit('confirm')
    },
    changePriceConfirm() {
      this.$emit('closse')
      this.change.isShow = true
    },
  },
}
</script>

<style lang="scss" scoped>
.line_specifications {
  margin: 0px 10px;
}

.select-contain .select-label[data-v-60e1668b] {
  margin-right: 0em !important;
}

.header_modal {
  display: flex;
  justify-content: space-between;
}

.close {
  margin-top: 15px;
  cursor: pointer;
}

.popup-block .title {
  font-weight: 500;
  font-size: 21px;
  line-height: 126%;
  text-align: center;
  color: #222222;
  margin: 30px 0 30px 0;
}

.popup-block.addtkp .label {
  margin: 0 0 16px 0;
}

.popup-block.addtkp label {
  display: block;
  font-size: 14px;
  line-height: 148%;
  color: #8992B5;
  margin: 0 0 4px 0;
}

.popup-block.addtkp .label span {
  display: block;
  font-size: 12px;
  line-height: 148%;
  color: #8992B5;
  opacity: 0.7;
  margin: 6px 0 0 0;
}

.popup-block.addtkp .flex {
  display: flex;
  justify-content: space-between;
}

.popup-block.addtkp .error-info {
  font-size: 14px;
  line-height: 148%;
  text-align: center;
  color: #E14523;
  margin: 8px 0 0 0;
  height: 21px;
}

.popup-block.addtkp .flex .label:nth-child(1) {
  width: 80px;
}

.popup-block.addtkp .flex .label:nth-child(2) {
  width: 132px;
}

.popup-block.addtkp .flex .label:nth-child(3) {
  width: 132px;
}

.line {
  margin: 25px 0;
}

.v-form .line:not(:first-child) {
  margin-top: 1em;
}

span.input-label {
  display: flex;
  margin-bottom: 4px;
}

.price {
  width: 100px;
}

.btn-wrapp {
  text-align: -webkit-center;
}

.btn_confirm {
  min-width: 250px !important;
}

.specifications {
  display: flex;
  justify-content: space-between;
}
</style>
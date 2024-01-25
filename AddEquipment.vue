<template>
  <v-dialog
      content-class="dialog-simple"
      @click:outside="closse"
      :value="dialogEquipment"
      overlay-color="#1B2A61"
      persistent
      max-width="475px"
  >
    <v-card>
      <v-card-text class="dialog-simple__text">
        <div id="addtkp-popup" class="popup-block addtkp" style="display: block;">
          <div class="block">
            <template v-if="sortType === 'addEquipment'">
              <div class="header_modal">
                <div class="title">Добавление оборудования в ТКП</div>
                <div class="close" @click="closse">
                  <ui_svg icon="popup-close" class="close"/>
                </div>
              </div>

              <v-form ref="form" class="form" @submit.prevent="confirm">
                <div class="line">
                  <label>Оборудование:</label>
                  <ui_select @change="objectSort" :rules="rules.required" required :items="items" :value="perPage"/>
                </div>

                <div class="line btn-wrapp">
                  <ui_btn class="btn_confirm" type="main" @click="confirm">Добавить</ui_btn>
                </div>
              </v-form>
            </template>
          </div>
        </div>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script>
import {mapGetters} from "vuex";
import api from "@/api";

export default {

  name: 'AddEquipment',
  data: () => ({
    create: {
      name: '',
      numberDetail: '',
      price: 0,
      unit: 'kg',
      col: 1
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
    name: 'Оборудование',
    props: 'equipment',
    items:[],
    perPage: {
      default: 'Выбрать оборудование'
    },
  }),

  props: {
    dialogEquipment: {
      required: true
    },
    sortType: {
      default: 'main'
    },
  },
  computed: {
    ...mapGetters(['rules']),
  },

  mounted() {
    this.getModels()
  },

  methods: {
    closse() {
      this.$emit('closse')
    },
    confirm() {
      if (!this.$refs.form.validate()) return
      this.$emit('confirm')
    },
    objectSort(e) {
      if (e === this.items) return
      this.$emit('objectSort', e)
    },
    async getModels() {
      const data = await api.modelRepository.GetModel.call(this)
      this.items = data.result.map(item => item.name)
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
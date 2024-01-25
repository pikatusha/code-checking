<template>
  <div>

    <DialogSimple
        :buttonCancel="dialog.buttonCancel"
        :buttonConfirm="dialog.buttonConfirm"
        :dialog="dialog.isShow"
        :text="dialog.text"
        :title="dialog.title"
        @cancel="dialog.isShow = false"
        @confirm="confirmDelete"
    />

    <div class="title-wrapper">

      <template v-if="isLoad">
        <div class="loader">
          <div class="loader__item">
            <SkeletLoader type="text@1"/>
          </div>
        </div>
      </template>

      <template v-else>
        <div class="title-wrapper__text">
          <div></div>

          <div class="title-text">
            Редактирование оборудования
            <p class="sub-title">
              <span>{{ edit.factory_number }}</span><span class="dots">·</span><span>{{ edit.location }}</span>
            </p>
          </div>

          <div style="display: flex; justify-content: flex-end;">
            <ui_btn type="secondary" @click="toArchive()">
              Поместить в архив
            </ui_btn>
          </div>

        </div>
      </template>
    </div>


    <div class="page-edit-tabs" style="display: flex; justify-content: center;">
      <DashTabs v-model="selectedTab" :items="itemsTabs" @change="changeTabs"/>
    </div>

    <template>

      <template v-if="selectedTab === 'basic_info'">

        <template v-if="isLoad">

          <div class="form-wrapper">
            <div class="inputs-wrapper">
              <div v-for="item in 6" :key="item+'load'" class="line">
                <SkeletLoader type="list-item"/>
              </div>
            </div>
          </div>

        </template>

        <v-form v-else ref="form" class="form-wrapper">

          <div class="inputs-wrapper">

            <div class="line">
              <ui_input v-model="edit.name" :rules="rules.required" label="Название оборудования" required/>
            </div>

            <div class="line">
              <ui_file accept=".png" v-model="edit.icon" id="fileIcon" label="Иконка оборудования (.png)"/>
            </div>

            <div class="line">
              <ui_file accept=".png" v-model="edit.scheme" :rules="rules.required" required
                       label="Чертеж для скачивания (.png)"/>
            </div>

            <div class="line">
              <ui_file accept=".pdf" v-model="edit.passport" id="filePassport" label="Паспорт оборудования (.pdf)"/>
            </div>

          </div>

        </v-form>

      </template>

      <template v-if="selectedTab === 'details'">
        <SearchInput v-model="search" @click="searchData"/>
        <div style="display: flex; justify-content: flex-end; margin-bottom: 1em">
          <ui_btn type="main" @click="addRowToTable()">
            Добавить деталь
          </ui_btn>
        </div>
        <TableMain
            ref="table"
            :colums="colums"
            :isEditable="true"
            :empty="{
              text:'Клиент еще не добавил детали',
              button: 'Добавить деталь'
            }"
            :isLoad="isLoad"
            :page="page"
            :perPage="perPage"
            :rows="edit.details"
            :sort="sort"
            :sortItems="sortItems"
            :sortType="'new_tkp'"
            :total="total"
            @createItem="createItemTable"
            @deleteRow="deleteDetail"
            @addRowToTable="addRowToTable"
            @changePage="changePage"
            @changePerPage="changePerPage"
            @changeSort="changeSort"
            @saveRowChange="saveRowChange"
        />
      </template>
    </template>

  </div>


</template>

<script>

import DashTabs from '@/components/DashTabs.vue'
import DialogSimple from '@/components/dialogs/DialogSimple.vue';
import SkeletLoader from '@/components/SkeletLoader.vue';
import TableMain from "@/components/TableMain";
import SearchInput from '@/components/SearchInput.vue'
import api from '@/api'
import {mapGetters} from "vuex";
import helper from '@/plugins/helper'

export default {
  name: 'EditModel',
  data: () => ({
    selectedTab: 'basic_info',
    isLoad: true,
    dialog: {
      isShow: false,
      type: '',
      text: ''
    },
    edit: {
      name: '',
      scheme: null,
      passport: null,
      icon: null,
      details: [],
      total_details: []
    },
    total: 1,
    page: 1,
    perPage: 10,
    itemsTabs: [
      {
        name: 'Основная информация',
        slug: 'basic_info',
      },
      {
        name: 'Детали',
        slug: 'details',

      }
    ],
    search: '',
    sort: null,
    sortItems: [
      {
        id: null,
        label: 'По умолчанию'
      },
      {
        id: 'external_id',
        label: 'id'
      },
      {
        id: 'position',
        label: 'Позиция'
      },
      {
        label: 'Название (Англ)',
        id: 'name_en'
      },
      {
        label: 'Название (Рус)',
        id: 'name_ru'
      },
      {
        label: 'Кол-во',
        id: 'count'
      },
      {
        label: 'Период. осмотра',
        id: 'inspection_period_id'
      },
      // {
      //   label: 'Цена',
      //   id: 'price'
      // },
    ],
    colums: [
      {
        name: 'id',
        props: 'external_id',
        edtType: 'input',
        required: true
      },
      {
        name: 'Позиция',
        props: 'position',
        edtType: 'input',
        required: true
      },
      {
        name: 'Название (Англ)',
        props: 'name_en',
        edtType: 'input',
        required: true
      },
      {
        name: 'Название (Рус)',
        props: 'name_ru',
        edtType: 'input',
        required: true
      },
      {
        name: 'Кол-во',
        props: 'count_in_model',
        edtType: 'input',
        required: true
      },
      {
        name: 'Период. осмотра',
        props: 'inspection_period_id',
        edtType: 'select',
        required: true,
        items: [
          {
            label: '6 мес',
            id: 1
          }, {
            label: '1 год',
            id: 2
          }, {
            label: '2 года',
            id: 3
          }, {
            label: '3 года',
            id: 4
          }, {
            label: '4 года',
            id: 5
          }, {
            label: '5 лет',
            id: 6
          },
        ],
      },
      {
        name: 'Действия',
        props: 'action',
        buttons: ['edit', 'trash']
      },
    ],
  }),


  components: {
    DashTabs,
    SkeletLoader,
    DialogSimple,
    TableMain,
    SearchInput
  },

  computed: {
    ...mapGetters(['rules', 'getStore']),
  },

  mounted() {
    this.loadModel()
  },

  methods: {
    async createDetail(el, index) {
      try {
        const data = await api.modelRepository.CreateOneDetail.call(this, {
          data: {details: [el]},
          id: this.$route.params.id
        })
        this.edit.details[index].id = data.result.id
      } catch (er) {
        console.log(er);
      }
    },

    async updateDetail(el) {
      try {
        const data = await api.modelRepository.UpdateOneDetail.call(this, el)
        return data
      } catch (er) {
        console.log(er);
      }
    },

    saveRowChange(index, el) {
      if (el.id) {
        this.updateDetail(el)
      } else {
        this.createDetail(el, index)
      }
    },

    addRowToTable() {
      this.edit.details.unshift({
        id: null,
        external_id: '',
        position: '',
        name_ru: '',
        name_en: '',
        inspection_period_id: 1,
        isEdit: true
      },)
    },

    searchData() {
      this.page = 1
      this.loadModel('details')
    },

    createItemTable() {
      this.edit.details.unshift({
        id: null,
        external_id: '',
        position: '',
        name_ru: '',
        name_en: '',
        inspection_period_id: 1,
        isEdit: true
      },)
    },

    changeSort(e) {
      this.sort = e
      this.loadModel('details')
    },

    changePerPage(e) {
      this.page = 1
      this.perPage = e
      this.loadModel('details')
    },

    changePage(e) {
      this.page = e
      this.loadModel('details')
    },

    onBack() {
      this.$router.replace('/model/' + this.$route.params.id)
    },

    async toArchive() {
      this.isLoad = true
      try {
        await api.modelRepository.DeleteModel.call(this, this.$route.params.id)
        this.$router.push('/library')
      } catch (e) {
        console.log(e);
      } finally {
        this.isLoad = false
      }
    },

    changeTabs(val) {
      this.selectedTab = val
    },


    async loadModel(str = 'any') {
      this.isLoad = true

      const query = {
        per_page: this.perPage,
        page: this.page,
        sort: this.sort
      }
      try {
        const data = await api.modelRepository.GetModelByIdPagination.call(this, query, this.$route.params.id)
        this.edit.total_details = data.result.total_details
        this.total = Math.ceil(Number(this.edit.total_details) / Number(this.perPage))

        if (str !== 'details') {
          let name = `${this.getStore}${data.result['scheme']}`
              .split('/')[`${this.getStore}${data.result['scheme']}`.split('/').length - 1]

          let namePassport = `${this.getStore}${data.result['passport']}`
              .split('/')[`${this.getStore}${data.result['passport']}`.split('/').length - 1]

          let nameIcon = `${this.getStore}${data.result['icon']}`
              .split('/')[`${this.getStore}${data.result['icon']}`.split('/').length - 1]

          this.edit = data.result

          fetch(`${this.getStore}${data.result['scheme']}`)
              .then(response => response.blob())
              .then(blob => {
                blob.name = name
                this.edit['scheme'] = blob
              })

          fetch(`${this.getStore}${data.result['passport']}`)
              .then(response => response.blob())
              .then(blob => {
                blob.name = namePassport
                this.edit['passport'] = blob
              })

          fetch(`${this.getStore}${data.result['icon']}`)
              .then(response => response.blob())
              .then(blob => {
                blob.name = nameIcon
                this.edit['icon'] = blob
              })
        }

        this.edit.details = data.result.details.map(item => ({
          ...item,
          isEdit: false,
          inspection_period_id: parseInt(helper.getInspectionId(item.inspection_period))
        }))


      } catch (e) {
        console.log(e);
      } finally {
        this.isLoad = false
      }

    },


    confirmDelete() {
      this.edit.details.splice(this.dialog.index, 1)
      this.dialog.isShow = false
      this.edit.details.forEach(item => item.isEdit = false)
      this.$refs.table.isEditedRow = false
      this.deleteItem(this.dialog.id)
    },

    async deleteItem(id) {
      this.isLoad = true
      try {
        await api.modelRepository.DeleteOneDetail.call(this, id)
        this.loadModel('details')
      } catch (e) {
        console.log(e);
      } finally {
        this.isLoad = false
      }
    },

    deleteDetail(str, id) {
      this.dialog = {
        isShow: true,
        title: 'Удалить деталь?',
        text: `Деталь будет удалена`,
        buttonCancel: 'Не удалять',
        buttonConfirm: 'Удалить',
        index: str,
        type: 'contact',
        id: id
      };
    },

    async saveItem() {
      if (this.isLoad) return
      try {
        let id = await this.saveModel()
        this.$router.push('/model/' + id)
      } catch (e) {
        console.log(e);
      } finally {
        this.isLoad = false
      }

    },

    async saveModel() {
      let formData = new FormData();

      for (let key in this.edit) {
        if (this.edit[key] !== null) {
          formData.append(key, this.edit[key]);
        }
      }

      try {
        await api.modelRepository.UpdateModel.call(this, this.edit.id, formData)
        return this.edit.id

      } catch (er) {
        console.log(er);
      }
    },
  }

}
</script>

<style lang="scss" scoped>
.title-wrapper {
  .loader {
    display: flex;
    justify-content: center;

    &__item {
      width: 40%;
    }
  }

  .title-text {
    font-weight: 500;
    font-size: 32px !important;
    line-height: 126%;
    color: #000000;
    text-align: center;
    margin: 0 0 8px 0;
  }

  &__text {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
  }

  .sub-title {
    font-family: 'Roboto';
    font-style: normal;
    font-weight: 400;
    font-size: 14px;
    line-height: 124%;
    color: #8992B5;
    text-transform: uppercase;
  }

  .dots {
    margin: 0 1em;
  }
}

.wrapper-info {
  max-width: 575px;
  width: 100%;
  display: block;
  margin: 0 auto;

  .child:not(:first-child) {
    margin-top: 16px;
  }

  .child {
    grid-template-columns: 0fr 12fr;
    grid-gap: 0px 20px;
    display: grid;

    .line:not(:first-child) {
      margin-top: 1em;
    }

    .inputs-wrapper {
      border-left: solid #AAB7CB 1px;
      padding: 0 20px;
    }
  }
}

.line__button {
  margin-top: 16px;
  display: flex;
  justify-content: center;
}

.form-wrapper {
  margin-top: 16px;
  display: flex;
  justify-content: center;

  .inputs-wrapper {
    width: 500px;

    .line:not(:first-child) {
      margin-top: 16px;
    }

    .inline {
      display: flex;
      justify-content: space-between;
    }
  }
}

</style>
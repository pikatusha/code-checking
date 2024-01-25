<template>
  <div>

    <DialogSimple
        :buttonConfirm="dialog.buttonConfirm"
        :buttonCancel="dialog.buttonCancel"
        :dialog="dialog.isShow"
        :title="dialog.title"
        :text="dialog.text"
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
            Редактирование клиента
            <p class="sub-title">{{ edit.name }}</p>
          </div>

          <div style="display: flex; justify-content: flex-end;">
            <ui_btn @click="toArchive()" type="secondary">
              Поместить в архив
            </ui_btn>
          </div>

        </div>
      </template>
    </div>


    <div class="page-edit-tabs" style="display: flex; justify-content: center;">
      <DashTabs @change="changeTabs" v-model="selectedTab" :items="itemsTabs"/>
    </div>

    <template>

      <template v-if="selectedTab === 'basic_info'">

        <template v-if="isLoad">

          <div class="form-wrapper">
            <div class="inputs-wrapper">
              <div class="line" v-for="item in 6" :key="item+'load'">
                <SkeletLoader type="list-item"/>
              </div>
            </div>
          </div>

        </template>

        <v-form v-else class="form-wrapper" ref="form">

          <div class="inputs-wrapper">

            <div class="line">
              <ui_input v-model="edit.short_name" label="Удобное название компании" hint="Пример: Кастомону" required
                        :rules="rules.required"/>
            </div>

            <div class="line">
              <ui_input v-model="edit.name" label="Полное название компании"
                        hint="Пример: ООО КАСТАМОНУ ИНТЕГРЕЭЙТЕД ВУД ИНДАСТРИ (г Алабуга)"/>
            </div>

            <div class="line">
              <ui_file v-model="edit.logo"/>
            </div>

          </div>

        </v-form>

      </template>

      <template v-if="selectedTab === 'account_mechanic'">

        <template v-if="isLoad">

          <div class="form-wrapper">
            <div class="inputs-wrapper">
              <div class="line" v-for="item in 6" :key="item+'load'">
                <SkeletLoader type="list-item"/>
              </div>
            </div>
          </div>

        </template>

        <v-form v-else class="form-wrapper" ref="form">

          <div class="inputs-wrapper">

            <div class="line">
              <ui_input class="line" v-model="edit.user.position"
                        hint="Например, Механик, Инженер, Директор предприятия" label="Должность" required
                        :rules="rules.required"/>
            </div>

            <div class="inline line">
              <ui_input v-model="edit.user.firstname" label="Имя" required :rules="rules.required"/>
              <ui_input v-model="edit.user.lastname" label="Фамилия" required :rules="rules.required"/>
            </div>

            <div class="line">
              <ui_input class="line" label="Отчество" v-model="edit.user.patronymic"/>
            </div>

            <div class="line">
              <ui_input class="line" v-model="edit.user.email" label="E-mail" required :rules="rules.email"/>
            </div>

            <div class="line">
              <ui_input class="line" v-model="edit.user.phone_number" label="Номер телефона" type="number"
                        :rules="rules.phone_number"/>
            </div>

            <div class="line">
              <ui_input type="password" v-model="edit.user.password" label="Пароль"/>
            </div>

          </div>

        </v-form>

      </template>

      <template v-if="selectedTab === 'contacts'">
        <div class="wrapper-info">
          <div v-for="(info, index) in edit.contacts" :key="'info-'+index" class="child">
            <div>
              <ui_btn @click="deleteInfo(index)" icon="icon-trash" type="secondary-icon"/>
            </div>
            <div class="inputs-wrapper">
              <div class="line">
                <ui_input v-model="info.description" label="Информация о контакте"
                          hint='Примеры: “Отдел снабжения” или “Директор по поставкам” или “Иван Иванович”'/>
              </div>
              <div class="line">
                <ui_input v-model="info.phone_number" label="Номер телефона" type="number"
                          :rules="rules.phone_number"/>
              </div>
              <div class="line">
                <ui_input v-model="info.email" label="E-mail"/>
              </div>
            </div>
          </div>
        </div>
        <div class="line line__button">
          <ui_btn type="main" icon="icon-plus" submit="btn" @click="addAccount">Добавить контакт</ui_btn>
        </div>
      </template>

      <template v-if="selectedTab === 'enterprises'">
        <EnterpriseList @delete-enterprise="deleteInfo" :page="page" :hasDelete="true" :enterprises="edit.enterprises"
                        @changePage="loadEnterprises" :total="total" :isLoad="isLoad"/>
      </template>

    </template>

  </div>


</template>

<script>

import DashTabs from '@/components/DashTabs.vue'
import DialogSimple from '@/components/dialogs/DialogSimple.vue';
import SkeletLoader from '@/components/SkeletLoader.vue';
import EnterpriseList from '@/components/enterprise/EnterpriseList.vue'
import api from '@/api'
import {mapGetters} from "vuex";

export default {
  name: 'EditCompany',
  data: () => ({
    selectedTab: 'basic_info',
    isLoad: true,
    dialog: {
      isShow: false,
      type: '',
      text: ''
    },
    edit: {
      logo: null,
      name: '',
      short_name: '',
      user: {
        password: '',
        position: '',
        role_id: 2,
        firstname: '',
        lastname: '',
        patronymic: '',
        email: '',
        phone_number: ''
      },
      contacts: [],
      enterprises: [],
    },
    total: 1,
    page: 1,
    perPage: 9,
    itemsTabs: [
      {
        name: 'Основная информация',
        slug: 'basic_info',
      },
      {
        name: 'Аккаунт главного механика',
        slug: 'account_mechanic',

      },
      {
        name: 'Контакты клиента',
        slug: 'contacts'
      },
      {
        name: 'Подразделения',
        slug: 'enterprises'
      }
    ],
  }),


  components: {
    DashTabs,
    SkeletLoader,
    DialogSimple,
    EnterpriseList
  },

  computed: {
    ...mapGetters(['rules', 'getStore']),
  },

  mounted() {
    this.loadCompany()
    this.loadEnterprises()
  },

  methods: {
    onBack() {
      this.$router.replace('/company/' + this.$route.params.id)
    },

    async toArchive() {
      this.isLoad = true
      try {
        await api.companyRepository.DeleteCompany.call(this, this.$route.params.id)
        this.$router.push('/dashboard')
      } catch (e) {
        console.log(e);
      } finally {
        this.isLoad = false
      }
    },

    changeTabs(val) {
      // if(this.isLoad) return

      // if(val === 'basic_info'){
      //     this.loadCompany()
      // }else if(val === 'account_mechanic'){
      //     this.loadUser()
      // }else if(val === 'contacts'){
      //     this.loadContacts()
      // }else if(val === 'enterprises'){
      //     this.loadEnterprises()
      // }

      this.selectedTab = val

    },

    async loadEnterprises() {
      const query = {
        company_id: this.$route.params.id,
        page: this.page
      }

      this.isLoad = true

      try {
        const data = await api.enterprisesRepository.LoadEnterprise.call(this, query)
        if(data.result) {
          this.edit.enterprises = data.result
        }
        
        this.total = Math.ceil(Number(data.total) / Number(this.perPage)) || 1
      } catch (e) {
        console.log(e);
      } finally {
        this.isLoad = false
      }

    },

    async loadCompany() {
      this.isLoad = true

      try {
        const data = await api.companyRepository.getById.call(this, this.$route.params.id)

        let name = `${this.getStore}${data.result['logo']}`.split('/')[`${this.getStore}${data.result['logo']}`.split('/').length - 1]
        fetch(`${this.getStore}${data.result['logo']}`)
            .then(response => response.blob())
            .then(blob => {
              blob.name = name
              this.edit['logo'] = blob
            })
        this.edit['id'] = data.result['id']
        this.edit['name'] = data.result['name']
        this.edit['short_name'] = data.result['short_name']
        if (data.result['contacts']) this.edit['contacts'] = data.result['contacts']
        if (data.result['responsible_user']) this.edit['user'] = data.result['responsible_user']
      } catch (e) {
        console.log(e);
      } finally {
        this.isLoad = false
      }

    },

    confirmDelete(index) {
      if (this.dialog.type === 'contact') {
        this.edit.contacts.splice(index, 1)
      }
      if (this.dialog.type === 'enterprises') {
        this.edit.enterprises.splice(index, 1)
      }
      this.dialog.isShow = false
    },

    deleteInfo(str) {
      if (this.selectedTab === 'contacts') {
        this.dialog = {
          isShow: true,
          title: 'Удалить контакт?',
          text: `Контакт ${this.edit.contacts[str] && this.edit.contacts[str].description ? this.edit.contacts[str].description : ''} будет удален`,
          buttonCancel: 'Не удалять',
          buttonConfirm: 'Удалить',
          index: str,
          type: 'contact'
        };
      } else if (this.selectedTab === 'enterprises') {
        let index = this.edit.enterprises.indexOf(str)

        this.dialog = {
          isShow: true,
          title: 'Удалить подразделения?',
          text: `Подразделения ${this.edit.enterprises[index].name} будет удален`,
          buttonCancel: 'Не удалять',
          buttonConfirm: 'Удалить',
          index: index,
          type: 'enterprises'
        };
      }

    },

    async saveItem() {
      if (this.isLoad) return

      try {
        const result = await this.saveUser()
        if(result.message == "updated"){
          await this.saveCompany(this.edit.user.id)
          await this.saveContact(this.edit.user.id)
        }
      } catch (e) {
        this.isLoad = false
        console.log(e);
      } finally {
        this.isLoad = false
      }

    },

    async saveContact(){
      let data = this.edit.contacts.map(x => ({
        ...x,
        company_id: this.$route.params.id,
      }))

      this.isLoad = true
      try {
        const res = await api.contactRepository.UpdateCompanyContact.call(this, data)
        return res
      } catch (e) {
        console.log(e);
      }

    },

    async saveUser() {
      if (!this.edit.user.password) {
        delete this.edit.user.password
      }
      this.edit.user.phone_number = this.edit.user.phone_number.toString()
      this.isLoad = true
      try {
        const data = await api.userRepository.UpdateUser.call(this, this.edit.user)
         return data
      } catch (e) {
        console.log(e);
      } finally {
        this.edit.user['password'] = ''
      }

    },

    async saveCompany(id) {
      let formData = new FormData();

      formData.append("logo", this.edit.logo)
      formData.append("name", this.edit.name);
      formData.append("id", this.edit.id);
      formData.append("short_name", this.edit.short_name);
      formData.append("manager_id", id);

      try {
        const data = await api.companyRepository.UpdateCompanies.call(this, this.edit.id, formData)
        return data
      } catch (er) {
        console.log(er);
      }
    },

    addAccount() {
      this.edit.contacts.push({
        phone_number: '',
        email: ''
      })
    },
  }

}
</script>

<style lang="scss" scoped>
/* Chrome, Safari, Edge, Opera */
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Firefox */
input[type=number] {
  -moz-appearance: textfield;
}

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
    font-size: 14px;
    line-height: 124%;
    text-align: center;
    color: #000000;
    text-align: -webkit-center;
    margin: 8px 0 32px 0px;
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
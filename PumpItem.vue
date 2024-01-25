<template>
  <div class="pump-item">
    <div class="header-wrapper">

      <div class="name">{{ pump.name }}</div>
      <div class="check-wrapper" v-if="this.getUser.role_id !== 3 && this.getUser.role_id !== 4">
        <span
            :class="!was_discussed ? 'error-text text-check' : 'text-check'">{{ was_discussed ? 'Обсуждено с клиентом' : 'Не обсуждено с клиентом' }}</span>
        <div :class="was_discussed ? 'confirm-wrapp' : 'error-wrapp'">
          <ui_check @change="changeStatus($event, index)" v-model="was_discussed"/>
        </div>
      </div>

    </div>
    <div class="wrapper-labels">
      <div class="item">
        <div class="label">
          Дата ТО
        </div>
        <div class="value">
          {{ moment(pump.next_maintenance_date.date).format('DD.MM.YYYY') }}
        </div>
      </div>
      <div class="item">
        <div class="label">
          Место установки
        </div>
        <div class="value">
          {{ pump.location }}
        </div>
      </div>
    </div>
    <div class="wrapper-button">
      <ui_btn @click="printTo(pump.id)" type="secondary">Распечатать ТО</ui_btn>
      <ui_btn type="secondary" @click="$router.push('/pump/'+pump.id)">К оборудованию</ui_btn>
    </div>
  </div>
</template>

<script>

import api from "@/api";
import {mapGetters} from "vuex";

export default {
  name: 'PumpItem',

  data: () => ({
    was_discussed: false
  }),

  props: {

    pump: {
      required: true
    },

    index: {
      required: true
    }

  },

  computed: {
    ...mapGetters(['getUser']),
  },

  methods: {
    async changeStatus(e) {
      const data = {
        id: this.pump.id,
        was_discussed:e
      }
      try {
        await api.pumpRepository.UpdatePumps.call(this, data)
      } catch (er) {
        console.log(er);
      }
    },

    async printTo(id) {
      try {
        var date = this.moment(new Date()).format('DD.MM.YYYY')
        const res = await api.pumpRepository.DownloadFile.call(this, id)
        let blob = new Blob([res], {
          type: 'application/vnd.ms-excel'
        });
        let link = document.createElement("a");
        link.href = window.URL.createObjectURL(blob);
        link.download = "Распечатка статусов ТО " + date + " " + this.pump.name +".xlsx";
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
      } catch (e) {
        console.log(e);
      }
    },

    setPump(id) {
      this.$router.push('/enterprise/' + id)
    }
  },

  mounted() {
    this.was_discussed = this.pump.was_discussed
  }
}
</script>

<style lang="scss" scoped>

.pump-item {
  cursor: pointer;
  background: #FFFFFF;
  border: 1px solid #DFE7EE;
  border-radius: 8px;
  width: 100%;
  margin: 0 2% 16px 0;
  padding: 22px 24px 18px 24px;
  display: flex;
  flex-direction: column;
  position: relative;

  .wrapper-labels {
    display: flex;
    margin-top: 16px;

    .item:last-child {
      margin-left: 57px;
    }
  }

  .wrapper-button {
    margin-top: 40px;

    .button:first-child {
      margin-right: 16px;
    }
  }

  .check-box-main {
    .v-label {
      font-family: 'Roboto';
      font-style: normal;
      font-weight: 400;
      font-size: 14px !important;
      transition: 1s;
    }

    &__activate {
      .v-label {
        color: #1B2A61 !important;
      }
    }

    &__deactivate {
      .v-label {
        color: #E14523 !important;
      }
    }
  }

  .check-box-main:hover {
    .v-label {
      color: #1B2A61 !important;
    }
  }

  .text-check {
    margin-right: 1em;
  }

  .error-text {
    color: #E14523;
  }

  .header-wrapper {
    display: flex;
    justify-content: space-between;
  }

  .check-wrapper {
    display: flex;
    align-items: center;

    .confirm-wrapp {
      padding: 5px;
      background: #D8DEE8;
      border-radius: 8px;
    }

    .error-wrapp {
      padding: 5px;
      background: #FECEC6;
      border-radius: 8px;
    }
  }

  .name {
    font-family: 'Roboto';
    font-style: normal;
    font-weight: 400;
    font-size: 18px;
    line-height: 148%;
    color: #000000;
    align-self: center;
  }

  .label {
    font-family: 'Roboto';
    font-style: normal;
    font-weight: 400;
    font-size: 14px;
    line-height: 124%;
    color: #8992B5;
  }

  .value {
    font-family: 'Roboto';
    font-style: normal;
    font-weight: 400;
    font-size: 14px;
    line-height: 124%;
    color: #000000;
  }

}

</style>
<template>
    <v-dialog
        content-class="dialog-simple"
        @click:outside="closse"
        :value="showInfoModal"
        overlay-color="#1B2A61"
        persistent
        max-width="500px"
        scrollable
    >
        <v-card>
            <v-card-text class="dialog-simple__text">                                  
                <div id="pinnedclients-popup" class="popup-block pinnedclients">
                    <div class="block">
                        <div class="header_modal">
                            <div>
                                <div class="title">Закрепленные клиенты</div>
                                <div class="n">{{setUserInfo.lastname }} {{setUserInfo.firstname}} {{setUserInfo.patronymic}}</div>
                                <div class="companies" v-for="company in companyInfo" :key="company.id">{{company.name}}</div>
                            </div>
                            <div @click="closse()"><ui_svg icon="popup-close" class="close"/></div>
                        </div>
                        <div class="block-scroll"></div>
                        <ui_btn @click="closse()" class="cancel">Закрыть</ui_btn>
                    </div>
                </div>
            </v-card-text>
        </v-card>
    </v-dialog>

    
</template>

<script>


export default {
    name: 'ShowInfo',
    props: {
        showInfoModal:{
            required: true
        },
        setUserInfo:{},
        companyInfo:[]
    },
    methods:{
        closse(){
            this.$emit('closse')
        }  
    },
}
</script>

<style lang="scss" scoped>
    .close{
        cursor: pointer;
    }
    .header_modal{
        display: flex;
        justify-content: space-between;
        padding-top: 20px;
    }
    .pinnedclients .title {
        text-align: left;
        margin: 0 0 6px 0;
        color: black;
    }
    .pinnedclients .n {
        font-size: 14px;
        line-height: 124%;
        color: #8992B5;
        margin: 0 0 24px 0;
    }
    .pinnedclients .block-scroll p {
        margin: 0 0 12px 0;
    }
    .pinnedclients .cancel {
        background: #FFFFFF;
        border-top: 1px #AAB7CB solid;
        padding: 15px 0 0 0;
        margin: 0 -24px;
    }
    .pinnedclients .block-scroll {
        overflow-y: auto;
        height: 309px;
        font-size: 15px;
        line-height: 114%;
        color: #000;
        scroll-margin: 1em 0.5em 1em 1em;
    }
    .pinnedclients .cancel {
        background: #FFFFFF;
        border-top: 1px #AAB7CB solid;
        padding: 15px 0 0 0;
        margin: 0 -24px;
    }
    .pinnedclients .cancel {
        display: table;
        cursor: pointer;
        width: 100%;
        max-width: 289px;
        text-align: center;
        background: #1D58BC;
        border-radius: 8px;
        padding: 9px;
        font-weight: 500;
        font-size: 16px;
        line-height: 154%;
        color: #FFFFFF;
        margin: 0 auto;
        margin-top: 25px;
    }
    .companies{
      font-size: 15px;
      color: black;
      line-height: 2em;
    }
</style>
<template>
  <profilemobile-page v-if="mobile"></profilemobile-page>
  <div class="content__grid-profile" v-else>
    <aside-bar-component></aside-bar-component>
    <chat-component></chat-component>
    <header-component></header-component>
  <section class="profile">
    <div class="profile__content">
      <img class="profile__user--img" :src="imageUrl">
      <div class="profile__card icon-diamond">
        <h1>{{ username }}</h1>
        <h2><img src="@/assets/icons-profile/icon-diamond-ore.png">{{ balance }}</h2>
      </div>
      <div class="profile__btns--payments">
        <a href="#" class="text-btn btn-bg btn-margin btn-display" @click="depositClick"><img class="icon-margin-deposit-withdraw" src="@/assets/icons-profile/icon-deposit.svg"> Пополнить</a>
        <a href="#" class="withdraw text-btn btn-bg btn-display" @click="withdrawClick"><img class="icon-margin-deposit-withdraw" src="@/assets/icons-profile/icon-withdraw.png"> Вывести</a>
      </div>
    </div>
    <div class="payments">
      <div class="payments__content">
        <div class="payments__types">
          <div class="types types-margin types-text">
            <div class="playerandvid">
            <h3>Вид транзакции</h3>
          </div>
            <div class="data1">
            <h3>Дата</h3>
          </div>
            <div class="summa1">
            <h3>Сумма</h3>
          </div>
          </div>
          <div class="types-line"></div>
        </div>
        <div class="hisorypaymond">
        <div class="payments__history">
          <div class="payments__card" v-for="payment in arrayHistory" :key="payment.id">
            <div class="user-info">
              <img src="@/assets/icons-test/person-icon-profile-userinfo.png">
              <div class="user-name user-name__text" :class="{ 'withdraw-color1': payment.name }">
                <h3>{{ payment.name }}</h3>
              </div>
              <div class="user-name1 user-name1__text">
                <h3>{{ payment.name1 }}</h3>
              </div>
            </div>
            <div class="data-info data-info__text">
              <h3>{{ payment.data }}</h3>
            </div>
            <div class="transaction-info transaction-info__text" :class="{ 'withdraw-color': payment.amount < 0, 'deposit-color': payment.amount > 0 }">
              <h3>{{ payment.amount }} АР</h3>
            </div>
          </div>
          </div>
        
      </div>
      </div>
    </div>
  </section>
    <payments-modal v-if="openModal" @notifacetionmoney="NotificationEventListener" @closemodal="openModal = false" :payments="payments"></payments-modal>
    <notiicationwindow-component @notificationremove="NotificationMethod" :notification="notification"></notiicationwindow-component>
  </div>
</template>
<script>
import AsideBarComponent from "@/components/AsidebarComponent.vue";
import axios from 'axios';
import ChatComponent from "@/components/ChatComponent.vue";
import HeaderComponent from "@/components/HeaderComponent.vue";
import PaymentsModal from "@/components/PaymentsModal.vue";
import ProfilemobilePage from "@/pages/adaptive-pages/ProfilemobilePage.vue";
import NotiicationwindowComponent from "@/components/NotiicationwindowComponent.vue";
import { GetCookie } from "@/assets/js/storage/CookieStorage";
import '@/assets/css/PagesStyles/profile.css'
import {GetCurrentMoney} from "@/assets/js/rest/RestMethods";
export default {
  components: {ProfilemobilePage, HeaderComponent, AsideBarComponent, ChatComponent, PaymentsModal, NotiicationwindowComponent },
  data() {
    return {
      username: 'Artemka',
      imageUrl: '',
      notification: false,
      balance: 0,
      mobile: false,
      openModal: false,
      payments: true,
      arrayHistory: [],
    }
  },
  
  emits: ['notificationremove'],
  mounted() {
    this.checkWindowSize()
    this.AddWindowListener()
  },
  beforeUnmount() {
    this.RemoveWindowListener()
  },
  created() {
    axios.post('https://spsystemcore20231122004605.azurewebsites.net/api/Payment/GetPaymentHistory', {
      searchToken: GetCookie('SearchToken'),
      authtoken: GetCookie('AUTHTOKEN')
    })
    .then(response => {
      // Обработка полученных данных
      const paymentHistory = response.data.paymentHistory;
      paymentHistory.forEach(payment => {
        if (payment.amount > 0) {
          this.claimDataDeposit(payment.amount,payment.utcDate); // Добавление депозита в историю
        } else {
          this.claimDataWithdraw(-payment.amount,payment.utcDate); // Добавление вывода в историю
        }
      });
    })
    .catch(error => {
      console.error('Error fetching hisorypaymond:', error);
    });
    this.username = GetCookie('SpUserName')
    GetCurrentMoney(GetCookie('AUTHTOKEN'), GetCookie('SearchToken'))
        .then((response) => {
          this.balance = response.currentMoney
          console.log('balance in profile', this.balance)
        })
        .catch((error) => {
          console.error('error in profile, get money', error)
        })
    this.imageUrl = `https://avatar.spworlds.ru/front/256/${this.username}`
  },
  methods: {
    depositClick() {
      this.openModal = true
      this.payments = true
    },
    withdrawClick() {
      this.openModal = true
      this.payments = false
    },
    NotificationEventListener() {
      this.notification = true
      this.openModal = false
    },
    NotificationMethod() {
      this.notification = false
    },
    formatNumber(number) {
      return number < 10 ? `0${number}` : number;
    },
    getCurrentFormattedDate() {
      const currentDate = new Date();
      return `${this.formatNumber(currentDate.getDate())}.${this.formatNumber(currentDate.getMonth() + 1)}.${currentDate.getFullYear()}, ${this.formatNumber(currentDate.getHours())}:${this.formatNumber(currentDate.getMinutes())}`;
    },
    claimDataDeposit(amount, utcDate) {
  const historyPayments = {
    name1: 'Пополнение',
    data: this.formatTime(utcDate),
    amount: amount
  }

  this.arrayHistory.unshift(historyPayments);
},

claimDataWithdraw(amount, utcDate) {
  const historyPayments = {
    name: 'Вывод',
    data: this.formatTime(utcDate),
    amount: -amount
  }

  this.arrayHistory.unshift(historyPayments);
},

formatTime(utcDate) {
  const dateUTC = new Date(utcDate);
  const userTimeZone = Intl.DateTimeFormat().resolvedOptions().timeZone;
  const currentTime = new Date();
  const diffTime = currentTime - dateUTC;
  const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24));
  const diffHours = Math.floor(diffTime / (1000 * 60 * 60));
  const diffMinuts = Math.floor(diffTime/(1000*60));

  if (diffDays < 7)
  {
    if (diffDays > 0)
    {
      return `${diffDays} ${diffDays === 1 ? 'день' : 'дня'} назад`;
    }
    else if(diffHours >0)
    {
      return `${diffHours} ${diffHours === 1 ? 'час' : 'часа'} назад`;
    }
    else{
      return `${diffMinuts} ${diffMinuts === 1 ? 'минута' : 'минут'} назад`;
    }
  }
  else
  {
    return dateUTC.toLocaleString('ru-RU', { timeZone: userTimeZone });
  }
}

  }
}
</script>

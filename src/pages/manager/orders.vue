<template>
  <fixed-left-column>
    <template v-slot:fixed>
      <div class="block">
        <DateRange v-model="date_period" placeholder="Выберите период" @change="checkData()" /><br />
        <br />
        <Search
          v-model:search-query="searchQuery"
          v-model:type="type"
          :types="types"
          @click="
            search_data[String(type)] = searchQuery;
            checkData();
          "
        />
      </div>
      <br />
      <div class="block">
        <select
          v-model="year"
          style="width: 100%; padding: 16px; margin-bottom: 15px; margin-top: 15px"
          placeholder="Выберите год"
        >
          <option value="2023">2023</option>
          <option value="2022">2022</option>
          <option value="2021">2021</option>
        </select>
        <br />
        <div class="buttons">
          <ButtonModule style="margin-right: 5px" size="s" color="purple-reverse" @click="new_data = true">
            Показать
          </ButtonModule>
          <ButtonModule
            size="s"
            color="purple-reverse"
            to="orders"
            @click="
              year = clearAll();
              new_data = false;
              searchQuery = '';
            "
          >
            Сбросить
          </ButtonModule>
        </div>
      </div>
    </template>
    <template v-slot:default>
      <div v-if="$route.query.id">
        <div
          v-for="note in info.filter(function (n) {
            return n.id === id;
          })"
          :key="note"
        >
          <div
            @click="
              $router.push({
                path: '/manager/orders',
                query: { id: note.id }
              })
            "
          >
            <div class="block">
              <h2><b>ID №</b>{{ note.id }}</h2>
              Телефон: {{ note.phone }} <br />
              Номер фотосессии: {{ note.psid }}<br />
              Клиент ID: {{ note.uid }}<br />
              Email: {{ note.email_payer }} <br />
              Дата: {{ note.date }} <br />
              Год: {{ note.date.substring(6, 10) }}
            </div>
          </div>
          <br />
        </div>
      </div>
      <div v-else>
        <div v-if="new_data">
          <div v-if="getInfo().length > 0">
            <div v-for="note in getInfo()" :key="note">
              <div
                @click="
                  id = note.id;
                  $router.push({ path: '/manager/orders', query: { id: note.id } });
                "
              >
                <div class="block">
                  <h2><b>ID №</b>{{ note.id }}</h2>
                  Телефон: {{ note.phone }} <br />
                  Номер фотосессии: {{ note.psid }}<br />
                  Клиент ID: {{ note.uid }}<br />
                  Email: {{ note.email_payer }} <br />
                  Дата: {{ note.date }} <br />
                  Год: {{ note.date.substring(6, 10) }}
                </div>
              </div>
            </div>
          </div>
          <div v-else>
            <Empty />
          </div>
        </div>
        <div v-else>
          <div v-for="note in info" :key="note">
            <div
              @click="
                id = note.id;
                $router.push({ path: '/manager/orders', query: { id: note.id } });
              "
            >
              <div class="block">
                <h2><b>ID №</b>{{ note.id }}</h2>
                Телефон: {{ note.phone }} <br />
                Номер фотосессии: {{ note.psid }}<br />
                Клиент ID: {{ note.uid }}<br />
                Email: {{ note.email_payer }} <br />
                Дата: {{ note.date }} <br />
                Год: {{ note.date.substring(6, 10) }}
              </div>
            </div>
          </div>
          <br />
        </div>
      </div>
    </template>
  </fixed-left-column>
</template>

<script>
import axios from "axios";

import fixedLeftColumn from "../../views/layout/fixed-left-column.vue";
import DateRange from "../../ui/date-range/index.vue";
import ButtonModule from "../../ui/button/index.vue";
import Empty from "../../ui/empty/index.vue";
import Search from "../../components/shared/search/index.vue";

export default {
  name: "OrdersList",
  data() {
    return {
      date_period: [],
      searchQuery: "",
      info: [],
      types: {
        order_number: { title: "Номер заказа" },
        psid: { title: "Номер фотосессии" },
        client_id: { title: "Клиент ID" },
        phone: { title: "Телефон" },
        email: { title: "Email" }
      },
      type: "order_number",
      year: "",
      id: "",
      search_data: {
        order_number: "",
        psid: "",
        client_id: "",
        phone: "",
        email: ""
      },
      new_data: false
    };
  },
  methods: {
    clearAll() {
      this.year = "";
      this.date_period = [];
      this.id = "";
      this.s_date = "";
      this.e_date = "";
      this.search_data.order_number = "";
      this.search_data.psid = "";
      this.search_data.client_id = "";
      this.search_data.phone = "";
      this.search_data.email = "";
    },
    checkData() {
      for (const el of Object.values(this.search_data)) {
        if (el !== "") {
          this.new_data = true;
          break;
        }
      }
      if (this.year) this.new_data = true;
      if (this.date_period) {
        this.new_data = true;
        console.log(this.date_period);
      }
    },
    compareDates(d1, d2) {
      // year
      if (parseInt(d1.substring(6, 10)) > d2.getFullYear()) return 1;
      if (parseInt(d1.substring(6, 10)) < d2.getFullYear()) return 2;
      // month
      if (parseInt(d1.substring(3, 5)) > d2.getMonth() + 1) return 1;
      if (parseInt(d1.substring(3, 5)) < d2.getMonth() + 1) return 2;
      // day
      if (parseInt(d1.substring(0, 2)) > d2.getDate()) return 1;
      if (parseInt(d1.substring(0, 2)) < d2.getDate()) return 2;
      // hour
      if (parseInt(d1.substring(13, 15)) > d2.getHours()) return 1;
      if (parseInt(d1.substring(13, 15)) < d2.getHours()) return 2;
      // minute
      if (parseInt(d1.substring(16, 18)) > d2.getMinutes()) return 1;
      if (parseInt(d1.substring(16, 18)) === d2.getMinutes()) return 0;
      if (parseInt(d1.substring(16, 18)) < d2.getMinutes()) return 2;
    },
    getInfo() {
      let data = this.info;
      if (this.date_period.length > 0) {
        data = data.filter(
          note =>
            (this.compareDates(note.date, this.date_period[0]) === 1 ||
              this.compareDates(note.date, this.date_period[0]) === 0) &&
            (this.compareDates(note.date, this.date_period[1]) === 2 ||
              this.compareDates(note.date, this.date_period[1]) === 0)
        );
      }
      if (this.year) {
        data = data.filter(note => note.date.substring(6, 10) === this.year);
      }
      if (this.search_data.psid) {
        data = data.filter(note => note.psid === this.search_data.psid);
      }
      if (this.search_data.phone) {
        data = data.filter(note => note.phone === this.search_data.phone);
      }
      if (this.search_data.email) {
        data = data.filter(
          note => note.email_payer === this.search_data.email || note.email_payer === this.search_data.email + " "
        );
      }
      if (this.search_data.client_id) {
        data = data.filter(note => note.uid === this.search_data.client_id);
      }
      if (this.search_data.order_number) {
        data = data.filter(note => note.id === this.search_data.order_number);
      }
      return data;
    }
  },
  components: { "fixed-left-column": fixedLeftColumn, DateRange, ButtonModule, Search, Empty },
  beforeMount() {
    if (this.$route.query.id) {
      this.id = this.$route.query.id;
    }
    
    try {
      axios.get("http://localhost:3001/lk/method/orders.getTest").then(response => {
        this.info = response.data.response.data.orders;
      });
    } catch (error) {}
  }
};
</script>

<style>
.block {
  padding: 16px;
  margin: 15px;
}

.buttons {
  display: flex;
  flex-direction: row;
  flex-grow: 1;
}
</style>

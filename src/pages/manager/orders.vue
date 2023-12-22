<template>
  <fixed-left-column>
    <template v-slot:fixed>
      <div class="block">
        <ButtonModule
          v-if="smallScreen()"
          size="s"
          color="purple-reverse"
          to="orders"
          @click="
            clearAll();
            new_data = false;
            searchQuery = '';
          "
        >
          Назад
        </ButtonModule>
        <DateRange
          v-model="date_period"
          placeholder="Выберите период"
          @change="changedDate($event), checkData()"
        /><br />
        <br />
        <Search
          v-model:search-query="searchQuery"
          v-model:type="type"
          :types="types"
          @click="addSearchData(), checkData()"
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
          <ButtonModule
            style="margin-right: 5px"
            size="s"
            color="purple-reverse"
            @click="
              search_data.year = year;
              $router.push({ path: '/manager/orders', query: { year: year } });
            "
          >
            Показать
          </ButtonModule>
          <ButtonModule
            size="s"
            color="purple-reverse"
            to="orders"
            @click="
              clearAll();
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
      <div v-if="checkData()">
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
        email: "",
        year: ""
      },
      new_data: false
    };
  },
  methods: {
    addSearchData() {
      this.search_data[String(this.type)] = this.searchQuery;
      if (this.searchQuery)
        this.$router.push({
          path: "/manager/orders",
          query: { search_type: this.type, search_value: this.searchQuery }
        });
      else this.$router.push({ path: "/manager/orders" });
    },
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
      this.search_data.year = "";
    },
    checkData() {
      for (const el of Object.values(this.search_data)) {
        if (el !== "") {
          return true;
        }
      }
      if (this.date_period && this.date_period.length > 0) {
        return true;
      }
      if (this.id) {
        return true;
      }
      return false;
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
      if (parseInt(d1.substring(0, 2)) === d2.getDate()) return 2;
    },
    getInfo() {
      let data = this.info;
      if (this.date_period && this.date_period.length > 0) {
        data = data.filter(
          note =>
            (this.compareDates(note.date, this.date_period[0]) === 1 ||
              this.compareDates(note.date, this.date_period[0]) === 0) &&
            (this.compareDates(note.date, this.date_period[1]) === 2 ||
              this.compareDates(note.date, this.date_period[1]) === 0)
        );
      }
      if (this.search_data.year) {
        data = data.filter(note => note.date.substring(6, 10) === this.search_data.year);
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
      if (this.id) {
        data = data.filter(note => note.id === this.id);
      }
      return data;
    },
    getDate(date) {
      const year = date.substring(0, 4);
      const month = date.substring(4, 6);
      const day = date.substring(6, 8);
      return new Date(year, month - 1, day);
    },
    changedDate(date) {
      this.date_period = [new Date(date.date_start * 1000), new Date(date.date_finish * 1000)];
      if (this.date_period && this.date_period[0])
        this.$router.push({
          path: "/manager/orders",
          query: {
            date_start: this.date_period[0].toISOString().slice(0, 10).replaceAll("-", ""),
            date_finish: this.date_period[1].toISOString().slice(0, 10).replaceAll("-", "")
          }
        });
    },
    smallScreen() {
      return window.innerWidth < 767;
    }
  },
  components: { "fixed-left-column": fixedLeftColumn, DateRange, ButtonModule, Search, Empty },
  beforeMount() {
    if (this.$route.query.id) {
      this.id = this.$route.query.id;
    }
    if (this.$route.query.year) {
      this.search_data.year = this.$route.query.year;
      this.year = this.$route.query.year;
    }
    if (this.$route.query.id) {
      this.id = this.$route.query.id;
    }
    if (this.$route.query.search_value) {
      this.type = this.$route.query.search_type;
      this.searchQuery = this.$route.query.search_value;
      this.search_data[this.$route.query.search_type] = this.$route.query.search_value;
    }
    if (this.$route.query.date_start && this.$route.query.date_finish) {
      this.date_period = [this.getDate(this.$route.query.date_start), this.getDate(this.$route.query.date_finish)];
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
@mixin mq($from, $to: false) {
  @if $to {
    @media (min-width: #{$from}px) and (max-width: #{$to}px) {
      @content;
    }
  } @else  {
    @media (max-width: #{$from}px) {
      @content;
    }
  }
}

.block {
  @include mq(767) {
    padding: 16px;
    margin: 15px;
  }
  font-size: small;
  padding: 16px;
  margin: 15px;
}

.buttons {
  display: flex;
  flex-direction: row;
  flex-grow: 1;
}
</style>

<template>
  <div id="app">
    <form action="#" @submit.prevent="getIssues">
      <div class="form-group">
        <input
          type="text"
          placeholder="owner/repo Name"
          v-model="repository"
          class="input"
        />
      </div>
    </form>

    <chart :issues="issues"></chart>
  </div>
</template>

<script>
import moment from "moment";
import axios from "axios";
import Chart from "./components/Chart.vue";
export default {
  name: "App",
  components: {
    Chart,
  },
  data() {
    return {
      issues: [],
      repository: "",
      startDate: null,
    };
  },
  methods: {
    getIssues() {
      this.startDate = moment().subtract(6, "days").format("YYYY-MM-DD");

      axios
        .get(
          `https://api.github.com/search/issues?q=repo:${this.repository}+is:issue+is:open+created:>=${this.startDate}`,
          { params: { per_page: 100 } }
        )
        .then((response) => {
          const payload = this.getDateRange();

          response.data.items.forEach((item) => {
            const key = moment(item.created_at).format("MMM Do YY");
            const obj = payload.filter((o) => o.day === key)[0];
            obj.issues += 1;
          });

          this.issues = payload;
          console.log(this.issues);
        });
    },
    getDateRange() {
      const startDate = moment().subtract(6, "days");
      const endDate = moment();
      const dates = [];

      while (startDate.isSameOrBefore(endDate)) {
        dates.push({
          day: startDate.format("MMM Do YY"),
          issues: 0,
        });

        startDate.add(1, "days");
      }
      return dates;
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.input {
  width: 300px;
  height: 40px;
  border-radius: 5px;
  border: 1px solid #ccc;
  padding: 0 15px;
  font-size: 14px;
  margin-bottom: 10px;
}
</style>

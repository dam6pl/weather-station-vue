<template>
  <div>
    <fade-transition :duration="1000" mode="out-in">
      <div class="row" v-if="!loading && !notFound">
        <div class="col-12 col-lg-6 mb-4">
          <h2 class="mb-0">
            Stacja: {{ this.currentStation.name }}
            <span style="font-size: small">
              ({{ this.currentStation.latitude }},
              {{ this.currentStation.longitude }})
            </span>
          </h2>
        </div>
        <div class="col-12 col-lg-6 mb-4">
          <div class="btn-group btn-group-toggle w-100" data-toggle="buttons">
            <label
              v-for="(option, index) in chartsCategories"
              :key="option"
              class="btn btn-sm btn-primary btn-simple"
              :class="{ active: temperatureChart.activeIndex === index }"
              :id="index"
              style="flex: 1"
            >
              <input
                type="radio"
                @click="initCharts(index)"
                name="options"
                autocomplete="off"
                :checked="temperatureChart.activeIndex === index"
              />
              {{ option }}
            </label>
            <div class="btn btn-sm btn-success btn-datepicker" style="flex: 1">
              <date-range-picker
                ref="picker"
                v-model="dateRange"
                :opens="windowWidth > 768 ? 'left' : 'center'"
                @update="initCharts(null)"
                :ranges="dateRanges"
                :max-date="date"
                :locale-data="localeData"
              >
                <div slot="input" slot-scope="picker">
                  {{ $moment(picker.startDate).format("DD.MM.YYYY") }}
                  -
                  {{ $moment(picker.endDate).format("DD.MM.YYYY") }}
                </div>
              </date-range-picker>
            </div>
          </div>
        </div>
      </div>
    </fade-transition>

    <fade-transition :duration="1000" mode="out-in">
      <div class="row" v-if="!loading && !notFound">
        <div class="col-12">
          <card type="chart">
            <template slot="header">
              <h5 class="card-category">Dane kolekcjonowane z czujników: DHT22, MPL115A2</h5>
              <h2 class="card-title">Temperatura</h2>
            </template>
            <div class="chart-area" v-if="!chartLoading">
              <line-chart
                style="height: 100%"
                ref="bigChart"
                chart-id="big-line-chart"
                :chart-data="temperatureChart.chartData"
                :gradient-colors="temperatureChart.gradientColors"
                :gradient-stops="temperatureChart.gradientStops"
                :extra-options="temperatureChart.extraOptions"
              ></line-chart>
            </div>
            <div class="loader" v-if="chartLoading">
              <div class="lds-ellipsis">
                <div></div>
                <div></div>
                <div></div>
                <div></div>
              </div>
            </div>
          </card>
        </div>
      </div>
    </fade-transition>

    <fade-transition :duration="1000" mode="out-in">
      <div class="row no-mb-lg" v-if="!loading && !notFound">
        <div class="col-lg-4">
          <card type="chart">
            <template slot="header">
              <h5 class="card-category">Dane kolekcjonowane z czujników: DHT22</h5>
              <h3 class="card-title">Wilgotność</h3>
            </template>
            <div class="chart-area" v-if="!chartLoading">
              <line-chart
                style="height: 100%"
                chart-id="purple-line-chart"
                :chart-data="humidityChart.chartData"
                :gradient-colors="humidityChart.gradientColors"
                :gradient-stops="humidityChart.gradientStops"
                :extra-options="humidityChart.extraOptions"
              ></line-chart>
            </div>

            <div class="loader" v-if="chartLoading">
              <div class="lds-ellipsis">
                <div></div>
                <div></div>
                <div></div>
                <div></div>
              </div>
            </div>
          </card>
        </div>
        <div class="col-lg-4">
          <card type="chart">
            <template slot="header">
              <h5 class="card-category">Dane kolekcjonowane z czujników: MPL115A2</h5>
              <h3 class="card-title">Ciśnienie atmosferyczne</h3>
            </template>
            <div class="chart-area" v-if="!chartLoading">
              <line-chart
                style="height: 100%"
                chart-id="purple-line-chart"
                :chart-data="pressureChart.chartData"
                :gradient-colors="pressureChart.gradientColors"
                :gradient-stops="pressureChart.gradientStops"
                :extra-options="pressureChart.extraOptions"
              ></line-chart>
            </div>

            <div class="loader" v-if="chartLoading">
              <div class="lds-ellipsis">
                <div></div>
                <div></div>
                <div></div>
                <div></div>
              </div>
            </div>
          </card>
        </div>
        <div class="col-lg-4">
          <card type="chart">
            <template slot="header">
              <h5 class="card-category">Dane kolekcjonowane z czujników: GL5528</h5>
              <h3 class="card-title">Natężenie światła</h3>
            </template>
            <div class="chart-area" v-if="!chartLoading">
              <line-chart
                style="height: 100%"
                chart-id="green-line-chart"
                :chart-data="illuminanceChart.chartData"
                :gradient-colors="illuminanceChart.gradientColors"
                :gradient-stops="illuminanceChart.gradientStops"
                :extra-options="illuminanceChart.extraOptions"
              ></line-chart>
            </div>
            <div class="loader" v-if="chartLoading">
              <div class="lds-ellipsis">
                <div></div>
                <div></div>
                <div></div>
                <div></div>
              </div>
            </div>
          </card>
        </div>
      </div>
    </fade-transition>

    <fade-transition :duration="1000" mode="out-in">
      <div class="content full" v-if="notFound">
        <div class="row justify-content-center align-items-center h-100">
          <div class="col-12 col-md-6 text-center">
            <h1 class="title text-danger">404 Nie znaleziono</h1>
            <h2 class="title">Oops! Strona której szukasz wydaje się nie istnieć.</h2>
          </div>
        </div>
      </div>
    </fade-transition>
  </div>
</template>
<script>
import LineChart from "@/components/Charts/LineChart";
import BarChart from "@/components/Charts/BarChart";
import * as chartConfigs from "@/components/Charts/config";
import config from "@/config";
import axios from "axios";
import { FadeTransition } from "vue2-transitions";
import DateRangePicker from "vue2-daterange-picker";
import "vue2-daterange-picker/dist/vue2-daterange-picker.css";

export default {
  components: {
    LineChart,
    BarChart,
    FadeTransition,
    DateRangePicker
  },
  data() {
    return {
      currentStation: null,
      loading: true,
      notFound: false,
      chartLoading: true,
      date: new Date(),
      dateRange: {
        startDate: this.$moment().startOf("month"),
        endDate: this.$moment()
      },
      dateRanges: {
        Dziś: [this.$moment(), this.$moment()],
        Wczoraj: [
          this.$moment().subtract(1, "days"),
          this.$moment().subtract(1, "days")
        ],
        "Ten miesiąc": [
          this.$moment().startOf("month"),
          this.$moment().endOf("month")
        ],
        "Ten rok": [
          this.$moment().startOf("year"),
          this.$moment().endOf("year")
        ],
        "Ostatni tydzień": [
          this.$moment()
            .subtract(1, "week")
            .startOf("week")
            .add(1, "day"),
          this.$moment()
            .subtract(1, "week")
            .endOf("week")
            .add(1, "day")
        ],
        "Ostatni miesiąc": [
          this.$moment()
            .subtract(1, "month")
            .startOf("month"),
          this.$moment()
            .subtract(1, "month")
            .endOf("month")
        ]
      },
      localeData: {
        applyLabel: "Szukaj",
        cancelLabel: "Anuluj"
      },
      windowWidth: 0,
      temperatureChart: {
        activeIndex: 0,
        chartData: {
          datasets: [
            {
              fill: true,
              borderColor: config.colors.primary,
              borderWidth: 2,
              borderDash: [],
              borderDashOffset: 0.0,
              pointBackgroundColor: config.colors.primary,
              pointBorderColor: "rgba(255,255,255,0)",
              pointHoverBackgroundColor: config.colors.primary,
              pointBorderWidth: 20,
              pointHoverRadius: 4,
              pointHoverBorderWidth: 15,
              pointRadius: 4,
              data: []
            }
          ],
          labels: [],
          units: " °C"
        },
        extraOptions: chartConfigs.purpleChartOptions,
        gradientColors: [
          "rgba(76, 211, 150, 0.1)",
          "rgba(53, 183, 125, 0)",
          "rgba(119,52,169,0)"
        ],
        gradientStops: [1, 0.4, 0],
        categories: []
      },
      humidityChart: {
        chartData: {
          datasets: [
            {
              fill: true,
              borderColor: "#e89805",
              borderWidth: 2,
              borderDash: [],
              borderDashOffset: 0.0,
              pointBackgroundColor: "#e89805",
              pointBorderColor: "rgba(255,255,255,0)",
              pointHoverBackgroundColor: "#e89805",
              pointBorderWidth: 20,
              pointHoverRadius: 4,
              pointHoverBorderWidth: 15,
              pointRadius: 4,
              data: []
            }
          ],
          labels: [],
          units: "%"
        },
        extraOptions: chartConfigs.purpleChartOptions,
        gradientColors: [
          "rgba(232, 152, 5, 0.1)",
          "rgba(232, 152, 5, 0)",
          "rgba(119,52,169,0)"
        ],
        gradientStops: [1, 0.4, 0],
        categories: []
      },
      illuminanceChart: {
        chartData: {
          datasets: [
            {
              fill: true,
              borderColor: "rgb(216, 17, 17)",
              borderWidth: 2,
              borderDash: [],
              borderDashOffset: 0.0,
              pointBackgroundColor: "rgb(216, 17, 17)",
              pointBorderColor: "rgba(255,255,255,0)",
              pointHoverBackgroundColor: "rgb(216, 17, 17)",
              pointBorderWidth: 20,
              pointHoverRadius: 4,
              pointHoverBorderWidth: 15,
              pointRadius: 4,
              data: []
            }
          ],
          labels: [],
          units: " lm"
        },
        extraOptions: chartConfigs.purpleChartOptions,
        gradientColors: [
          "rgba(216, 17, 17, 0.1)",
          "rgba(216, 17, 17, 0)",
          "rgba(119,52,169,0)"
        ],
        gradientStops: [1, 0.4, 0],
        categories: []
      },
      pressureChart: {
        chartData: {
          datasets: [
            {
              fill: true,
              borderColor: "rgb(3, 169, 244)",
              borderWidth: 2,
              borderDash: [],
              borderDashOffset: 0.0,
              pointBackgroundColor: "rgb(3, 169, 244)",
              pointBorderColor: "rgba(255,255,255,0)",
              pointHoverBackgroundColor: "rgb(3, 169, 244)",
              pointBorderWidth: 20,
              pointHoverRadius: 4,
              pointHoverBorderWidth: 15,
              pointRadius: 4,
              data: []
            }
          ],
          labels: [],
          units: " hPa"
        },
        extraOptions: chartConfigs.purpleChartOptions,
        gradientColors: [
          "rgba(3, 169, 244, 0.1)",
          "rgba(3, 169, 244, 0)",
          "rgba(119,52,169,0)"
        ],
        gradientStops: [1, 0.4, 0],
        categories: []
      }
    };
  },
  computed: {
    chartsCategories() {
      return ["Godzinowo", "Dziennie", "Miesięcznie"];
    }
  },
  methods: {
    initCharts(index) {
      if (index !== null) {
        if (this.temperatureChart.activeIndex === index) {
          return;
        }

        this.temperatureChart.activeIndex = index;
      } else {
        index = this.temperatureChart.activeIndex;
      }

      this.chartLoading = true;

      let url = "",
        from = "",
        date = new Date(),
        temperature = [],
        humidity = [],
        illuminance = [],
        pressure = [],
        labels = [];

      switch (index) {
        case 0:
          url =
            "?interval=hourly&from=" +
            this.$moment(this.dateRange.startDate).format(
              "YYYY-MM-DD HH:mm:ss"
            ) +
            "&to=" +
            this.$moment(this.dateRange.endDate).format("YYYY-MM-DD HH:mm:ss") +
            "&per_page=999999";
          break;
        case 1:
          url =
            "?interval=daily&from=" +
            this.$moment(this.dateRange.startDate).format(
              "YYYY-MM-DD HH:mm:ss"
            ) +
            "&to=" +
            this.$moment(this.dateRange.endDate).format("YYYY-MM-DD HH:mm:ss") +
            "&per_page=999999";
          break;
        case 2:
          url =
            "?interval=monthly&from=" +
            this.$moment(this.dateRange.startDate).format(
              "YYYY-MM-DD HH:mm:ss"
            ) +
            "&to=" +
            this.$moment(this.dateRange.endDate).format("YYYY-MM-DD HH:mm:ss") +
            "&per_page=999999";
          break;
      }

      axios
        .get(
          process.env.VUE_APP_API_URL +
            "v1/stations/" +
            this.$router.currentRoute.params.id +
            "/measurements" +
            url
        )
        .then(response => {
          if (response.data.success) {
            response.data.data.forEach(el => {
              temperature.push(el.temperature);
              humidity.push(el.humidity);
              illuminance.push(el.illuminance);
              pressure.push(el.pressure);
              switch (index) {
                case 0:
                  labels.push(
                    el.created_at
                      .replace(/\d{4}-\d{2}-\d{2}\s/, "")
                      .replace(/59/g, "00")
                  );
                  break;
                case 1:
                  labels.push(el.created_at.replace(/\s\d{2}:\d{2}:\d{2}/, ""));
                  break;
                case 2:
                  labels.push(
                    el.created_at.replace(/-\d{2}\s\d{2}:\d{2}:\d{2}/, "")
                  );
                  break;
              }
            });

            this.temperatureChart.chartData.datasets[0].data = temperature.reverse();
            this.humidityChart.chartData.datasets[0].data = humidity.reverse();
            this.illuminanceChart.chartData.datasets[0].data = illuminance.reverse();
            this.pressureChart.chartData.datasets[0].data = pressure.reverse();

            this.temperatureChart.chartData.labels = this.humidityChart.chartData.labels = this.illuminanceChart.chartData.labels = this.pressureChart.chartData.labels = labels.reverse();

            this.chartLoading = false;
          }
        });
    }
  },
  mounted() {
    this.$moment().locale("pl");
    axios
      .get(
        process.env.VUE_APP_API_URL +
          "v1/stations/" +
          this.$router.currentRoute.params.id
      )
      .then(response => {
        if (response.data.success) {
          this.currentStation = response.data.data;
          this.loading = false;
        }
      })
      .catch(error => {
        this.notFound = true;
        this.$notify({
          type: "danger",
          message: "Błąd strony: " + error.message
        });
      });

    this.initCharts(1);

    this.windowWidth = window.innerWidth;
    window.addEventListener("resize", () => {
      this.windowWidth = window.innerWidth;
    });
  }
};
</script>
<style></style>

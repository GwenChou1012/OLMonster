<template>
  <div class="flex">
    <div>
      <h1>前30天的歷史股價</h1>
      <h2>{{ stockId }} {{ name }}</h2>
      <h3>股價:{{ stockPrice }} 現金:{{ cash }} 目前持股: {{have}}</h3>
      <div class="container-fluid" id="chart">
        <apexchart
          type="area"
          height="350"
          :options="chartOptions"
          :series="series"
          v-if="series[0].data.length !== 0"
        ></apexchart>
      </div>
    </div>
    <div class="group">
      <input type="number" v-model="units" />
      <div class="span">
        <span v-if="units * stockPrice > this.$store.state.asset" style="color:red;">可憐哪~買入資金不足</span>
      </div>
      <div>
        <button type="button" class="btn btn-danger" @click="buy()">買入</button>
      </div>
      <input type="number" v-model="units2" />
      <div class="span">
        <span v-if="units2 > have" style="color:green;">醒醒吧~張數不足</span>
      </div>
      <div>
        <button type="button" class="btn btn-success" @click="sell()">賣出</button>
      </div>
    </div>
  </div>
</template>

<script>
import api from "../../utils/api";

export default {
  data() {
    return {
      stockId: this.$route.query.stockId,
      name: "",
      stockPrice: 0,
      units: 0,
      units2: 0,
      have: 0,
      cash: this.$store.state.asset,
      series: [
        {
          name: "股價",
          data: [],
        },
      ],
      chartOptions: {
        chart: {
          height: 350,
          type: "line",
          zoom: {
            enabled: true,
          },
        },
        dataLabels: {
          enabled: true,
        },
        stroke: {
          curve: "straight",
        },
        grid: {
          row: {
            colors: ["#f3f3f3", "transparent"], // takes an array which will be repeated on columns
            opacity: 0.5,
          },
        },
        colors: ["green"],
        yaxis: {
          labels: {
            show: false,
          },
        },
        xaxis: {
          labels: {
            show: false,
          },
          categories: [
            "29日",
            "28日",
            "27日",
            "26日",
            "25日",
            "24日",
            "23日",
            "22日",
            "21日",
            "20日",
            "19日",
            "18日",
            "17日",
            "16日",
            "15日",
            "14日",
            "13日",
            "12日",
            "11日",
            "10日",
            "9日",
            "8日",
            "7日",
            "6日",
            "5日",
            "4日",
            "3日",
            "2日",
            "1日",
            "今日",
          ],
        },
      },
    };
  },
  methods: {
    buy() {
      if (this.units < 0) {
        this.units = 0;
        alert("請輸入整數");
        return;
      }
      if (confirm("是否購買?")) {
        api
          .buystock({
            stockId: this.stockId,
            units: this.units,
            price: this.stockPrice,
          })
          .then((res) => {
            if (res.data.status == 200) {
              this.cash -= this.units * this.stockPrice;
              this.have += this.units;
              alert("買入成功");
            } else {
              alert("買入失敗");
            }
            api
              .getallstockdaydata()
              .then((res) => {
                let stocks = res.data.message;
                for (let i = 0; i < stocks.length; i++) {
                  if (stocks[i][0] == this.stockId) {
                    this.name = stocks[i][1];
                    this.stockPrice = stocks[i][2];
                  }
                }
                api
                  .userholdallstock()
                  .then((res) => {
                    let stocks = res.data.message;
                    let name = stocks.map((item) => item.stockName);
                    let userhave = stocks.map((item) => item.units);
                    for (let i = 0; i < name.length; i++) {
                      if (this.name == name[i]) {
                        this.have = userhave[i];
                      }
                    }
                  })
                  .catch(function (error) {
                    console.log("請求失敗", error);
                  });
              })
              .catch(function (error) {
                console.log(error);
              });
          })
          .catch(function (error) {
            console.log(error);
          });
      }
    },
    sell() {
      if (this.units2 < 0) {
        this.units2 = 0;
        alert("請輸入整數");
        return;
      }
      if (confirm("是否賣出?")) {
        api
          .sellstock({
            stockId: this.stockId,
            units: this.units2,
            price: this.stockPrice,
          })
          .then((res) => {
            console.log(res.data);
            if (res.data.status == 200) {
              this.cash += this.units * this.stockPrice;
              this.have -= this.units2;
              alert("賣出成功");
            } else {
              alert("賣出失敗");
            }
            api
              .getallstockdaydata()
              .then((res) => {
                let stocks = res.data.message;
                for (let i = 0; i < stocks.length; i++) {
                  if (stocks[i][0] == this.stockId) {
                    this.name = stocks[i][1];
                    this.stockPrice = stocks[i][2];
                  }
                }
                api
                  .userholdallstock()
                  .then((res) => {
                    let stocks = res.data.message;
                    let name = stocks.map((item) => item.stockName);
                    let userhave = stocks.map((item) => item.units);
                    for (let i = 0; i < name.length; i++) {
                      if (this.name == name[i]) {
                        this.have = userhave[i];
                      }
                    }
                  })
                  .catch(function (error) {
                    console.log("請求失敗", error);
                  });
              })
              .catch(function (error) {
                console.log(error);
              });
          })
          .catch(function (error) {
            console.log(error);
          });
      }
    },
  },
  mounted() {
    this.$store.commit("setAsset");
    let data = this.$route.query;
    api
      .get30before(data)
      .then((res) => {
        if (res.data.status == 200) {
          let data = res.data.message.before30price;
          let length = data.length;
          if (data[length - 1] > data[length - 2]) {
            this.chartOptions.colors = ["red"];
          } else if (data[length - 1] < data[length - 2]) {
            this.chartOptions.colors = ["green"];
          } else {
            this.chartOptions.colors = ["gray"];
          }
          // console.log(res.data.message.before30price);
          this.series[0].data = res.data.message.before30price;
        }
      })
      .catch(function (error) {
        console.log(error);
      });
    api
      .getallstockdaydata()
      .then((res) => {
        let stocks = res.data.message;
        for (let i = 0; i < stocks.length; i++) {
          if (stocks[i][0] == this.stockId) {
            this.name = stocks[i][1];
            this.stockPrice = stocks[i][2];
          }
        }
        api
          .userholdallstock()
          .then((res) => {
            let stocks = res.data.message;
            let name = stocks.map((item) => item.stockName);
            let userhave = stocks.map((item) => item.units);
            for (let i = 0; i < name.length; i++) {
              if (this.name == name[i]) {
                this.have = userhave[i];
              }
            }
          })
          .catch(function (error) {
            console.log("請求失敗", error);
          });
      })
      .catch(function (error) {
        console.log(error);
      });
  },
};
</script>

<style src="./style.css" scoped></style>

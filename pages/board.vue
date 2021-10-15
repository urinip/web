<template>
  <v-row>
    <v-col class="text-center">
      <v-card class="mx-auto" max-width="300">
        <v-card-title>{{ currentTimeSeries }}</v-card-title>
        <v-simple-table v-if="isFileLoaded" dense>
          <thead>
            <tr>
              <th class="text-left">sell</th>
              <th class="text-left">price</th>
              <th class="text-left">buy</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in boardSells[currentTimeSeries]" :key="item.Price">
              <td>{{ item.Qty }}</td>
              <td>{{ item.Price }}</td>
            </tr>
            <tr v-for="item in boardBuys[currentTimeSeries]" :key="item.Price">
              <td></td>
              <td>{{ item.Price }}</td>
              <td>{{ item.Qty }}</td>
            </tr>
          </tbody>
        </v-simple-table>
        <v-card-text v-if="isFileLoaded">
          <v-row>
            {{ rawTimeSerires[currentTimeSeries].CurrentPriceTime }}
          </v-row>
          <v-row>
            {{
              priceChangeStatusMapping[
                rawTimeSerires[currentTimeSeries].CurrentPriceChangeStatus
              ]
            }}
          </v-row>
          <v-row>
            {{ rawTimeSerires[currentTimeSeries].CurrentPrice }}
          </v-row>
          <v-row>
            {{ rawTimeSerires[currentTimeSeries].TradingVolume }}
          </v-row>
        </v-card-text>
        <v-slider
          v-if="isFileLoaded"
          v-model="currentTimeSeries"
          :max="maxTimeSeries"
        ></v-slider>
        <v-file-input @change="getFilecontent"></v-file-input>
      </v-card>
    </v-col>
    <v-col class="text-center">
      <v-card class="mx-auto" max-width="400">
        <v-card-title>{{ currentTimeSeries }}</v-card-title>
        <v-simple-table v-if="isFileLoaded" dense>
          <thead>
            <tr>
              <th class="text-left">time</th>
              <th class="text-left">volume</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in ticks">
              <td>{{ item.time }}</td>
              <td>{{ item.volume }}</td>
              <td>{{ item.price }}</td>
            </tr>
          </tbody>
        </v-simple-table>
      </v-card>
    </v-col>
  </v-row>
</template>


<script lang="ts">
// import { Vue, Component } from 'vue-property-decorator'
import Vue from 'vue'
import Component from 'vue-class-component'

type Quote = {
  Price: number
  Qty: number
  Sign: string
  Time: string
}

type Board = {
  Buy1: Quote
  Buy2: Quote
  Buy3: Quote
  Buy4: Quote
  Buy5: Quote
  Buy6: Quote
  Buy7: Quote
  Buy8: Quote
  Buy9: Quote
  Buy10: Quote

  Sell1: Quote
  Sell2: Quote
  Sell3: Quote
  Sell4: Quote
  Sell5: Quote
  Sell6: Quote
  Sell7: Quote
  Sell8: Quote
  Sell9: Quote
  Sell10: Quote

  CurrentPrice: number
  CurrentPriceTime: string
  CurrentPriceChangeStatus: string
  TradingVolume: number
  TradingVolumeTime: string
}

@Component
export default class extends Vue {
  public isFileLoaded = false
  public boardSells: Quote[][] = []
  public boardBuys: Quote[][] = []
  public ticks: any[] = []

  public maxTimeSeries: number = 0
  public currentTimeSeries: number = 0
  public rawTimeSerires: Board[] = []

  public priceChangeStatusMapping = {
    '0000': '事象なし',
    '0056': '変わらず',
    '0057': 'UP',
    '0058': 'DOWN',
    '0059': '中断板寄り後の初値',
    '0060': 'ザラバ引け',
    '0061': '板寄り引け',
    '0062': '中断引け',
    '0063': 'ダウン引け',
    '0064': '逆転終値',
    '0066': '特別気配引け',
    '0067': '一時留保引け',
    '0068': '売買停止引け',
    '0069': 'サーキットブレーカ引け',
    '0431': 'ダイナミックサーキットブレーカ引け',
  }

  public getFilecontent(file: File) {
    this.readFileAsync(file).then((jsonl) => {
      const jsons = jsonl.split('\n')
      this.rawTimeSerires = jsons
        .map((x) => {
          return JSON.parse(x || 'null')
        })
        .filter((x) => x)
      this.maxTimeSeries = jsons.length - 1
      this.boardSells = this.rawTimeSerires.map((x) => {
        return [
          x.Sell10,
          x.Sell9,
          x.Sell8,
          x.Sell7,
          x.Sell6,
          x.Sell5,
          x.Sell4,
          x.Sell3,
          x.Sell2,
          x.Sell1,
        ]
      })

      this.boardBuys = this.rawTimeSerires.map((x) => {
        return [
          x.Buy1,
          x.Buy2,
          x.Buy3,
          x.Buy4,
          x.Buy5,
          x.Buy6,
          x.Buy7,
          x.Buy8,
          x.Buy9,
          x.Buy10,
        ]
      })

      this.ticks = this.rawTimeSerires
        .map((x, i) => {
          if (i + 1 >= this.maxTimeSeries) {
            return null
          }

          return {
            time: this.rawTimeSerires[i + 1].TradingVolumeTime,
            volume: this.rawTimeSerires[i + 1].TradingVolume - x.TradingVolume,
            price: this.rawTimeSerires[i + 1].CurrentPrice,
          }
        })
        .filter((x) => x)
        .filter((x) => {
          if (x === null) {
            return false
          } else {
            return x.volume !== 0
          }
        })
        .reverse()

      console.log(this.ticks)

      this.isFileLoaded = true
    })
  }

  private readFileAsync(file: File): Promise<string> {
    return new Promise((resolve, reject) => {
      const reader = new FileReader()
      reader.onload = () => {
        resolve(reader.result as string)
      }
      reader.onerror = reject
      reader.readAsText(file)
    })
  }
}
</script>
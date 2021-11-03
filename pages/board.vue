<template>
  <v-row>
    <v-col class="text-center">
      <v-card class="mx-auto" max-width="400">
        <v-card-title>{{ currentTimeSeries }}</v-card-title>
        <v-simple-table v-if="isFileLoaded" dense>
          <thead>
            <tr>
              <th class="text-left">change</th>
              <th class="text-left">sell</th>
              <th class="text-left">price</th>
              <th class="text-left">buy</th>
              <th class="text-left">change</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="board in boards[currentTimeSeries]" :key="board.price">
              <td>{{ board.sellChange }}</td>
              <td>{{ board.sellVolume }}</td>
              <td>{{ board.price }}</td>
              <td>{{ board.buyVolume }}</td>
              <td>{{ board.buyChange }}</td>
            </tr>
            <!-- <tr v-for="item in boardBuys[currentTimeSeries]" :key="item.Price">
              <td></td>
              <td></td>
              <td>{{ item.Price }}</td>
              <td>{{ item.Qty }}</td>
              <td>{{ item.Diff }}</td>
            </tr>
          </tbody> -->
          </tbody></v-simple-table
        >

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
  Diff: number
}

type Raw = {
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

type BoardRow = {
  sellChange: number
  sellVolume: number
  price: number
  buyVolume: number
  buyChange: number
}

@Component
export default class extends Vue {
  public isFileLoaded = false
  public boards: BoardRow[][] = []
  public ticks: any[] = []

  public maxTimeSeries: number = 0
  public currentTimeSeries: number = 0
  public rawTimeSerires: Raw[] = []

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
      const sells = this.rawTimeSerires
        .map((x) => {
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
        .map(this.calcDiff)

      const buys = this.rawTimeSerires
        .map((x) => {
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
        .map(this.calcDiff)

      this.boards = sells.map((rawBoard, index) => {
        const boardSells = rawBoard.reduce<Map<number, BoardRow>>((map, x) => {
          map.set(x.Price, {
            sellChange: x.Diff,
            sellVolume: x.Qty,
            price: x.Price,
            buyVolume: 0,
            buyChange: 0,
          })
          return map
        }, new Map())

        const boardBuys = buys[index].reduce<Map<number, BoardRow>>(
          (map, x) => {
            map.set(x.Price, {
              sellChange: 0,
              sellVolume: 0,
              price: x.Price,
              buyVolume: x.Qty,
              buyChange: x.Diff,
            })
            return map
          },
          new Map()
        )

        let board = boardSells

        boardBuys.forEach((buy) => {
          const sell = boardSells.get(buy.price)
          if (sell) {
            board.set(buy.price, {
              sellChange: sell.sellChange,
              sellVolume: sell.sellVolume,
              price: sell.price,
              buyVolume: buy.buyVolume,
              buyChange: buy.buyChange,
            })
          } else {
            board.set(buy.price, buy)
          }
        })

        console.log(board)

        return Array.from(board.values())
      })

      // return board

      // this.boards = this.boardSells.map((board, index) => {
      //   return board.map((quote) => {
      //     const buy = this.boardBuys[index].find((element) => {
      //       return element.Price === quote.Price
      //     })
      //     const res: BoardRow = {
      //       sellChange: quote.Diff,
      //       sellVolume: quote.Qty,
      //       price: quote.Price,
      //       buyVolume: buy ? buy.Qty : 0,
      //       buyChange: buy ? buy.Diff : 0,
      //     }

      //     return res
      //   })
      // })

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

  private calcDiff(current: Quote[], index: number, array: Quote[][]) {
    if (index === 0) {
      return current
    }
    const previous = array[index - 1].reduce<Map<number, Quote>>((map, x) => {
      map.set(x.Price, x)
      return map
    }, new Map())

    const onlyPreviousQuotes = Array.from(previous.values())
      .filter((x) => {
        return !current.find((element) => {
          return element.Price === x.Price
        })
      })
      .map((v) => {
        const res: Quote = {
          Price: v.Price,
          Qty: 0,
          Diff: -v.Qty,
          Sign: '',
          Time: '',
        }
        return res
      })

    return current
      .map((x) => {
        const previousQty = previous.get(x.Price)
        if (previousQty === undefined) {
          x.Diff = x.Qty
        } else {
          x.Diff = x.Qty - previousQty.Qty
        }
        return x
      })
      .concat(onlyPreviousQuotes)
      .sort((a, b) => b.Price - a.Price)
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
<template>
  <div class="columns selected-section">
    <div class="column is-7 image-section">
      <img :src="`/static/${selectedCryptoCurrency.id}_large_logo.png`" class="cryptoCurrency-image"/>
      <h1 class="cryptoCurrency-title">{{ selectedCryptoCurrency.name }}</h1>
      <span class="tag is-primary">Rank {{ selectedCryptoCurrency.rank}}</span>
      <p class="cryptoCurrency-description">{{ selectedCryptoCurrency.description }}</p>
      <div class="icons-section">
        <span v-if="selectedCryptoCurrency.website" class="icon">
          <a :href="selectedCryptoCurrency.website" target="_blank"><icon name="link" scale="2"></icon></a>
        </span>
        <span v-if="selectedCryptoCurrency.paper" class="icon">
          <a :href="selectedCryptoCurrency.paper" target="_blank"><icon name="file-text" scale="2"></icon></a>
        </span>
        <span v-if="selectedCryptoCurrency.github" class="icon">
          <a :href="selectedCryptoCurrency.github" target="_blank"><icon name="github" scale="2"></icon></a>
        </span>
      </div>
    </div>
    <div class="column information-section">
      <div class="dropdown-section">
        <div class="control">
          <button class="button" type="button" @click="toggleDropDown">
            {{ selectedFiatCurrency }}
            <span class="arrow-icon">
              <icon v-if="!dropDownOpen" name="caret-down"></icon>
              <icon v-if="dropDownOpen" name="caret-up"></icon>
            </span>
          </button>

          <div class="box dropdown" :class="{'is-open': dropDownOpen}">
            <ul>
              <li v-for="fiatCurrency in fiatCurrencies"><a class="nav-item" @click="selectFiatCurrency(fiatCurrency)">{{ fiatCurrency }}</a></li>
            </ul>
          </div>
        </div>
      </div>
      <div class="price-section">
        <p class="price-tag">Current Price</p>
        <p class="price-amount">{{ selectedFiatCurrency }} {{ selectedCryptoCurrency.selectedPrice }} 
          <span :class="{'positive-percent-change': positivePercentChange, 'negative-percent-change': negativePercentChange}">
            ({{ selectedCryptoCurrency.percent_change_24h }}%)
          </span>
        </p>
      </div>
      <div class="price-section">
        <p class="price-tag">Circulating Supply</p>
        <p class="price-amount">{{ selectedCryptoCurrency.selectedSupply }} {{ selectedCryptoCurrency.symbol }}</p>
      </div>
      <div class="price-section">
        <p class="price-tag">Market Cap</p>
        <p class="price-amount">{{ selectedFiatCurrency }} {{ selectedCryptoCurrency.selectedMarketCap }} </p>
      </div>
    </div>
  </div>
</template>

<script>
import { EventBus } from '../../event-bus.js'
const fiatCurrencies = [ 'AUD', 'BRL', 'CAD', 'CHF', 'CNY', 'EUR', 'GBP', 'HKD', 'IDR', 'INR', 'JPY', 'USD', 'KRW', 'MXN', 'RUB' ]

export default {
  name: 'selectedHero',
  data () {
    return {
      selectedCryptoCurrency: {},
      fiatCurrencies: fiatCurrencies,
      selectedFiatCurrency: fiatCurrencies[11],
      positivePercentChange: true,
      negativePercentChange: false,
      dropDownOpen: false
    }
  },
  created () {
    EventBus.$on('cryptoCurrencySelected', cryptoCurrency => {
      if (cryptoCurrency.percent_change_24h.indexOf('-') > -1) {
        this.positivePercentChange = false
        this.negativePercentChange = true
        cryptoCurrency.percent_change_24h = cryptoCurrency.percent_change_24h.replace(/^-/, '')
      }
      cryptoCurrency.selectedPrice = Number(cryptoCurrency.price_usd).toFixed(2)
      cryptoCurrency.selectedSupply = Number(cryptoCurrency.available_supply).toLocaleString()
      cryptoCurrency.selectedMarketCap = Number(cryptoCurrency.market_cap_usd).toLocaleString()
      this.selectedCryptoCurrency = cryptoCurrency
    })
  },
  beforeDestroy () {
    EventBus.$off('cryptoCurrencySelected')
  },
  methods: {
    toggleDropDown () {
      this.dropDownOpen = !this.dropDownOpen
    },
    selectFiatCurrency (fiatCurrency) {
      this.selectedFiatCurrency = fiatCurrency
      this.$http.get(`https://api.coinmarketcap.com/v1/ticker/${this.selectedCryptoCurrency.id}/?convert=${this.selectedFiatCurrency}`)
        .then(cryptoCurrency => {
          this.toggleDropDown()
          this.selectedCryptoCurrency.selectedPrice = Number(cryptoCurrency.body[0]['price_' + this.selectedFiatCurrency.toLowerCase()]).toLocaleString(
            undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })
          this.selectedCryptoCurrency.selectedMarketCap = Number(cryptoCurrency.body[0]['market_cap_' + this.selectedFiatCurrency.toLowerCase()]).toLocaleString()
        })
    }
  }
}
</script>

<style scoped lang="scss">
.selected-section {
  .image-section {
    position: relative;

    .cryptoCurrency-image {
      height: 200px;

      -webkit-animation-name: spinner; 
      -webkit-animation-timing-function: linear; 
      -webkit-animation-iteration-count: infinite; 
      -webkit-animation-duration: 60s; 
      animation-name: spinner; 
      animation-timing-function: linear; 
      animation-iteration-count: infinite; 
      animation-duration: 60s; 
      -webkit-transform-style: preserve-3d; 
      -moz-transform-style: preserve-3d; 
      -ms-transform-style: preserve-3d; 
      transform-style: preserve-3d;
    }

    .cryptoCurrency-title {
      font-size: 20px;
      font-weight: 600;
    }

    .cryptoCurrency-description {
      font-size: 14px;
      line-height: 1.65;
      max-width: 450px;
      display: block;
      margin: 0 auto;
      padding: 20px 0 30px;
    }

    .icons-section {
      .icon {
        margin: 0px 30px;
        -o-transition: .5s;
        -ms-transition: .5s;
        -moz-transition: .5s;
        -webkit-transition: .5s;
        transition: .5s;

        &:hover {
          color: #fd6721;
        }
      }
    }
  }

  .information-section {
    position: relative;
    padding-top: 25px;

    .dropdown-section {
      position: absolute;
      right: 50px;

      .control {
        .button {
          width: 81px;
          background-color: transparent;
          color: #FFF;
          position: relative;

          &:focus {
            border-color: #FFF;
          }

          .arrow-icon {
            position: absolute;
            right: 5px;
            bottom: 1px;
            color: #fd6721;

            icon {
              position: absolute;
              right: 5px;
              bottom: 1px;
            }
          }
        }

        .box {
          padding: 7px;
          background-color: transparent;
        }

        .dropdown {
          box-shadow: 0 0 3px 0 #fd6721;
          display: none;
          left: 0;
          position: absolute;
          top: 100%;
          z-index: 1000;
          height: 150px;
          overflow-y: scroll;
          
          ul {
          }

          &.is-open {
            display: block;
          }

          .nav-item {
            color: #FFF;
            -o-transition: .5s;
            -ms-transition: .5s;
            -moz-transition: .5s;
            -webkit-transition: .5s;
            transition: .5s;

            &:hover {
              color: #fd6721;
            }
          }
        }
      }
    }

    .price-section {
      text-align: left;
      padding: 20px 0;

      .price-tag {
      }

      .price-amount {
        font-size: 40px;

        .positive-percent-change {
          color: #00d1b2
        }

        .negative-percent-change {
          color: #ff3860
        }
      }
    }
  }
}

@-webkit-keyframes spinner {
    from 
    { 
        -webkit-transform: rotateY(0deg); 
    } 
    to { 
        -webkit-transform: rotateY(-360deg); 
    } 
}
@keyframes spinner { 
    from { 
        -moz-transform: rotateY(0deg); 
        -ms-transform: rotateY(0deg); 
        transform: rotateY(0deg); 
    } 
    to 
    { 
        -moz-transform: rotateY(-360deg); 
        -ms-transform: rotateY(-360deg); 
        transform: rotateY(-360deg); 
    
    } 
}

::-webkit-scrollbar {
    -webkit-appearance: none;
    width: 7px;
}
::-webkit-scrollbar-thumb {
    border-radius: 4px;
    background-color: #FFF;
    -webkit-box-shadow: 0 0 1px rgba(255,255,255,.5);
}
</style>
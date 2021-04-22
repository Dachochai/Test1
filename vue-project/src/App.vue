<template>
  <div id="app">
    <img src="./assets/logo.png">
    <router-view/>
    <RecycleScroller emit-update @update="handleUpdate" page-mode :items="Object.keys(items)" :item-size="50" :buffer="500">
      <template v-slot="{ item: key, index }">
      <div :class="`item-css ${index%2 ? 'even' : 'odd'}`">{{ items[key] }}</div>
      </template>
    </RecycleScroller>
  </div>
</template>

<script>
import { RecycleScroller } from 'vue-virtual-scroller'
import axios from 'axios'
import { throttle } from 'lodash'

// const items = new Array(50).fill().map((each, index) => {
//     const id = index + 1
//     return {
//         id,
//         name: `Item ${id}`
//     }
// })

const url = 'https://jsonplaceholder.typicode.com/photos'
const chunkSize = 50
const aheadBuffer = 10

export default {
  name: 'App',

    components: {
        RecycleScroller
    },

    data() {
        return {
            items: {},
            itemTotal: 0
        }
    },

    created() {
        this.fetchPhotos = throttle(this.fetchPhotos, 5000)
    },

    async mounted() {
        this.fetchPhotos()
    },

    methods: {
        handleUpdate(stratIndex, endIndex) {
            if (!endIndex) {
                return
            }

            const itemCount = Object.keys(this.items).length
            if (!(endIndex >= itemCount - aheadBuffer)) {
                console.log('data still in range')
                console.log(1234)
                return
            }

            if (!(itemCount < this.itemTotal)) {
                return
            }

            this.fetchPhotos(itemCount / chunkSize + 1)
        },

        async fetchPhotos(page = 1) {
            const { headers, data } = await axios.get(`${url}?_limit=${chunkSize}&_page=${page}`)

            this.itemTotal = parseInt(headers['x-total-count'])

            this.items = data.reduce((acc, each, index) => {
                const key = (page - 1) * chunkSize + index

                return {
                    ...acc,
                    [key]: { ...each }
                }
            }, this.items)
        }
    }
}
</script>

<style scoped>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

.item-css {
  height: 50px;
  width: 100%;
  position: absolute;
}

.even {
  background-color: antiquewhite;
}

.odd {
  background-color: aquamarine;
}
</style>

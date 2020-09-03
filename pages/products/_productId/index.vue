<template>
  <v-card v-if="product != null">
    <v-card-title>{{ product.title }}</v-card-title>
    <v-card-text>
      <v-carousel
        cycle
        height="400"
        hide-delimiter-background
        show-arrows-on-hover
      >
        <v-carousel-item v-for="(picture, i) in productPictures" :key="i">
          <v-img :src="picture.source" :height="300" contain></v-img>
        </v-carousel-item>
      </v-carousel>

      <v-row align="center">
        <v-rating
          readonly
          :value="product.score || 4.7"
          half-increments
          background-color="orange lighten-3"
          color="orange"
          medium
          dense
          class="px-2"
        ></v-rating>
        <span class="px-2">{{ product.score || 4.5 }} ( 1344 reviews ) </span>
      </v-row>

      <v-row>
        <v-col>
          <v-card
            v-for="review in productReviews"
            :key="review.id"
            class="my-2"
          >
            <v-card-title>
              <v-rating
                readonly
                :value="review.score"
                half-increments
                background-color="orange lighten-3"
                color="orange"
                small
                dense
                class="pr-2"
              />

              {{ review.title }}
            </v-card-title>
            <v-card-text>{{ review.content }} </v-card-text>
            <v-card-actions>
              <div
                v-if="review.authorUsername != null"
                justify="center"
                class="mx-2"
              >
                <v-icon small>person</v-icon>{{ review.authorUsername }}
              </div>
              <div justify="center">
                <v-icon small>schedule</v-icon>
                {{ toLocalDateTime(review.createdOn) }}
              </div>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </v-card-text>
  </v-card>
</template>

<script>
const { DateTime } = require('luxon')
export default {
  data() {
    return {
      product: null,
      productPictures: [],
      productReviews: []
    }
  },
  computed: {
    productId() {
      return this.$route.params.productId
    }
  },
  created() {
    this.$axios
      .get(`/product/${this.productId}`)
      .then((response) => {
        this.product = response.data
        this.getProductPictures()
        this.getProductReviews(this.product.universalProductCode)
      })
      .catch((ex) => {
        console.error('Could not fetch product: ' + this.productId, ex)
      })
  },
  methods: {
    getProductPictures() {
      this.loadingPictures = true
      this.$axios
        .get(`/product/${this.productId}/pictures`)
        .then((response) => {
          this.productPictures = response.data
        })
        .catch((ex) => {
          this.productPictures = []
          console.error('Could not fetch product pictures: ', ex)
        })
        .finally(() => {
          this.loadingPictures = false
        })
    },
    /**
     * @param {String} gtin - gtin = Global Trade Item Number
     */
    getProductReviews(gtin) {
      this.loadingReviews = true
      this.$axios
        .get(`/review/gtin/${gtin}/search`)
        .then((response) => {
          this.productReviews = response.data.items
          this.productReviews = this.productReviews.filter(
            (review) => review.title != null
          )
        })
        .catch((ex) => {
          console.error(
            'Could not fetch product reviews for product: ' + this.ProductId,
            ex
          )
        })
        .finally(() => (this.loadingReviews = false))
    },
    toLocalDateTime(dateTime) {
      return DateTime.fromISO(dateTime).toFormat('dd-LL-yyyy HH:mm:ss')
    }
  }
}
</script>
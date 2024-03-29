<template>
  <v-card height="100%">
    <map-view
      :mapStyle="mapViewClass"
      :map="map"
      :markers="markers"
      @clickedOnMap="clickedOnMap"
      @clickedMarker="clickedMarker"
    />
    <post-form
      v-if="postFormVisible"
      :position="map.center"
      @success="postSuccess"
      @failed="postFailed"
    />
  </v-card>
</template>

<script>
import MapView from '../components/MapView.vue'
import PostForm from '../components/PostForm.vue'
import firebase from '../plugins/firebase.js'
import { mapActions } from 'vuex'

export default {
  components: {
    MapView,
    PostForm
  },
  data () {
    return {
      map: {
        center: { lat: 35.696096, lng: 139.776776 },
        zoom: 15
      },
      markers: [],
      fullScreenMap: false,
      reviewVisible: true,
      postFormVisible: false,
      mapViewClass: 'width: 100vw; height: 100%'
    }
  },
  computed: {
    fullScreen: () => ('width: 100%; height: 100%'),
    separateScreen: () => ('width: 100%; height: 30%')
  },
  // asyncDataはpagesコンポーネントでのみ使用できる
  async asyncData (context) {
    // firebaseのデータfetch
    const querySnapshot = await firebase.firestore().collection('storeInfos').get()
    const records = querySnapshot.docs.map(elem => elem.data())
    if (records.length > 0) {
      return { markers: records }
    }
  },
  mounted () {
    if (!navigator.geolocation) {
      // 現在位置を取得できない場合はデフォルトのmapオブジェクトをそのまま使用する
    } else {
      // 現在位置をマップの中央にセット
      navigator.geolocation.getCurrentPosition((position) => {
        this.map.center.lat = position.coords.latitude
        this.map.center.lng = position.coords.longitude
      })
    }
  },
  methods: {
    ...mapActions('snackbar', ['snackOn']),
    ...mapActions('snackbar', ['setMessage']),
    async clickedOnMap (center) {
      await firebase.auth().onAuthStateChanged((user) => {
        if (user) {
          this.mapViewClass = this.separateScreen
          this.map.center = center
          this.fullScreenMap = false
          this.reviewVisible = false
          this.postFormVisible = true
        } else {
          this.setMessage('レビューを投稿するにはログインする必要があります')
          this.snackOn()
        }
      })
    },
    async clickedMarker (store) {
      await firebase.auth().onAuthStateChanged((user) => {
        if (user) {
          this.setMessage('レビューを投稿するにはログインする必要があります')
          this.snackOn()
        } else {
          this.mapViewClass = this.separateScreen
          this.map.center = store.position
          this.fullScreenMap = false
          this.reviewVisible = true
          this.postFormVisible = true
        }
      })
    },
    setFullscreenMap () {
      this.mapViewClass = this.fullScreen
      this.fullScreenMap = true
      this.reviewVisible = false
      this.postFormVisible = false
    },
    postSuccess () {
      this.mapViewClass = this.fullScreen
      this.fullScreenMap = false
      this.reviewVisible = true
      this.postFormVisible = false
    },
    postFailed () {
      this.mapViewClass = this.fullScreen
      this.fullScreenMap = false
      this.reviewVisible = true
      this.postFormVisible = false
    }
  }
}
</script>

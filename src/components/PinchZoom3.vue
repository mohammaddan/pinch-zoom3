<template>
  <div id='entryPoint' 
        :style="'width:'+width+';height:'+height+';background: url('+image+
       ');background-size:'+(scale*100)+
       '% ;background-repeat:no-repeat;background-position:-'
       +leftOffset+'px -'+topOffset+'px'"
       style="background-color: white"
       @touchend.prevent="gestureEnd"
       @touchstart.prevent="gestureStart"
       @touchmove.prevent="gestureMove"
       @mousedown="panStart" @mousemove="panMove" @mouseup="firstTouches=touches=[]"
       @wheel="mouseWheel" ref="pz">
  </div>
</template>

<script>
export default {
  name: "PinchZoom3",
  props: {image: {},width: {default: '100%'}},
  data() {
    return {
      touches: [],
      firstTouches: [],
      scale: 1,
      top: 0,
      left: 0,
      leftOffset: 0,
      topOffset: 0,
      height:'250px',
      originWidth:300,
      originHeight:300,
      maxScale:2,
    }
  },
  mounted(){
    if(this.image){
      let img=new Image();
      img.src=this.image
      let vm=this
      img.onload=(i)=>{
        this.originHeight=img.height
        this.originWidth=img.width
        let w=this.$refs.pz.clientWidth
        let h=w *img.height/ img.width
        this.maxScale=2*this.originWidth/w;
        if(h) vm.height=h+'px'
      }
    }
  },
  methods: {
    lineLength(p) {
      let x = p[0].clientX - p[1].clientX
      let y = p[0].clientY - p[1].clientY;
      return Math.sqrt(x ** 2 + y ** 2);
    },
    averagePoint(p) {
      let left = (p[0].clientX + p[1].clientX) / 2
      let top = (p[0].clientY + p[1].clientY) / 2;
      return {left, top}
    },
    mouseWheel(e) {
      let client = e.target.getBoundingClientRect()
      let oldScale = this.scale
      this.scale *= (e.wheelDeltaY > 0 ? 1.1 : 0.91)
      if (this.scale < 1) this.scale = 1
      if(this.scale>this.maxScale) this.scale=this.maxScale
      this.zoomImage(e.x - client.x, e.y - client.y, client.width, client.height, oldScale, this.scale)
    },
    panStart(e){
      console.log(e)
      this.firstTouches = this.touches=[{clientX: e.clientX, clientY: e.clientY}]
    },
    gestureStart(e) {
      if (e.touches?.length > 1) {
        this.firstTouches = this.touches = e.touches
      } else if (e.touches?.length === 1) {
        this.firstTouches = this.touches = e.touches
      }
    },
    panMove(e){
      let client = e.target.getBoundingClientRect()
      if(this.touches?.length === 1 && this.firstTouches?.length === 1){
        let t = e
        this.leftOffset -= (t.clientX - this.firstTouches[0].clientX)
        if (this.leftOffset > client.width * (this.scale - 1)) this.leftOffset = client.width * (this.scale - 1);
        if (this.leftOffset < 0) this.leftOffset = 0
        this.topOffset -= (t.clientY - this.firstTouches[0].clientY)
        if (this.topOffset > client.height * (this.scale - 1)) this.topOffset = client.height * (this.scale - 1);
        if (this.topOffset < 0) this.topOffset = 0
        this.firstTouches = [{clientX: e.clientX, clientY: e.clientY}]
      }
    },
    gestureMove(e) {
      let client = e.target.getBoundingClientRect()
      if (this.touches?.length > 1 && e.touches?.length > 1 && this.firstTouches.length > 1) {
        let oldScale = this.scale
        let t = e.touches
        let w = Math.abs(this.lineLength(t) / this.lineLength(this.touches));
        this.touches = t;
        let temp = this.averagePoint(this.touches)
        this.left = temp.left - client.x
        this.top = temp.top - client.y
        this.scale *= w
        if (this.scale < 1) this.scale = 1
        if(this.scale>this.maxScale) this.scale=this.maxScale
        this.zoomImage(this.left, this.top, client.width, client.height, oldScale, this.scale)
      } else if (this.touches?.length === 1 && e.touches?.length === 1 && this.firstTouches.length === 1) {
        let t = e.touches[0]
        this.leftOffset -= (t.clientX - this.firstTouches[0].clientX)
        if (this.leftOffset > client.width * (this.scale - 1)) this.leftOffset = client.width * (this.scale - 1);
        if (this.leftOffset < 0) this.leftOffset = 0
        this.topOffset -= (t.clientY - this.firstTouches[0].clientY)
        if (this.topOffset > client.height * (this.scale - 1)) this.topOffset = client.height * (this.scale - 1);
        if (this.topOffset < 0) this.topOffset = 0

        this.firstTouches = e.touches
      }
    },
    gestureEnd() {
      this.touches = []
    },
    zoomImage(clientX, clientY, clientWidth, clientHeight, oldScale, newScale) {
      let sr = newScale / oldScale;
      let offsetX = (this.leftOffset + clientX) * sr
      let offsetY = (this.topOffset + clientY) * sr

      this.leftOffset = Math.max(0, offsetX - clientX)
      this.topOffset = Math.max(0, offsetY - clientY)

      if (this.leftOffset > clientWidth * (newScale - 1)) this.leftOffset = clientWidth * (newScale - 1);
      if (this.topOffset > clientHeight * (newScale - 1)) this.topOffset = clientHeight * (newScale - 1);
    }
  },
}
</script>


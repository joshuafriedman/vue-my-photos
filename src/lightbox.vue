<template>
  <div>
    <transition name="lightbox-fade">
      <div
        class="lightbox"
        v-show="visible"
        @mousedown.stop="hide"
        @touchdown.stop="hide"
      >
        <div
          class="lightbox-close"
          @mousedown.stop="hide"
          @touchdown.stop="hide"
        >
          &times;
        </div>
        <div
          class="lightbox-zoom-in unselectable"
          @mousedown.stop="zoomIn"
          @touchdown.stop="zoomIn"
        >
          +
        </div>
        <div
          class="lightbox-zoom-out unselectable"
          @mousedown.stop="zoomOut"
          @touchdown.stop="zoomOut"
        >
          &minus;
        </div>
        <div
          class="lightbox-arrow lightbox-arrow-left"
          @mousedown.stop.prevent="prev"
          @touchdown.stop.prevent="prev"
        >
          <transition name="lightbox-fade">
            <div
              class="lightbox-arrow-left-icon"
              v-show="has_prev() && controlsVisible"
            >
              <svg
                height="24"
                viewBox="0 0 24 24"
                width="24"
                xmlns="http://www.w3.org/2000/svg"
              >
                <circle cx="12" cy="12" r="12" fill="rgba(20, 20, 20, 0.4)" />
                <path
                  d="M15.41 16.09l-4.58-4.59 4.58-4.59L14 5.5l-6 6 6 6z"
                  fill="white"
                />
                <path d="M0-.5h24v24H0z" fill="none" />
              </svg>
            </div>
          </transition>
        </div>
        <div
          class="lightbox-arrow lightbox-arrow-right"
          @mousedown.stop.prevent="next"
          @touchdown.stop.prevent="next"
        >
          <transition name="lightbox-fade">
            <div
              class="lightbox-arrow-right-icon"
              v-show="has_next() && controlsVisible"
            >
              <svg
                height="24"
                viewBox="0 0 24 24"
                width="24"
                xmlns="http://www.w3.org/2000/svg"
              >
                <circle cx="12" cy="12" r="12" fill="rgba(20, 20, 20, 0.4)" />
                <path
                  d="M8.59 16.34l4.58-4.59-4.58-4.59L10 5.75l6 6-6 6z"
                  fill="white"
                />
                <path d="M0-.25h24v24H0z" fill="none" />
              </svg>
            </div>
          </transition>
        </div>
        <transition name="lightbox-fade">
          <div
            class="lightbox-caption"
            v-show="controlsVisible && filteredImages[index].alt"
            @mousedown.stop
            @touchdown.stop
          >
            <span unselectable="on">{{ filteredImages[index].alt }}</span>
          </div>
        </transition>
        <div
          class="lightbox-main"
          @mousedown.stop="hide"
          @touchdown.stop="hide"
        >
          <div class="lightbox-image-container" @mousedown.stop @touchdown.stop>
            <transition :name="slideTransitionName" mode="out-in">
              <img
                class="lightbox-image"
                :key="index"
                :src="directory + filteredImages[index].name"
                v-if="visible"
              />
            </transition>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  name: "MyLightBox",
  props: {
    // images = [{ name:'image1.jpg', alt:'Redwoods', filter:'nature' }, ...]
    images: {
      type: Array,
      required: true
    },
    // Used to display a subset of photos. If used, images array must contain a property
    // titled 'filter' which contains the desired filter string
    filter: {
      type: String,
      default: "all"
    },
    // Used if images are located in a separate folder (e.g. '/images/')
    directory: {
      type: String,
      default: ""
    },
    // Used to set the duration in miliseconds of key/mouse inactivity before caption
    // and arrows disappear
    timeoutDuration: {
      type: Number,
      default: 3000
    }
  },
  data: function() {
    return {
      visible: false, // Lightbox not visible by default
      controlsVisible: true, // Lightbox controls (arrows, caption, close button)
      index: 0, // Index indicates which photo to display. Default to 1st photo
      timer: null, // Timer to show/hide lightbox controls
      slideTransitionName: "lightbox-slide-next", //Controls animation's transition direction (next or prev)
      dimensions: [] // original height and width of last seen picture
    };
  },
  mounted() {
    window.addEventListener("resize", this.resizeListener);
    window.addEventListener("keydown", this.keyEventListener);
    window.addEventListener("mousemove", this.mouseEventListener);
    window.addEventListener("touchmove", this.mouseEventListener);
    window.addEventListener("mouseup", this.mouseEventListener);
    // document.getElementsByClassName("lightbox-image")[0].maxWidth = window.innerWidth - 100;
  },
  destroyed() {
    window.removeEventListener("resize", this.resizeListener);
    window.removeEventListener("keydown", this.keyEventListener);
    window.removeEventListener("mousemove", this.mouseEventListener);
    window.removeEventListener("touchmove", this.mouseEventListener);
    window.removeEventListener("mouseup", this.mouseEventListener);
  },
  methods: {
    show(imageName) {
      imageName = imageName.replace("http://localhost:8080", "");
      imageName = imageName.replace(/%20/g, " ");

      //   document.getElementsByClassName("lightbox-image")[0].height =
      //     window.innerHeight;
      // Find the index of the image passed to Lightbox
      for (var i = 0; i < this.filteredImages.length; i++) {
        if (this.filteredImages[i].name == imageName) {
          this.index = i;
          this.visible = true;
          this.controlsVisible = true;
          var that = this;
          break;
        }
      }

      clearTimeout(this.timer);
      this.timer = setTimeout(function() {
        that.controlsVisible = false;
      }, that.timeoutDuration);
      this.preloadNextImage();
    },
    hide() {
      this.visible = false;
      this.index = 0;
      clearTimeout(this.timer);
      //   document.getElementsByClassName("lightbox-image")[0].style.cssText ="";
    },
    has_next() {
      return this.index + 1 < this.filteredImages.length;
    },
    has_prev() {
      return this.index - 1 >= 0;
    },
    prev() {
      if (this.has_prev()) {
        this.slideTransitionName = "lightbox-slide-prev";
        this.index -= 1;
        // document.getElementsByClassName("lightbox-image")[0].height = window.innerHeight;
        // document.getElementsByClassName("lightbox-image")[0].style.height = window.innerHeight;
      }
    },
    next() {
      if (this.has_next()) {
        this.slideTransitionName = "lightbox-slide-next";
        this.index += 1;
        this.preloadNextImage();
        // document.getElementsByClassName("lightbox-image")[0].height = window.innerHeight;
        // document.getElementsByClassName("lightbox-image")[0].style.height = window.innerHeight;
      }
    },
    keyEventListener(e) {
      if (this.visible) {
        var that = this;
        this.controlsVisible = true;
        clearTimeout(this.timer);
        this.timer = setTimeout(function() {
          that.controlsVisible = false;
        }, that.timeoutDuration);

        switch (e.key) {
          case "ArrowRight":
            this.next();
            break;
          case "ArrowLeft":
            this.prev();
            break;
          case "ArrowDown":
          case "ArrowUp":
          case " ":
            e.preventDefault();
            break;
          case "Escape":
            this.hide();
            break;
          case "m":
            this.zoomIn();
            break;
          case "n":
            this.zoomOut();
            break;
          case "w":
            document
              .getElementsByClassName("lightbox-image-container")[0]
              .scrollBy(0, -5);
            break;
          case "d":
            document
              .getElementsByClassName("lightbox-image-container")[0]
              .scrollBy(5, 0);
            break;
          case "s":
            document
              .getElementsByClassName("lightbox-image-container")[0]
              .scrollBy(0, 5);
            break;
          case "a":
            document
              .getElementsByClassName("lightbox-image-container")[0]
              .scrollBy(-5, 0);
            break;
        }
      }
    },
    // This event shows the arrows and caption on the lightbox when the mouse is moved or clicked.
    // Also used for touch events on touchscreen devices. The elements are set to disappear
    // after a given duration via a timer.
    mouseEventListener() {
      if (this.visible) {
        var that = this;
        this.controlsVisible = true;
        clearTimeout(this.timer);
        this.timer = setTimeout(function() {
          that.controlsVisible = false;
        }, that.timeoutDuration);
      }
    },
    preloadNextImage() {
      if (this.has_next()) {
        try {
          var _img = new Image();
          _img.src = this.directory + this.filteredImages[this.index + 1].name;
          //   document.getElementsByClassName("lightbox-image")[0].height =
          //     window.innerHeight;
        } catch (e) {
          window.console.log(e);
        }
      }
    },
    resizeListener: function() {
    //   window.console.log("resizin");
      // window.console.log(document.getElementsByClassName("lightbox-image")[0].width);

      // document.getElementsByClassName("lightbox-image")[0].height = window.innerHeight;

      // window.console.log(document.getElementsByClassName("lightbox-image")[0].maxWidth);
      // window.console.log(document.getElementsByClassName("lightbox-image")[0].width);
      // if(window.innerWidth-document.getElementsByClassName("lightbox-image")[0].width<128) document.getElementsByClassName("lightbox-image")[0].width=window.innerWidth-128;
    },
    zoomIn: function() {
      const el = document.getElementsByClassName("lightbox-image-container")[0];
    
      const img = document.getElementsByClassName("lightbox-image")[0];
      img.width = img.width * 1.25;
      img.height = img.height * 1.25;
      //   document.getElementsByClassName("lightbox-image")[0].style.cssText = "min-height: 400px !important";
      document.getElementsByClassName("lightbox-image")[0].style.cssText =
        "height: " + img.height * 1.25 + "px !important; ";
      
      // check if scroll height is bigger than window size
      const height_diff = el.scrollHeight - window.innerHeight;
      var sy = 0;
      if (height_diff > 0)
        sy = el.scrollTop * 1.25 + 0.125 * window.innerHeight;
      const width_diff = el.scrollWidth - window.innerWidth;
      var sx = 0;
      if (width_diff > 0) sx = el.scrollLeft * 1.25 + 0.125 * window.innerWidth;
      if (width_diff - 0.2 * el.scrollWidth < 2 && width_diff > 0) {
        sx = width_diff / 2;
      }
      el.scroll(sx, sy);

          },
    zoomOut: function() {
      const el = document.getElementsByClassName("lightbox-image-container")[0];
      const img = document.getElementsByClassName("lightbox-image")[0];
      const scroll_height = el.scrollTop;
      const scroll_width = el.scrollLeft;
      
      // check if scroll height is bigger than window size
      const window_height = window.innerHeight;
      
      const img_height = img.height * 1.25; // cuz we multiplies by 0.8 a few lines above
      var syp = 0.8 * scroll_height - 0.1 * window_height;
      var over_bounds_y = syp + window_height - 0.8 * img_height;
      if (over_bounds_y > 0) syp -= over_bounds_y;


      const window_width = window.innerWidth;
      
      const img_width = img.width; 
      var sxp = 0.8 * scroll_width - 0.1 * window_width;
      var over_bounds_x = sxp + window_width - 0.8 * img_width;
         if (over_bounds_x > 0) sxp -= over_bounds_x;

      el.scroll(sxp, syp);
        img.width = img.width * 0.8;
      img.height = img.height * 0.8;
      document.getElementsByClassName("lightbox-image")[0].style.cssText =
        "height: " + img.height * 0.8 + "px !important;";

        // now check if img_height<window height
        // if(img_height*0.8*0.8<window_height){
        //     // center the image by moving it
        //     document.getElementsByClassName("lightbox-image")[0].style.cssText =
        // "height: " + img.height * 0.8 + "px !important; " + "top: "+(window_height-img_height*0.8*0.8)/2+"px; "+"position: relative;";
        // }

    }
  },
  computed: {
    filteredImages: function() {
      var that = this;
      if (that.filter === "all" || !that.filter.length) {
        return that.images;
      } else {
        return that.images.filter(function(item) {
          return item.filter === that.filter;
        });
      }
    }
  }
};
</script>

<style scoped>
.lightbox {
  position: fixed;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.9);
  width: 100%;
  height: 100%;
  display: -webkit-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 200;
  color: rgba(255, 255, 255, 0.8);
}
.lightbox-close {
  position: fixed;
  z-index: 210;
  right: 0;
  top: 0;
  padding: 1rem;
  font-size: 1.7rem;
  cursor: pointer;
  width: 4rem;
  height: 4rem;
  color: white;
}
.lightbox-zoom-in {
  position: fixed;
  z-index: 210;
  left: 0;
  top: 0;
  margin: 1rem;
  font-size: 1.7rem;
  cursor: pointer;
  width: 1.9rem;
  height: 1.9rem;
  color: white;
}
.lightbox-zoom-out {
  position: fixed;
  z-index: 210;
  left: 30px;
  top: 0px;
  margin: 1rem;
  font-size: 1.7rem;
  cursor: pointer;
  width: 1.9rem;
  height: 1.9rem;
  color: white;
}

.lightbox-main {
  display: -webkit-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  width: 100%;
  height: 100%;
}

.lightbox-arrow {
  padding: 0 2rem;
  display: -webkit-box;
  display: -ms-flexbox;
  display: -webkit-flex;
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  padding: 0 2rem;
  height: 100%;
  width: 2rem;
  z-index: 100;
}

.lightbox-arrow-right {
  right: 0;
}

.lightbox-arrow-left {
  left: 0;
}

.lightbox-arrow-right-icon {
  cursor: pointer;
  padding: 20px;
}

.lightbox-arrow-left-icon {
  cursor: pointer;
  padding: 20px;
}

.lightbox-image-container {
  overflow: scroll;
  -webkit-box-flex: 1;
  width: 20%;
  -webkit-flex: 1;
  -ms-flex: 1;
  flex: 1;
}

.lightbox-image {
  height: 100%;
  background-size: contain;
  background-repeat: no-repeat;
  margin-left: auto;
  margin-right: auto;
  position: relative;
}

.lightbox-caption {
  position: absolute;
  bottom: 15px;
  width: 100%;
  z-index: 100;
  text-align: center;
  text-shadow: 1px 1px 3px rgb(26, 26, 26);
}

.lightbox-caption span {
  border-radius: 12px;
  background-color: rgba(0, 0, 0, 0.6);
  padding: 2px 10px;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.lightbox-slide-next-enter-active,
.lightbox-slide-next-leave-active,
.lightbox-slide-prev-enter-active,
.lightbox-slide-prev-leave-active {
  transition: all 0.4s ease;
}

.lightbox-slide-next-enter {
  -webkit-transform: translateX(100px);
  -ms-transform: translateX(100px);
  transform: translateX(100px);
  opacity: 0;
}

.lightbox-slide-next-leave-to {
  -webkit-transform: translateX(-100px);
  -ms-transform: translateX(-100px);
  transform: translateX(-100px);
  opacity: 0;
}

.lightbox-slide-prev-enter {
  -webkit-transform: translateX(-100px);
  -ms-transform: translateX(-100px);
  transform: translateX(-100px);
  opacity: 0;
}

.lightbox-slide-prev-leave-to {
  -webkit-transform: translateX(100px);
  -ms-transform: translateX(100px);
  transform: translateX(100px);
  opacity: 0;
}

.lightbox-fade-enter-active,
.lightbox-fade-leave-active {
  transition: all 0.4s ease;
}

.lightbox-fade-enter,
.lightbox-fade-leave-to {
  opacity: 0;
}
.unselectable {
  -moz-user-select: -moz-none;
  -khtml-user-select: none;
  -webkit-user-select: none;

  /*
     Introduced in IE 10.
     See http://ie.microsoft.com/testdrive/HTML5/msUserSelect/
   */
  -ms-user-select: none;
  user-select: none;
}
</style>

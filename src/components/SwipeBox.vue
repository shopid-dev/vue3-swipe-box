<template>
  <div class="mainswiperc" ref="mainswiper">
    <slot />
  </div>
</template>

<script>
export default {
  name: 'SwipeBox',
  props: {
    tes: String,
    speed: {
      default: 100,
    },
  },
  data: function () {
    return {
      mainEl: {},
      selfSwC: {
        start: 0,
        mainpos: 0,
        stpx: 0,
        numx: 0,
        swdirection: 'ltr',

        mousedown: false,
      },
    };
  },

  methods: {
    doSwip: function (anim) {
      this.$emit('onChange', this.selfSwC.stpx);

      

      var xpos = 0;
      if (
        typeof this.mainEl.children[this.selfSwC.stpx] != 'undefined' &&
        typeof this.mainEl.children[this.selfSwC.stpx].offsetLeft != 'undefined'
      ) {
        xpos = this.mainEl.children[this.selfSwC.stpx].offsetLeft;
      }

      var maincntx = this;
      if (anim == true) {
        this.animate(
          function (newValue) {
            maincntx.mainEl.scrollLeft = newValue;
          },
          maincntx.mainEl.scrollLeft,
          xpos,
          maincntx.speed,
          function (x) {
            return x;
          }
        );
      } else {
        maincntx.mainEl.scrollLeft = xpos;
      }

      this.selfSwC.mainpos = -1 * xpos;
    },

    swipetoNext: function () {
      if (this.selfSwC.stpx < this.mainEl.childElementCount - 1) {
        this.selfSwC.stpx++;
        this.doSwip(true);
      } else {
        this.selfSwC.mainpos = this.mainEl.children[this.selfSwC.stpx].offsetLeft*-1;
      }
    },
    swipetoPrevious: function () {
      if (this.selfSwC.stpx > 0) {
        this.selfSwC.stpx--;
        this.doSwip(true);
      } else {
        this.selfSwC.mainpos = 0;
      }
    },

    goTo: function (id) {
      if (id >= 0 && id < this.mainEl.childElementCount) {
        this.selfSwC.stpx = id;
        this.doSwip(true);
      }
    },

    animate: function (render, from, to, duration, timeFx) {
      var startTime = performance.now();
      requestAnimationFrame(function step(time) {
        var pTime = (time - startTime) / duration;
        if (pTime > 1) pTime = 1;
        render(from + (to - from) * timeFx(pTime));
        if (pTime < 1) {
          requestAnimationFrame(step);
        }
      });
    },

    swipStart: function (e) {
      e.preventDefault();
      this.selfSwC.mousedown = true;
      if (typeof e.changedTouches !== 'undefined') {
        this.selfSwC.start = e.changedTouches[0].pageX;
      } else {
        this.selfSwC.start = e.pageX;
      }
    },

    swiping: function (e) {
      var dif = 0;
      if (this.selfSwC.mousedown) {
        e.preventDefault();
        e.stopPropagation();
        if (typeof e.changedTouches !== 'undefined') {
          dif = e.changedTouches[0].pageX - this.selfSwC.start;
        } else {
          dif = e.pageX - this.selfSwC.start;
        }

        this.mainEl.scrollLeft = -1 * (this.selfSwC.mainpos + dif);
      }
    },

    swipingtouch: function (e) {
      var dif = 0;
      if (this.selfSwC.mousedown) {
        // e.preventDefault();
        e.stopPropagation();
        if (typeof e.changedTouches !== 'undefined') {
          dif = e.changedTouches[0].pageX - this.selfSwC.start;
        } else {
          dif = e.pageX - this.selfSwC.start;
        }

        this.mainEl.scrollLeft = -1 * (this.selfSwC.mainpos + dif);
      }
    },

    swipEnd: function (e) {
      var dif = 0;
      this.selfSwC.mousedown = false;
      if (typeof e.changedTouches !== 'undefined') {
        dif = e.changedTouches[0].pageX - this.selfSwC.start;
      } else {
        dif = e.pageX - this.selfSwC.start;
      }

      this.selfSwC.mainpos = this.selfSwC.mainpos + dif;

      if (dif < 0) {
        if (this.swdirection == 'ltr') {
          this.swipetoNext();
        } else {
          this.swipetoPrevious();
        }
      } else if (dif > 0) {
        if (this.swdirection == 'rtl') {
          this.swipetoNext();
        } else {
          this.swipetoPrevious();
        }
      } else if (dif == 0) {
        if (typeof this.selfSwC.onClick !== 'undefined') {
          //this.selfSwC.onClick(this.selfSwC.numx, fields.items[this.selfSwC.numx]);
        }
      }
    },

    mouseout: function (e) {
      if (this.selfSwC.mousedown) {
        this.swipEnd(e);
      }
    },

    rendersw: function () {
      var mainsw = this.$refs.mainswiper.firstElementChild;

      if (
        window.getComputedStyle(mainsw, null).getPropertyValue('direction') ==
        'rtl'
      ) {
        this.swdirection = 'rtl';
      } else {
        this.swdirection = 'ltr';
      }

      mainsw.style.position = 'relative';
      mainsw.style.display = 'flex';

      /*if (
        'ontouchstart' in window ||
        navigator.maxTouchPoints > 0 ||
        navigator.msMaxTouchPoints > 0
      ) {
        mainsw.style.overflow = 'scroll';
      } else {
        mainsw.style.overflow = 'hidden';
      }
      */
      
      mainsw.style.overflow = 'hidden';

      this.mainEl = mainsw;

      for (var i = 0; i < mainsw.childElementCount; i++) {
        mainsw.children[i].style.boxSizing = 'border-box';
        mainsw.children[i].style.minWidth = '100%';
      }

      mainsw.removeEventListener('mousedown', this.swipStart, true);
      mainsw.removeEventListener('mousemove', this.swiping, true);
      mainsw.removeEventListener('mouseup', this.swipEnd, true);
      mainsw.removeEventListener('mouseout', this.mouseout, true);
      mainsw.removeEventListener('touchstart', this.swiping, true);
      mainsw.removeEventListener('touchmove', this.swipEnd, true);
      mainsw.removeEventListener('touchend', this.mouseout, true);

      mainsw.addEventListener('mousedown', this.swipStart, true);
      mainsw.addEventListener('mousemove', this.swiping, true);
      mainsw.addEventListener('mouseup', this.swipEnd, true);
      mainsw.addEventListener('mouseout', this.mouseout, true);

      mainsw.addEventListener('touchstart', this.swipStart, true);
      mainsw.addEventListener('touchmove', this.swipingtouch, true);
      mainsw.addEventListener('touchend', this.swipEnd, true);

      if (mainsw.childElementCount - 1 < this.selfSwC.stpx) {
        this.goTo(mainsw.childElementCount - 1);
      }
    },
  },

  mounted() {
    this.rendersw();
    var maincp = this;
    window.addEventListener('resize', function () {
      if (typeof maincp.goTo !== 'undefined') {
        maincp.doSwip(false);
      }
    });
  },
  updated() {
    this.rendersw();
  },
};
</script>

<style>
.mainswiperc > *::-webkit-scrollbar {
  display: none;
}

.mainswiperc > * {
  -ms-overflow-style: none;
  scrollbar-width: none;
}
</style>


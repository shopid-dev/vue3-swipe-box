<template>
  <div class="mainswiperc" ref="mainswiper">
    <slot />
  </div>
</template>

<script>
export default {
  name: 'SwipBox',
  props: {
    tes: String,
  },
  data: function () {
    return {
      mainEl: {},
      selfSwC: {
        start: 0,
        mainpos: 0,
        stpx: 0,
        numx: 0,

        mousedown: false,
      },
    };
  },

  methods: {
    doSwip: function (anim) {
      this.$emit('onChange', this.selfSwC.stpx);
      var allelems = this.mainEl.getElementsByTagName('*');


     var xpos = 0;
      if (typeof allelems[this.selfSwC.stpx] != "undefined" && typeof allelems[this.selfSwC.stpx].offsetLeft != "undefined") {
        xpos = allelems[this.selfSwC.stpx].offsetLeft;
      } 

      var maincntx = this;
      if (anim == true) {
        this.animate(
          function (newValue) {
            maincntx.mainEl.scrollLeft = newValue;
          },
          maincntx.mainEl.scrollLeft,
          xpos,
          50,
          function (x) {
            return x;
          }
        );
      } else {
        maincntx.mainEl.scrollLeft = xpos;
      }

      this.selfSwC.mainpos = -1 * xpos;
    },

    swiptoLeft: function () {
      if (
        this.selfSwC.stpx <
        this.mainEl.getElementsByTagName('*').length - 1
      ) {
        this.selfSwC.stpx++;
        this.doSwip(true);
      }
    },
    swiptoRight: function () {
      if (this.selfSwC.stpx > 0) {
        this.selfSwC.stpx--;
        this.doSwip(true);
      }
    },

    goTo: function (id) {
      if (id >= 0 && id < this.mainEl.getElementsByTagName('*').length) {
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
      if (this.selfSwC.mousedown) {
        e.preventDefault();
        e.stopPropagation();
        var dif = 0;
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
      this.selfSwC.mousedown = false;
      var dif = 0;
      if (typeof e.changedTouches !== 'undefined') {
         dif = e.changedTouches[0].pageX - this.selfSwC.start;
      } else {
         dif = e.pageX - this.selfSwC.start;
      }

      this.selfSwC.mainpos = this.selfSwC.mainpos + dif;

      if (dif < 0) {
        this.swiptoLeft();
      } else if (dif > 0) {
        this.swiptoRight();
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
      var mainsw = this.$refs.mainswiper.getElementsByTagName('*')[0];

      mainsw.style.position = 'relative';
      mainsw.style.display = 'flex';

      if (
        'ontouchstart' in window ||
        navigator.maxTouchPoints > 0 ||
        navigator.msMaxTouchPoints > 0
      ) {
        mainsw.style.overflow = 'scroll';
       
      } else {
       
        mainsw.style.overflow = 'hidden';
      }

      this.mainEl = mainsw;

      var allel = this.mainEl.getElementsByTagName('*');

      for (var i = 0; i < allel.length; i++) {
        allel[i].style.boxSizing = 'border-box';
        allel[i].style.minWidth = '100%';
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

      if (allel.length - 1 < this.selfSwC.stpx) {
        this.goTo(allel.length - 1);
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
.mainswiperc:first-child ::-webkit-scrollbar {
  display: none;
}

.mainswiperc:first-child {
  -ms-overflow-style: none;
  scrollbar-width: none;
}
</style>


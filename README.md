# vue3-swipe-box
Swipe any elements to left and right in vue.js\
*for vue2 use [@shopid/vue2-swipe-box](https://www.npmjs.com/package/@shopid/vue2-swipe-box)*

## installation

```
npm i @shopid/vue3-swipe-box
```

### How to use
you just need one box inside SwipeBox , and children inside that box
```
    <SwipeBox>
      <div style="width:200px;height:200px;border:1px solid black">
        <div>1</div>
        <div>2</div>
        <div>3</div>
      </div>
    </SwipeBox>
```
- support both right to left and left to right (rtl,ltr) css layouts
- support both mouse and touch

## Live demo on [stackblitz](https://stackblitz.com/edit/vue-xsw3r9)


### onChange
onChange method will call when user swipe

### speed
you can change speed of magnet swiping with speed prop

### swipetoNext , swipetoPrevious , goTo
you can swipe to next previous and index by calling these methods : (swipetoNext , swipetoPrevious , goTo)

#### soruce on github [https://github.com/shopid-dev/vue3-swipe-box](https://github.com/shopid-dev/vue3-swipe-box)

### Full example
```
<template>
  <div id="app">
    <SwipeBox ref="myswipe" @onChange="mySwipeChanged" speed="150">
      <div style="width: 200px; height: 200px; border: 1px solid black">
        <div v-for="image in images" :key="image.id">
        <img :src="image.url" />
        </div>
      </div>
    </SwipeBox>

    <button @click="next">next</button><br />
    <button @click="previous">previous</button><br />
    <button @click="goto2">goto index #2</button><br />
  </div>
</template>

<script>
import SwipeBox from '@shopid/vue3-swipe-box';

export default {
  name: 'App',
  components: {
    SwipeBox,
  },
  data: () => {
    return {
      images: [
        { id: '1', url: 'https://picsum.photos/200' },
        { id: '2', url: 'https://picsum.photos/200?2' },
        { id: '3', url: 'https://picsum.photos/200?3' },
        { id: '4', url: 'https://picsum.photos/200?4' },
        { id: '5', url: 'https://picsum.photos/200?5' },
      ],
    };
  },

  methods: {
    mySwipeChanged: function (index) {
      console.log('index' + index);
    },

    next: function (x) {
      this.$refs.myswipe.swipetoNext();
    },
    previous: function (x) {
      this.$refs.myswipe.swipetoPrevious();
    },
    goto2: function () {
      this.$refs.myswipe.goTo(2);
    },
  },
};
</script>


```





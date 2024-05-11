<template>
  <div class="main" :style="{ backgroundPosition: startOffset.x + 'px ' + startOffset.y + 'px' }">
    <div class="main__plane" ref="plane"
      :style="{ transform: 'translate(' + startOffset.x + 'px, ' + startOffset.y + 'px)' }"><img
        src="@/assets/plane.svg"></div>
    <button :class="{ 'main__button': !isActive, 'main__button_active': isActive }" @click="handleClick">
      {{ isActive ? 'Стоп' : 'Почати' }}
    </button>
  </div>
</template>

<script>
import anime from 'animejs';
import flightData from '../assets/flight_data.json';

export default {
  name: 'FlightPath',
  data() {
    return {
      flightData,
      isActive: false,
      keyframes: [],
      animation: null,
      timeline: anime.timeline({ duration: 100 }),
      startOffset: { x: 0, y: 0 },
      x: 0,
      y: 0,
    };
  },
  mounted() {
    this.plane = this.$refs.plane;

    const max = [0, 0];
    const min = [0, 0];

    let tmpX = 0;
    let tmpY = 0;
    for (let i = 0; i < this.flightData.length - 1; i++) {
      const { x, y } = this.calculate(flightData[i], flightData[i + 1],);
      tmpX += x;
      tmpY -= y;
      if (tmpX > max[0]) max[0] = tmpX;
      if (tmpX < min[0]) min[0] = tmpX;
      if (tmpY > max[1]) max[1] = tmpY;
      if (tmpY < min[1]) min[1] = tmpY;
    }

    const startOffsetX = -(max[0] - min[0]) / 2;
    const startOffsetY = -(max[1] - min[1]) / 2;

    this.startOffset = { x: startOffsetX, y: startOffsetY };
  },
  methods: {
    calculate(flightData, nextFlightData) {
      const { direction, speed, timestamp } = flightData;
      const { timestamp: nextTimestamp } = nextFlightData;
      const distance = 1.8 * (speed / 3600) * (nextTimestamp - timestamp);

      const rotation = direction;
      const x = distance * Math.sin(rotation * (Math.PI / 180));
      const y = distance * Math.cos(rotation * (Math.PI / 180));

      return { x, y, rotation };
    },
    animateFlight() {
      for (let i = 0; i < this.flightData.length - 1; i++) {
        const { x, y, rotation } = this.calculate(flightData[i], flightData[i + 1],);
        this.x += x;
        this.y -= y;

        this.timeline.add({
          targets: this.plane,
          translateX: this.x,
          translateY: this.y,
          rotate: [rotation, rotation],
          //begin: () => { console.log({ rotation, x, y }); },
          easing: 'linear',
        });
      }

      this.timeline.play();
    },
    reset() {
      if (this.timeline) {
        this.timeline.pause();
        this.timeline.seek(0);
      }
    },
    handleClick() {
      this.isActive = !this.isActive;
      this.isActive ? this.animateFlight() : this.reset();
    },
  }
};
</script>

<style scoped>
.main {
  @apply flex flex-col relative w-full h-full bg-[url('/public/map.png')] bg-center bg-[length:3840px_2400px];
}

.main__plane {
  @apply absolute w-10 h-10 inset-1/2 place-self-center;
}

.main__button {
  @apply absolute self-center bottom-32 w-44 py-2 bg-yellow-500 rounded font-bold hover:bg-yellow-600;
}

.main__button_active {
  @apply absolute self-center bottom-32 w-44 py-2 bg-gray-800 text-white rounded font-bold hover:bg-gray-900;
}
</style>

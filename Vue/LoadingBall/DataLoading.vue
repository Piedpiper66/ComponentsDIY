<template>
  <div class="flex justify-center">
    <div
      class="loader"
      :class="{ show }"
      :style="{ '--ball-color': color, '--ball-size': size }"
    >
      <div class="circle"></div>
      <div class="circle"></div>
      <div class="circle"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Loading",
  props: {
    show: Boolean,
    color: {
      type: String,
      default: "cornflowerblue",
    },
    size: {
      type: String,
      default: "10px",
    },
  },
};
</script>

<style scoped>
.loader {
  opacity: 0;
  display: flex;
  /* position: fixed;
  bottom: 30px;
  transition: all 0.3s ease-in;
  left: 50%;
  transform: translateX(-50%); */
  /* --ball-color: cornflowerblue; */
}

.loader.show {
  opacity: 1;
}

.circle {
  background-color: var(--ball-color);
  width: var(--ball-size);
  height: var(--ball-size);
  border-radius: 50%;
  margin: 5px;
  animation: myBounce 0.5s ease-in-out infinite;
}

.circle:nth-of-type(2) {
  animation-delay: 0.1s;
}

.circle:nth-of-type(3) {
  animation-delay: 0.2s;
}

@keyframes myBounce {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}
</style>

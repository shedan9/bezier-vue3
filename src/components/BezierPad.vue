<template>
  <svg
    version="1.1"
    baseProfile="full"
    :width="width"
    :height="height"
    xmlns="http://www.w3.org/2000/svg"
    ref="svg"
  >
    <rect width="100%" height="100%" fill="transparent" @click="dotAdd" />

    <line
      v-for="(line, index) in lines"
      :key="index"
      :x1="line.x1"
      :y1="line.y1"
      :x2="line.x2"
      :y2="line.y2"
      stroke="#ffdfc2"
    />

    <g
      v-for="(dot, index) in dots"
      :key="index"
      cursor="pointer"
      @click="handleDotDelete(index)"
    >
      <circle
        :cx="dot[0]"
        :cy="dot[1]"
        r="6"
        fill="#fff"
        stroke="#dba058"
      />
      <text :x="dot[0] - 3" :y="dot[1] + 3" font-size="10">{{index + 1}}</text>
    </g>

    <line
      v-show="isDraw"
      v-for="(tangent, index) in tangents"
      :key="index"
      :x1="tangent.x1"
      :y1="tangent.y1"
      :x2="tangent.x2"
      :y2="tangent.y2"
      stroke="#58addb"
    />

    <path
      v-show="t > 0"
      ref="path"
      :d="bezierPath"
      stroke="#db585c"
      fill="transparent"
      :stroke-dasharray="dashArrayStr"
      :stroke-dashoffset="dashOffset"
    />

    <circle
      v-if="isDraw && bezierDot"
      :cx="bezierDot[0]"
      :cy="bezierDot[1]"
      r="4"
      fill="#db585c"
    />

  </svg>
</template>

<script>

  import { ref, watch, computed, nextTick } from 'vue'

  function drawLine(refT, refDashOffset, pathLength) {
    let fn = (resolve) => {
      const tStep = 0.002;
      refT.value += tStep;
      refDashOffset.value -= pathLength * tStep;
      if (refT.value <= 1) {
        setTimeout(() => fn(resolve), 17);
      } else {
        resolve(true);
      }
    };
    return new Promise((resolve) => {
      fn(resolve);
    });

  }

  function computedDotPosition(x, y, el) {
    return [x - el.offsetLeft, y - el.offsetTop];
  }

  function getDotByT(line, t) {
    const x1 = line.x1, y1 = line.y1, x2 = line.x2, y2 = line.y2;
    const dx = Math.abs(x1 - x2), dy = Math.abs(y1 - y2);
    let x = 0, y = 0;

    if (x1 > x2) {
      x = x1 - dx * t;
    } else {
      x = x1 + dx * t;
    }

    if (y1 > y2) {
      y = y1 - dy * t;
    } else {
      y = y1 + dy * t;
    }

    return [x, y];
  }

  function getTangents(lines, t) {
    const dots = lines.map(line => getDotByT(line, t));

    const tangents = [];
    const len = dots.length - 1;
    for (let i = 0; i < len; i ++) {
      tangents.push({
        x1: dots[i][0],
        y1: dots[i][1],
        x2: dots[i + 1][0],
        y2: dots[i + 1][1],
      });
    }

    if (tangents.length > 1) {
      return tangents.concat(getTangents(tangents, t));
    } else {
      return tangents;
    }
  }

  export default {
    setup() {

      const width = 1000;
      const height = 600;

      const svg = ref(null);
      const path = ref(null);
      const t = ref(0);
      const isDraw = ref(false);

      const dots = ref([]);

      const pathLength = ref(0);
      const dashOffset = ref(0);

      const dashArrayStr = computed(() => `${pathLength.value} ${pathLength.value}`);

      const lines = computed(() => {
        const ret = [];
        const len = dots.value.length - 1;
        for (let i = 0; i < len; i ++) {
          ret.push({
            x1: dots.value[i][0],
            y1: dots.value[i][1],
            x2: dots.value[i + 1][0],
            y2: dots.value[i + 1][1],
          });
        }

        return ret;
      });

      const tangents = computed(() => {
        return getTangents(lines.value, t.value);
      });

      const bezierDot = computed(() => {
        const len = tangents.value.length;
        return len > 0 ? getDotByT(tangents.value[len - 1], t.value) : null;
      });

      const bezierPath = computed(() => {
        if (!dots.value.length) return '';
        const start = `M${dots.value[0][0]} ${dots.value[0][1]}`;
        let curve = '';
        for (let i = 0; i < dots.value.length - 1; i++) {
          curve += ` ${dots.value[i + 1][0]} ${dots.value[i + 1][1]}`;
        }
        if (dots.value.length === 3) {
          return `${start} Q${curve}`;
        } else if (dots.value.length === 4) {
          return `${start} C${curve}`;
        } else {
          return '';
        }
      });

      const dotAdd = (e) => {
        if (isDraw.value || dots.value.length >= 4) return;
        dots.value.push(computedDotPosition(e.pageX, e.pageY, svg.value.parentNode));
      };

      const handleDotDelete = index => {
        dots.value.splice(index, 1);
      };

      const curveDraw = () => {
        if (isDraw.value) return;
        isDraw.value = true;
        t.value = 0;
        dashOffset.value = pathLength.value;
        drawLine(t, dashOffset, pathLength.value).then(() => {
          isDraw.value = false;
        });
      };

      const clear = () => {
        dots.value = [];
        t.value = 0;
      };

      watch(() => dots.value.length, async () => {
        await nextTick();
        dashOffset.value = pathLength.value = path.value.getTotalLength();
      });

      return {
        t,
        width,
        height,
        dotAdd,
        handleDotDelete,
        curveDraw,
        clear,
        isDraw,
        svg,
        path,
        dots,
        lines,
        tangents,
        bezierDot,
        bezierPath,
        dashOffset,
        dashArrayStr,
      };
    },
  }




</script>

<style scoped>

</style>

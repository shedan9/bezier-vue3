<template>
  <h1 class="title">贝塞尔曲线</h1>
  <main class="content">
    <BezierPad ref="bezierPad" />
  </main>
  <div class="btn-group">
    <button class="btn" @click="curveDraw">绘制</button>
    <button class="btn grey" @click="clear">清空</button>
  </div>
</template>

<script>
  import BezierPad from './components/BezierPad.vue';
  import { ref } from 'vue';

  export default {
    components: {
      BezierPad,
    },
    setup() {

      const bezierPad = ref(null);

      const curveDraw = () => {
        bezierPad.value.curveDraw();
      };

      const clear = () => {
        bezierPad.value.clear();
      };

      return {
        bezierPad,
        curveDraw,
        clear,
      };
    },
  };

</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .btn-group {
    display: flex;
  }
  .btn {
    width: 100px;
    height: 32px;
    border-radius: 4px;
    background: #5aa5fa;
    color: #fff;
    border: none;
    cursor: pointer;
    transition: background-color .3s ease;
    line-height: 32px;
    margin-top: 30px;
  }
  .btn:hover {
    background: #83baf7;
  }

  .btn.grey {
    background: #bfbfbf;
    margin-left: 20px;
  }
  .btn.grey:hover {
    background: #d4d4d4;
  }
  .content {
    height: 600px;
    border-right: 1px solid #E4E7ED;
    border-bottom: 1px solid #E4E7ED;
    background-image: linear-gradient(#E4E7ED 1px, transparent 0),
    linear-gradient(90deg, #E4E7ED 1px, transparent 0);
    background-size: 20px 20px;
  }
</style>

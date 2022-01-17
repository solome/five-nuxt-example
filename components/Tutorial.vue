<!-- Please remove this file from your project -->
<template>
  <div class="realsee-app">
    <nav class="navbar" v-if="fiveModelReady"></nav>
    <div class="five-container" ref="five"></div>
    <div class="five-opts">
      <button v-on:click="loadFive">载入数据</button>
      <button v-on:click="changeFiveMode('Panorama')" v-show="fiveModelReady">切至全景</button>
      <button v-on:click="changeFiveMode('Floorplan')" v-show="fiveModelReady">切至模型</button>
      <button v-on:click="changeFiveMode('Topview')" v-show="fiveModelReady">切至户型</button>
    </div>
    <div v-bind:class="{'five-plugins-PanoFloorplanRadarPlugin--enable': floorpalnRadarEnable}" class="five-plugins-PanoFloorplanRadarPlugin" v-on:click="changeFiveMode('Topview')" ref="PanoFloorplanRadarPlugin"></div>
    <div class="five-plugins-TopviewFloorplanPlugin" ref="TopviewFloorplanPlugin"></div>
  </div>
</template>

<script>
import {Five, parseWork} from '@realsee/five'
import PanoFloorplanRadarPlugin from '@realsee/dnalogel/plugins/floorplan/PanoFloorplanRadarPlugin'
import TopviewFloorplanPlugin from '@realsee/dnalogel/plugins/floorplan/TopviewFloorplanPlugin'

import {data} from './data'

export default {
  name: 'NuxtTutorial',
  data () {
    return {
      // 模型是否就绪
      fiveModelReady: false,
      // 是否展示雷达图
      floorpalnRadarEnable: false,
      five: null,
    }
  },
  mounted () {
    this.$nextTick(() => {
      // 初始化 Five 实例
      this.five = new Five({
        plugins: [
          [
            PanoFloorplanRadarPlugin,
            'floorpalnRadar',
            {}
          ],
          [
            TopviewFloorplanPlugin,
            'topviewFloorplan',
            {}
          ],
        ]
      })

      // 事件：模型加载完成
      this.five.on('modelLoaded', () => {
        // 判断是否出现 户型雷达图
        this.floorpalnRadarEnable = this.five.state.mode === Five.Mode.Panorama
        // 模型 就绪，展示相关按钮
        this.fiveModelReady = true
      })

      // 状态发生改变
      this.five.on('modeChange', (mode) => {
        // 只有在全景的模式才展示 户型雷达图
        this.floorpalnRadarEnable = mode === Five.Mode.Panorama
      })
    })
  },

  methods: {
    // 加载 Five 
    loadFive() {
      // 此处的数据 可以 调整为 异步请求获取
      const work = parseWork(data.datas.vr)
      const container = this.$refs.five
      this.five.appendTo(container, {
        width: container.offsetWidth,
        height: container.offetHeight,
      })

      // VR 
      this.five.load(work)

      const floorpalnRadar = this.five.plugins.floorpalnRadar
      const topviewFloorplan = this.five.plugins.topviewFloorplan

      // 相关插件
      floorpalnRadar.appendTo(this.$refs.PanoFloorplanRadarPlugin)
      topviewFloorplan.appendTo(this.$refs.TopviewFloorplanPlugin)

      // 载入数据
      const floorplanData = data.datas.floor
      floorpalnRadar.load(floorplanData)
      topviewFloorplan.load(floorplanData)
    },

    // 更改 Five 状态
    changeFiveMode(mode) {
      if (!this.five) return
      this.five.setState({ mode })
    }
  }
}
</script>

<style scoped>
  .five-container {
    background: #00162a;
  }

  .five-opts {
    position: absolute;
    left: 0;
    top: 0;
    display: flex;
    flex-direction: column;
    pointer-events: none;
    padding: 36px;
  }

  .five-opts button {
    pointer-events: all;
    cursor: pointer;
    background: darkblue;
    border: none;
    padding: 5px 10px;
    margin-bottom: 15px;
    color: white;
    border-radius: 4px;
  }

  .five-plugins-PanoFloorplanRadarPlugin {
    width: 150px;
    height: 200px;
    background-color: rgb(0 0 0 / 12%);
    border-radius: 6px;
    position: absolute;
    right: 36px;
    top: 36px;
    pointer-events: none;
    opacity: 0;
  }

  .five-plugins-PanoFloorplanRadarPlugin--enable {
    opacity: 1;
    transition: opacity .45s;
    pointer-events: auto;

  }
</style>
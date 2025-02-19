<template>
  <div
    class="node-children"
    :class="[isTop ? 'isTop' : '', layoutClass]"
    :ref="el => setWrapperRef(el)"
    :style="{ zoom: isTop && zoom, transform: isTop && transformStyle }"
  >
    <div class="node__wrapper">
      <div class="node__header">
        <div class="node-name" :title="orgData.name">{{ orgData.name }}</div>
        <div class="node-type" :title="orgData.type" v-if="orgData.type">{{ orgData.type }}</div>
      </div>
      <div class="node__info">
        <div class="info-item">部门编码：{{ orgData.code }}</div>
        <div class="info-item">负责人：{{ orgData.manager }}</div>
        <div class="info-item">成员数：{{ orgData.members || 0 }}</div>
      </div>
      <template v-if="orgData.children && orgData.children.length">
        <div v-if="isVertical" class="line-v__bottom"></div>
        <div v-else class="line-h__right"></div>
      </template>
    </div>
    <div v-if="orgData.children && orgData.children.length" class="node-children">
      <div class="children-wrapper" :class="[layoutClass]">
        <div
          class="child-node"
          :class="{ isVertical, isHorizontal: !isVertical}"
          v-for="(child, index) in orgData.children"
          :key="index"
        >            
          <div :class="[isVertical ? 'line-v__top' : 'line-h__left']"></div>
          <org-chart :org-data="child" :layout="layout" :is-top="false"></org-chart>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import html2canvas from 'html2canvas'

export default {
  name: 'OrgChart',
  props: {
    orgData: Object,
    layout: String,
    isTop: {
      type: Boolean,
      default: true
    }
  },
  mounted() {
    this.centerlize()
  },
  data() {
    return {
      wrapperRef: null,
      // 缩放等级
      zoom: 1,
      transform: {
        translateX: 0,
        translateY: 0
      }
    }
  },
  computed: {
    isVertical() {
      return this.layout === 'vertical'
    },
    transformStyle() {
      const { translateX, translateY } = this.transform
      return `translate(${translateX}, ${translateY})`
    },
    layoutClass() {
      if (this.isVertical) return 'isVertical'
      return 'isHorizontal'
    }
  },
  watch: {
    layout() {
      this.transform.translateX = 0
      this.transform.translateY = 0
      this.$nextTick(this.centerlize)
    },
    zoom() {
      this.$emit('zoom', this.zoom)
    }
  },
  methods: {
    setWrapperRef(el) {
      if (this.isTop) {
        this.wrapperRef = el
      } 
    },
    // 放大
    zoomUp() {
      this.zoom = +(this.zoom + 0.1).toFixed(1)

      this.$nextTick(this.centerlize)
    },
    // 缩小
    zoomDown() {
      if (this.zoom <= 0.1) return

      // 始终保留一位小数 避免小数出现精度问题
      this.zoom = +(this.zoom - 0.1).toFixed(1)

      this.$nextTick(this.centerlize)
    },
    // 展示完整的组织架构图
    showFull() {
      // 根据自身的实际尺寸和父元素的尺寸进行缩放
      const wrapper = this.wrapperRef
      const { width, height } = wrapper.getBoundingClientRect()
      const { width: _width, height: _height } = wrapper.parentNode.getBoundingClientRect()
      
      const zoomX = _width / width
      const zoomY = _height / height
      this.zoom = +((Math.min(zoomX, zoomY) * this.zoom).toFixed(4))

      this.$nextTick(this.centerlize)
    },
    // 下载
    download() {
      const target = this.wrapperRef
      let cloned = target.cloneNode(true)
      cloned.style.zoom = 1
      cloned.style.position = 'fixed'
      cloned.style.top = '-9999px'
      cloned.style.left = '-9999px'
      document.body.appendChild(cloned)
      const { width, height } = cloned.getBoundingClientRect()
      html2canvas(cloned, {
        width, height,
        scale: 2,
        backgroundColor: '#f6f6f6'
      }).then(canvas => {
        this.exportImage(canvas.toDataURL())
        cloned.remove()
        cloned = null
      })
    },
    // 工具函数：导出图片
    exportImage(base64) {
      const link = document.createElement('a')
      link.href = base64
      link.download = 'chart.png'
      document.body.appendChild(link)
      link.click()
      link.remove()
    },
    // 计算自身的尺寸与父元素的尺寸，居中显示内容
    centerlize() {
      if (!this.isTop) return

      const nodeWrapper = this.wrapperRef
      const parentNode = nodeWrapper.parentNode
      const { width, height } = nodeWrapper.getBoundingClientRect()
      const { width: width1, height: height1 } = parentNode.getBoundingClientRect()
      if (width < width1) {
        this.transform.translateX = `${(width1 - width) / 2 / this.zoom}px`
      }
      if (height < height1) {
        this.transform.translateY = `${(height1 - height) / 2 / this.zoom}px`
      }
    }
  }
}
</script>

<style lang="scss" scoped>
$lineColor: #ddd;
$gap: 20px;
$width: 200px;

.node-children {
  position: relative;
  width: fit-content;
  box-sizing: border-box;
  display: flex;
  align-items: center;

  &.isTop {
    padding: 10px;
  }

  &.isHorizontal {
    flex-direction: row;

    .node__wrapper {
      margin-right: $gap * 2;
    }
  }

  &.isVertical {
    flex-direction: column;

    .node__wrapper {
      margin-bottom: $gap * 2;
    }
  }

  .children-line {
    position: absolute;
    background-color: $lineColor;
  }

  .node__wrapper {
    position: relative;
    min-width: 180px;
    width: 180px;
    background: #fff;
    box-shadow: 2px 2px 3px rgba(0, 0, 0, 0.1);
    border-radius: 5px;
    cursor: pointer;

    .node__header {
      padding: 5px 8px 5px 12px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      border-radius: 5px 5px 0 0;
      background: #07845A;
      color: #fff;

      .node-name {
        white-space: nowrap;
        text-overflow: ellipsis;
        overflow: hidden;
        padding-right: 4px;
      }

      .node-type {
        padding: 2px 5px;
        white-space: nowrap;
        background-color: #57B191;
        border-radius: 5px;
        font-size: 14px;
      }
    }

    .node__info {
      padding: 5px 12px;
      font-size: 14px;
      .info-item {
        text-align: left;
        margin-bottom: 5px;
        white-space: nowrap;
        text-overflow: ellipsis;
        overflow: hidden;

        &:last-of-type {
          margin-bottom: 0;
        }
      }
    }

    .line-v__bottom {
      position: absolute;
      width: 2px;
      height: $gap;
      bottom: 0;
      transform: translateY(100%);
      left: 50%;
      background-color: $lineColor;
    }

    .line-h__right {
      position: absolute;
      width: $gap;
      height: 2px;
      right: 0;
      transform: translateX(100%);
      top: 50%;
      background-color: $lineColor;
    }
  }

  .children-wrapper {
    display: flex;
    align-items: flex-start;

    &.isVertical {
      flex-direction: row;
    }

    &.isHorizontal {
      flex-direction: column;
    }

    .child-node {
      position: relative;

      &.isVertical {
        padding-left: $gap;
        padding-right: $gap;

        &:first-of-type {
          padding-left: 0;
        }

        &:last-of-type {
          padding-right: 0;
        }

        &::before {
          content: '';
          position: absolute;
          left: 0;
          top: -$gap;
          width: 50%;
          height: 2px;
          background-color: $lineColor;
        }

        &::after {
          content: '';
          position: absolute;
          right: 0;
          top: -$gap;
          width: 50%;
          height: 2px;
          background-color: $lineColor;
        }
      }

      &.isHorizontal {
        padding-top: $gap;
        padding-bottom: $gap;

        &:first-of-type {
          padding-top: 0;
        }

        &:last-of-type {
          padding-bottom: 0;
        }

        &::before {
          content: '';
          position: absolute;
          top: 0;
          left: -$gap;
          height: 50%;
          width: 2px;
          background-color: $lineColor;
        }

        &::after {
          content: '';
          position: absolute;
          bottom: 0;
          left: -$gap;
          height: 50%;
          width: 2px;
          background-color: $lineColor;
        }
      }

      &:nth-of-type(1) {
        &::before {
          background-color: transparent;
        }
      }

      &:last-of-type {
        &::after {
          background-color: transparent;
        }
      }

      .line-v__top {
        position: absolute;
        width: 2px;
        height: $gap;
        top: 0;
        transform: translateY(-100%);
        left: 50%;
        background-color: $lineColor;
      }

      .line-h__left {
        position: absolute;
        width: $gap;
        height: 2px;
        left: 0;
        transform: translateX(-100%);
        top: 50%;
        background-color: $lineColor;
      }
    }
  }
}
</style>
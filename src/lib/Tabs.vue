<template>
  <div class="pokemon-tabs">
    <div class="pokemon-tabs-nav" ref="container">
      <div class="pokemon-tabs-nav-item" @click="select(t)" :class="{selected:t===selected}" v-for="(t,index) in titles"
           :ref="el=>{if(t===selected)selectedItem=el}"
           :key="index">
        {{ t }}
      </div>
      <div class="pokemon-tabs-nav-indicator" ref="indicator"></div>
    </div>
    <div class="pokemon-tabs-content">
      <component class="pokemon-tabs-content-item" :class="{selected:c.props.title ===selected}" v-for="c in defaults"
                 :is="c"/>
    </div>
  </div>
</template>

<script lang="ts">
import Tab from './Tab.vue'
import {computed, ref, onMounted, onUpdated} from "vue";

export default {
  props: {
    selected: {
      type: String,
    }
  },
  setup(props, context) {
    const selectedItem = ref<HTMLDivElement>(null)
    const indicator = ref<HTMLDivElement>(null)
    const container = ref<HTMLDivElement>(null)
    const x = () => {
      const {width} = selectedItem.value.getBoundingClientRect()//获取宽度
      indicator.value.style.width = width + 'px'
      const {left: left1} = container.value.getBoundingClientRect()//拿到容器container的左坐标
      const {left: left2} = selectedItem.value.getBoundingClientRect()
      const left = left2 - left1
      indicator.value.style.left = left + 'px'
    }
    onMounted(x)////只在第一次渲染执行
    onUpdated(x)
    const defaults = context.slots.default()
    defaults.forEach((tag) => {
      //@ts-ignore
      if (tag.type.name !== Tab.name) {
        throw new Error('Tabs 子标签必须是 Tab')
      }
    })
    const current = computed(() => {
      return defaults.filter((tag) => {
        return tag.props.title === props.selected
      })[0]
    })
    const titles = defaults.map((tag) => {
      return tag.props.title
    })
    const select = (title: string) => {
      context.emit('update:selected', title)
    }
    return {
      defaults,
      titles,
      current,
      select,
      selectedItem,
      indicator,
      container
    }
  }
}
</script>

<style lang="scss">
$blue: #40a9ff;
$color: #333;
$border-color: #d9d9d9;
.pokemon-tabs {
  &-nav {
    display: flex;
    color: $color;
    position: relative;
    border-bottom: 1px solid $border-color;


    &-item {
      padding: 8px 0;
      margin: 0 16px;
      cursor: pointer;

      &:first-child {
        margin-left: 0;
      }

      &.selected {
        color: $blue;
      }
    }

    &-indicator {
      position: absolute;
      height: 3px;
      background: $blue;
      left: 0;
      bottom: -1px;
      width: 100px;
      transition: all 250ms
    }
  }

  &-content {
    padding: 8px 0;

    &-item {
      display: none;

      &.selected {
        display: block;
      }
    }
  }
}
</style>
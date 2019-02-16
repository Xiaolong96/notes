## Tab 标签页组件

### 基础用法

默认情况下启用第一个标签，可以通过`v-model`绑定当前激活的标签索引

```js
<tabs v-model="active">
  <tab title="标签 1">内容 1</tab>
  <tab title="标签 2">内容 2</tab>
  <tab title="标签 3">内容 3</tab>
</tabs>
export default {
  data() {
    return {
      active: 2
    };
  }
}
```

### 点击事件

可以在tabs上绑定click事件和change事件，事件传参为标签对应的索引和标题

```js
<tabs @click="handleClick"></tabs>
<tabs @change="handleChange"></tabs>
```

### Tabs API

| 属性名      | 类型   | 默认值 | 说明 |
| ----------- | ------ | ------ | ---- |
| v-model     | String Number | 0      | 当前标签的索引 |
| color | String | red      | 标签和底部条颜色 |
| duration      | Number | 0.3      | 动画时间，单位秒 |
| line-width | Number | tab.offsetWidth / 2      | 底部条宽度，单位 px |
| line-height | Number | 3      | 底部条高度，单位 px |

### Tab API
| 属性名      | 类型   | 默认值 | 说明 |
| ----------- | ------ | ------ | ---- |
| title     | String | -      | tab页的标题 |

### code

#### tab

```js
<template>
  <div
    v-show="isSelected"
    class="tab__pane"
  >
    <slot />
  </div>
</template>

<script>
export default {
  name: 'Tab',
  props: {
    title: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      parent: null,
    };
  },
  computed: {
    index() {
      return this.parent.tabs.indexOf(this);
    },
    isSelected() {
      return this.index === this.parent.curActive;
    },
  },
  watch: {
    // title() {
    //   this.parent.setLine();
    // }
  },
  created() {
    this.findParent('Tabs');
  },
  mounted() {
    const { tabs } = this.parent;
    const index = this.parent.$slots.default.indexOf(this.$vnode);
    tabs.splice(index === -1 ? tabs.length : index, 0, this);
  },
  beforeDestroy() {
    this.parent.tabs.splice(this.index, 1);
  },
  methods: {
    findParent(name) {
      let parent = this.$parent;
      while (parent) {
        if (parent.$options.name === name) {
          this.parent = parent;
          break;
        }
        parent = parent.$parent; // 多层嵌套
      }
    },
  },
};
</script>

<style scoped>
/* .tab__pane {

} */
</style>


```

#### tabs

```js
<template>
  <div class="tabs">
    <div class="tabs__nav">
      <div
        :style="lineStyle"
        class="tabs__line"
      />
      <div
        v-for="(tab, index) in tabs"
        ref="tabs"
        :key="index"
        :style="getTabStyle(tab, index)"
        :class="{'tab--active': index === curActive,}"
        class="tab"
        @click="onClick(index)"
      >
        <span>{{ tab.title }}</span>
      </div>
    </div>
    <slot/>
  </div>
</template>

<script>
export default {
  name: 'Tabs',
  model: {
    prop: 'active',
  },
  props: {
    color: {
      type: String,
      default: 'red',
    },
    lineWidth: {
      type: Number,
      default: null,
    },
    lineHeight: {
      type: Number,
      default: 3,
    },
    duration: {
      type: Number,
      default: 0.3,
    },
    active: {
      type: [Number, String],
      default: 0,
    },
  },
  data() {
    return {
      tabs: [],
      curActive: null,
      lineStyle: {
        backgroundColor: this.color,
      },
    };
  },
  computed: {},
  watch: {
    active(val) {
      if (val !== this.curActive) {
        this.correctActive(val);
      }
    },
    color() {
      this.setLine();
    },
    tabs() {
      this.correctActive(this.curActive || this.active);
      this.setLine();
    },
    curActive() {
      this.setLine();
    },
  },
  mounted() {
    this.correctActive(this.active);
    this.isFirstLoaded = true;
    this.$nextTick(() => {
      this.isFirstLoaded = false;
    });
  },
  methods: {
    onClick(index) {
      const { title } = this.tabs[index];
      this.setCurActive(index);
      this.$emit('click', index, title);
    },
    setActive() {},
    correctActive(active) {
      // eslint-disable-next-line
      active = +active;
      // console.log(active, 'active');
      const exist = this.tabs.some(tab => tab.index === active);
      const defaultActive = (this.tabs[0] || {}).index || 0;
      this.setCurActive(exist ? active : defaultActive);
    },
    setCurActive(active) {
      if (active !== this.curActive) {
        // console.log(active, 'active');
        if (this.curActive !== null) {
          this.$emit('change', active, this.tabs[active].title);
        }
        this.curActive = active;
      }
    },
    setLine() {
      const animation = this.isFirstLoaded;
      this.$nextTick(() => {
        const { tabs } = this.$refs;
        if (!tabs || !tabs[this.curActive]) {
          return;
        }
        const tab = tabs[this.curActive];
        const { lineWidth, lineHeight } = this;
        /* eslint-disable no-unneeded-ternary */
        const width = lineWidth ? lineWidth : (tab.offsetWidth / 2);
        /* eslint-disable no-unneeded-ternary */
        const left = tab.offsetLeft + ((tab.offsetWidth - width) / 2);
        const lineStyle = {
          width: `${width}px`,
          backgroundColor: this.color,
          transform: `translateX(${left}px)`,
        };
        if (!animation) {
          lineStyle.transitionDuration = `${this.duration}s`;
        }
        let height = '';
        if (lineHeight) {
          height = `${lineHeight}px`;
        } else {
          height = '3px';
        }
        lineStyle.height = height;
        lineStyle.borderRadius = height;
        this.lineStyle = lineStyle;
      });
    },
    getTabStyle(tab, index) {
      if (index === this.curActive) {
        return {
          color: this.color,
          fontWeight: 'bold',
        };
      }
    },
  },
};
</script>
<style lang="scss" scoped>
.tab {
  flex: 1;
  cursor: pointer;
  min-width: 0;
  padding: 0 5px;
  font-size: 14px;
  position: relative;
  color: #000;
  line-height: 50px;
  text-align: center;
  box-sizing: border-box;
  background-color: #fff;
  span {
    display: block;
  }
  &--active {
    font-weight: 500!important;
  }
}
.tabs {
  position: relative;
  &__nav {
    display: flex;
    user-select: none;
    position: relative;
    background-color: #fff;
    height: 100%;
    box-sizing: content-box;
  }
  &__line {
    z-index: 1;
    left: 0;
    bottom: 0px;
    height: 3px;
    position: absolute;
    border-radius: 3px;
    background-color: red;
  }
}
</style>
```

END
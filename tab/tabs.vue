<template>
  <div class="tabs">
    <div class="tabs__nav">
      <div
       class="tabs__line"
       :style="lineStyle"
      ></div>
      <div 
        v-for="(tab, index) in tabs" 
        :key="index" 
        ref="tabs"
        @click="onClick(index)"
        class="tab"
        :style="getTabStyle(tab, index)"
        :class="{'tab--active': index === curActive,}"
      >
        <span>{{tab.title}}</span>
      </div>
    </div>
    <slot/>
  </div>
</template>

<script>
export default {
  name: 'tabs',
  model: {
    prop: 'active',
  },
  props: {
    color: String,
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
      active = +active;
      // console.log(active, 'active');
      const exist = this.tabs.some(tab => tab.index === active);
      const defaultActive = (this.tabs[0] || {}).index || 0;
      this.setCurActive(exist ? active : defaultActive);
    },
    setCurActive(active) {
      if (active !== this.curActive) {
        console.log(this.curActive);
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
<style>
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

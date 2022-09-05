<template>
  <div
    class="sidebar-logo-container"
    :class="{'collapse':collapse}"
    :style="{ background:formatThemeClass('BackgroundColor','logo')}"
  >
    <!-- sideTheme === 'theme-dark' ? variables.menuBackground : variables.menuLightBackground -->
    <transition name="sidebarLogoFade">
      <router-link
        v-if="collapse"
        key="collapse"
        class="sidebar-logo-link collapse"
        to="/"
      >
        <img
          v-if="logo"
          :src="logo"
          class="sidebar-logo"
        >
        <!-- <h1
          v-else
          class="sidebar-title"
          :style="{ color:formatThemeClass('background','logo') }"
        >{{ title }} </h1> -->
        <!-- sideTheme === 'theme-dark' ? variables.logoTitleColor : variables.logoLightTitleColor -->
      </router-link>
      <router-link
        v-else
        key="expand"
        class="sidebar-logo-link"
        to="/"
      >
        <img
          v-if="logo"
          :src="logo"
          class="sidebar-logo"
        >
        <!-- <h1
          class="sidebar-title"
          :style="{ color: formatThemeClass('TitleColor','logo')}"
        >{{ title }} </h1> -->
        <!-- sideTheme === 'theme-dark' ? variables.logoTitleColor : variables.logoLightTitleColor  -->
      </router-link>
    </transition>
  </div>
</template>

<script>
import logoImg from '@/assets/logo/logo.png'
import variables from '@/assets/styles/variables.scss'

export default {
  name: 'SidebarLogo',
  props: {
    collapse: {
      type: Boolean,
      required: true
    }
  },
  data() {
    return {
      title: 'TISC运维系统',
      logo: logoImg
    }
  },
  computed: {
    variables() {
      return variables
    },
    sideTheme() {
      return this.$store.state.settings.sideTheme
    }
  },
  methods: {
    formatThemeClass(type, before) {
      const unBefore = before || 'menu'
      let theme = this.sideTheme || 'theme-dark'
      const classType = this.titleCase(type)
      theme = this.titleCase(theme.replace('theme-', ''))
      if (theme === 'Dark') {
        theme = ''
      }
      const scssClass = this.variables[`${unBefore}${theme}${classType}`]
      return scssClass
    },
    titleCase(str) {
      const newStr = str.slice(0, 1).toUpperCase() + str.slice(1)
      return newStr
    }
  }
}
</script>

<style lang="scss" scoped>
.sidebarLogoFade-enter-active {
  transition: opacity 1.5s;
}

.sidebarLogoFade-enter,
.sidebarLogoFade-leave-to {
  opacity: 0;
}

.sidebar-logo-container {
  position: relative;
  width: 100%;

  background: #2b2f3a;
  text-align: center;
  overflow: hidden;

  & .sidebar-logo-link {
    height: 117px;
    line-height: 117px;
    &.collapse {
      height: 50px;
      line-height: 50px;
      .sidebar-logo {
        width: 32px;
        height: 32px;
        vertical-align: middle;
      }
    }
    & .sidebar-logo {
      width: 106px;
      height: 85px;
      vertical-align: middle;
      margin-right: 12px;
    }

    & .sidebar-title {
      display: inline-block;
      margin: 0;
      color: #fff;
      font-weight: 600;
      line-height: 50px;
      font-size: 14px;
      font-family: Avenir, Helvetica Neue, Arial, Helvetica, sans-serif;
      vertical-align: middle;
    }
  }

  &.collapse {
    .sidebar-logo {
      margin-right: 0px;
    }
  }
}
</style>

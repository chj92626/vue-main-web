<template>
  <div>
    <el-scrollbar wrap-class="scrollbar-wrapper">
      <el-menu
        :default-active="activeMenu"
        :collapse="isCollapse"
        :background-color="variables.menuBg"
        :text-color="variables.menuText"
        :unique-opened="false"
        :active-text-color="variables.menuActiveText"
        :collapse-transition="false"
        mode="vertical"
      >
        <sidebar-item v-for="route in allMenusfull" :key="route.path" :item="route" :base-path="route.path" />
      </el-menu>
    </el-scrollbar>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import { Scrollbar } from 'element-ui'
import SidebarItem from './SidebarItem'
import { splitUrl } from '@/utils/qiankunMethods.js'
import { constantRoutes } from '@/router/index'
import variables from '@/styles/variables.scss'

export default {
  components: { SidebarItem, elScrollbar: Scrollbar },
  data() {
    return {
      allMenusfull: []
    }
  },
  computed: {
    ...mapGetters([
      'sidebar'
    ]),
    routes() {
      return this.$router.options.routes
    },
    activeMenu() {
      const route = this.$route
      const { path } = route
      return path
    },
    variables() {
      return variables
    },
    isCollapse() {
      return !this.sidebar.opened
    }
  },
  created() {
    this.makeAllDynamicMenus()
    this.makeStaticRouter()
  },
  methods: {
    fullMenuInfo(element) {
      element.path = element.id
      element.meta = {
        icon: element.icon || element.privilegeIcon,
        title: element.name
      }
    },
    fullRouternfo(element) {
      const pathSplit = splitUrl(element.url)
      const allPath = pathSplit[1]
      element.path = allPath
      element.meta = {
        title: element.name
      }
    },
    fullFisrtNoChildren(element) {
      const pathSplit = splitUrl(element.url)
      const allPath = pathSplit[1]
      const splitPath = allPath.split('/')
      const firstPath = splitPath[1]
      element.path = `/${firstPath}${element.id}`
      element.meta = {
        title: element.name
      }
      element.children = [
        {
          path: allPath,
          name: element.id,
          meta: { title: element.name, icon: 'form' }
        }
      ]
    },
    makeAllDynamicMenus() {
      const allInfoJson = window.localStorage.getItem('allUserInfo') || '{}'
      const allUserInfo = JSON.parse(allInfoJson)
      const menus = allUserInfo.menus || []
      // 单排
      for (let firstIndex = 0; firstIndex < menus.length; firstIndex++) {
        const firstElement = menus[firstIndex] || {}
        this.fullFisrtNoChildren(firstElement)
      }
      this.allMenusfull = menus
    },
    makeStaticRouter() {
      // copy一下 不能破坏路由原始文件
      const constantRoutesCopy = JSON.parse(JSON.stringify(constantRoutes))
      const menusNoHidden = constantRoutesCopy.filter(item => !item.hidden)
      this.allMenusfull = [...menusNoHidden, ...this.allMenusfull]
      console.log('左侧全量的菜单数据为:', this.allMenusfull)
    }
  }
}
</script>

<template>
  <div :class="wrpCls">
    <avatar-dropdown :menu="showMenu" :current-user="currentUser" :class="prefixCls" @showUp = "updatePassword"/>
    <select-lang :class="prefixCls" />
    <user-password :class="prefixCls" ref="userPassword"></user-password>
  </div>
</template>

<script>
import AvatarDropdown from './AvatarDropdown'
import SelectLang from '@/components/SelectLang'
import store from 'store'
import {USER_INFO } from '@/store/mutation-types'
// import UserPassword from './UserPassword'
const UserPassword = ()=>import("./UserPassword")
export default {
  name: 'RightContent',
  components: {
    AvatarDropdown,
    SelectLang,
    UserPassword
  },
  props: {
    prefixCls: {
      type: String,
      default: 'ant-pro-global-header-index-action'
    },
    isMobile: {
      type: Boolean,
      default: () => false
    },
    topMenu: {
      type: Boolean,
      required: true
    },
    theme: {
      type: String,
      required: true
    }
  },
  data () {
    return {
      showMenu: true,
      currentUser: {}
    }
  },
  computed: {
    wrpCls () {
      return {
        'ant-pro-global-header-index-right': true,
        [`ant-pro-global-header-index-${(this.isMobile || !this.topMenu) ? 'light' : this.theme}`]: true
      }
    }
  },
  mounted () {
    setTimeout(() => {
      var name = store.get(USER_INFO).userName
      this.currentUser = {
        name: name
      }
    }, 3000)
  },
  methods:{
    updatePassword(){
        this.$refs.userPassword.show('王五')
      },
  }
}
</script>

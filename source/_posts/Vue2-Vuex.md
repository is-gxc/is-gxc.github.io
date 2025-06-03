---
title: Vue2-Vuex
date: 2025-06-03 19:44:03
categories: Vue
tags: [前端, Vue2]
---

在开发大型Vue应用时，组件间状态管理会变得复杂。Vuex作为Vue的官方状态管理库，提供了一个集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。对vue应用中多个组件的共享状态进行集中式的管理（读/写），也是一种组件间通信的方式，且适用于任意组件间通信。



# 一、 Vuex核心概念

## 1. State - 单一状态树

Vuex使用单一状态树，用一个对象就包含了全部的应用层级状态。

``` js
const state = {
  count: 0,
  user: {
    name: 'John Doe',
    id: 123
  }
}
```



## 2. Getters - 计算属性

用于从state中派生出一些状态，类似于computed属性。

``` js 
const getters = {
  doubleCount: state => state.count * 2,
  userId: state => state.user.id
}
```



## 3. Mutations - 更改状态

更改Vuex的store中的状态的唯一方法是提交mutation。必须是同步函数。

``` js
const mutations = {
  increment(state) {
    state.count++
  },
  setUser(state, user) {
    state.user = user
  }
}
```

## 4. Actions - 提交Mutations

Action提交的是mutation，而不是直接变更状态。可以包含任意异步操作。

``` js
const actions = {
  incrementAsync({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  },
  fetchUser({ commit }, userId) {
    return api.getUser(userId).then(user => {
      commit('setUser', user)
    })
  }
}
```

## 5. Modules - 模块化

当应用变得复杂时，可以将store分割成模块。

``` js
const userModule = {
  state: () => ({ /* ... */ }),
  mutations: { /* ... */ },
  actions: { /* ... */ },
  getters: { /* ... */ }
}

const store = new Vuex.Store({
  modules: {
    user: userModule
  }
})
```



# 二、 VueX使用示例

## 1. 搭建Vuex环境

1. 创建文件 `src/store/index.js`

``` js
//引入Vue核心库
import Vue from 'vue'
//引入Vuex
import Vuex from 'vuex'
//应用Vuex插件
Vue.use(Vuex)

//准备actions对象——响应组件中用户的动作
const actions = {}
//准备mutations对象——修改state中的数据
const mutations = {}
//准备state对象——保存具体的数据
const state = {}

//创建并暴露store
export default new Vuex.Store({
	actions,
	mutations,
	state
})
```

2. 在`main.js`中创建vm时传入`store`配置项

``` js
......
//引入store
import store from './store'
......

//创建vm
new Vue({
	el:'#app',
	render: h => h(App),
	store
})
```



## 2. 基本使用

1. 初始化数据、配置`actions`、配置`mutations`，操作文件`store.js`

   ``` js
   //引入Vue核心库
   import Vue from 'vue'
   //引入Vuex
   import Vuex from 'vuex'
   //引用Vuex
   Vue.use(Vuex)
   
   const actions = {
       //响应组件中加的动作
   	jia(context,value){
   		// console.log('actions中的jia被调用了',miniStore,value)
   		context.commit('JIA',value)
   	},
   }
   
   const mutations = {
       //执行加
   	JIA(state,value){
   		// console.log('mutations中的JIA被调用了',state,value)
   		state.sum += value
   	}
   }
   
   //初始化数据
   const state = {
      sum:0
   }
   
   //创建并暴露store
   export default new Vuex.Store({
   	actions,
   	mutations,
   	state,
   })
   ```

2. 组件中读取vuex中的数据：`$store.state.sum`

3. 组件中修改vuex中的数据：`$store.dispatch('action中的方法名',数据)` 或 `$store.commit('mutations中的方法名',数据)`



> 备注：若没有网络请求或其他业务逻辑，组件中也可以越过actions，即不写`dispatch`，直接编写`commit`



# 三、 模块化详解

## 1. 为什么需要模块化？

在大型应用中，所有状态集中在一个文件中会导致：

- 状态树过于庞大，难以维护
- 命名冲突风险增加
- 团队协作困难
- 代码复用性差

## 2. 基础模块实现

```js
// user模块
const userModule = {
  namespaced: true, // 启用命名空间
  state: () => ({
    profile: null,
    preferences: {}
  }),
  mutations: {
    SET_PROFILE(state, profile) {
      state.profile = profile;
    },
    UPDATE_PREFERENCE(state, { key, value }) {
      state.preferences = {...state.preferences, [key]: value};
    }
  },
  actions: {
    async fetchUserProfile({ commit }, userId) {
      const profile = await api.getUserProfile(userId);
      commit('SET_PROFILE', profile);
    }
  },
  getters: {
    isPremiumUser: state => {
      return state.profile?.subscription?.type === 'premium';
    }
  }
};

// cart模块
const cartModule = {
  namespaced: true,
  state: () => ({
    items: [],
    checkoutStatus: null
  }),
  mutations: {
    ADD_TO_CART(state, product) {
      const existingItem = state.items.find(item => item.id === product.id);
      if (existingItem) {
        existingItem.quantity++;
      } else {
        state.items.push({ ...product, quantity: 1 });
      }
    },
    SET_CHECKOUT_STATUS(state, status) {
      state.checkoutStatus = status;
    }
  },
  actions: {
    async checkout({ commit, state }) {
      commit('SET_CHECKOUT_STATUS', 'processing');
      try {
        await api.processOrder(state.items);
        commit('SET_CHECKOUT_STATUS', 'success');
        commit('cart/CLEAR_CART', null, { root: true }); // 访问根模块action
      } catch (error) {
        commit('SET_CHECKOUT_STATUS', 'failed');
      }
    }
  },
  getters: {
    cartTotal: state => {
      return state.items.reduce((total, item) => 
        total + (item.price * item.quantity), 0);
    }
  }
};

// 主store
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

export default new Vuex.Store({
  modules: {
    user: userModule,
    cart: cartModule
  },
  state: {
    appLoading: false
  },
  mutations: {
    SET_APP_LOADING(state, isLoading) {
      state.appLoading = isLoading;
    }
  }
});
```

## 3. 命名空间访问规则

| 访问类型 | 非命名空间               | 命名空间                            |
| :------- | :----------------------- | :---------------------------------- |
| State    | `state.moduleName.key`   | `state.moduleName.key`              |
| Getter   | `getters.key`            | `getters['moduleName/key']`         |
| Mutation | `commit('mutationName')` | `commit('moduleName/mutationName')` |
| Action   | `dispatch('actionName')` | `dispatch('moduleName/actionName')` |



# 四、 辅助函数：简化组件与Vuex的交互

## 1. mapState：优雅访问状态

``` js
import { mapState } from 'vuex';

export default {
  computed: {
    // 对象展开运算符混合到computed中
    ...mapState({
      // 访问根状态
      appLoading: state => state.appLoading,
      
      // 访问user模块状态
      userProfile: state => state.user.profile,
      
      // 访问cart模块状态
      cartItems: state => state.cart.items,
      checkoutStatus: state => state.cart.checkoutStatus
    }),
    
    // 数组语法（仅适用于根状态）
    ...mapState(['appLoading']),
    
    // 带命名空间的模块状态
    ...mapState('user', {
      userPreferences: state => state.preferences
    }),
    
    // 模块名称+数组语法
    ...mapState('cart', ['items', 'checkoutStatus'])
  }
}
```

用于帮助我们映射 `state` 中的数据为计算属性，若没有mapState，则访问需要用   this.$store.state.xxxx，使用后，可直接使用计算属性中的key。

mapState前边三个点的作用是：把mapState中的{}中的每一组展开放在computed计算属性中。mapState返回的是一个{}对象，如果想使用，请使用ES6的语法`...`



## 2. mapGetters：简化计算属性

``` js
import { mapGetters } from 'vuex';

export default {
  computed: {
    ...mapGetters({
      // 映射根getter
      isLoading: 'isAppLoading',
      
      // 映射user模块getter
      isPremium: 'user/isPremiumUser',
      
      // 映射cart模块getter
      cartTotal: 'cart/cartTotal'
    }),
    
    // 数组语法（根getter）
    ...mapGetters(['isAppLoading']),
    
    // 带命名空间的模块getter
    ...mapGetters('user', ['isPremiumUser']),
    
    // 重命名模块getter
    ...mapGetters('cart', {
      totalPrice: 'cartTotal'
    })
  }
}
```

用于帮助我们映射 `getter` 中的数据为计算属性

## 3. mapMutations：简化提交变更

``` js
import { mapMutations } from 'vuex';

export default {
  methods: {
    ...mapMutations({
      // 映射根mutation
      setLoading: 'SET_APP_LOADING',
      
      // 映射user模块mutation
      updatePreference: 'user/UPDATE_PREFERENCE'
    }),
    
    // 数组语法（根mutation）
    ...mapMutations(['SET_APP_LOADING']),
    
    // 带命名空间的模块mutation
    ...mapMutations('cart', {
      addToCart: 'ADD_TO_CART'
    }),
    
    // 自定义方法中结合mutation
    updateThemePreference(theme) {
      // 调用映射的方法
      this.updatePreference({
        key: 'theme',
        value: theme
      });
    }
  }
}
```



## 4. mapActions：简化分发操作

``` js
import { mapActions } from 'vuex';

export default {
  methods: {
    ...mapActions({
      // 映射根action
      initializeApp: 'initialize',
      
      // 映射user模块action
      loadUserProfile: 'user/fetchUserProfile',
      
      // 映射cart模块action
      processCheckout: 'cart/checkout'
    }),
    
    // 数组语法（根action）
    ...mapActions(['initialize']),
    
    // 带命名空间的模块action
    ...mapActions('user', ['fetchUserProfile']),
    
    // 自定义方法中组合多个action
    async completeCheckout() {
      this.setAppLoading(true);
      try {
        await this.processCheckout();
        this.showSuccessMessage('Checkout completed!');
      } catch (error) {
        this.showErrorMessage('Checkout failed');
      } finally {
        this.setAppLoading(false);
      }
    }
  }
}
```



# 五、 常见使用误区

## 1. 错误：尝试直接修改模块状态

``` js 
// 错误！
this.$store.state.user.profile = newProfile;

// 正确：通过mutation修改
this.$store.commit('user/SET_PROFILE', newProfile);
```

## 2.  错误：混淆状态路径

``` js
// 假设有命名空间模块：settings/theme
// 错误访问：
this.$store.state.settings.theme.currentTheme;

// 正确访问：
this.$store.state.settings.theme.currentTheme; // 状态访问正确
this.$store.getters['settings/theme/darkModeEnabled']; // getter需要路径
```

## 3. 错误：动态模块的状态访问

``` js 
// 动态注册模块后
store.registerModule('dynamic', dynamicModule);

// 错误：立即访问
console.log(store.state.dynamic); // 可能undefined

// 正确：在nextTick中访问
Vue.nextTick(() => {
  console.log(store.state.dynamic); // 现在可用
});
```


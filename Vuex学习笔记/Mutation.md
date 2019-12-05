# Mutation

mutation 是更改 Vuex 的 store 中状态的唯一方法。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。
mutation 类似于事件：每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是实际进行状态更改的地方，并且它会接受 state 作为第一个参数：

	const store = new Vuex.Store({
		state: {
			count: 1
		},
		mutations: {
			increment (state) {
				// 变更状态
				state.count++
			}
		}
	})

要提交一个 mutation，需要使用 store.commit 方法：

	store.commit('increment')

## 带参数提交 mutation

1. 载荷方式

		// ...
		mutations: {
			increment (state, n) {
				state.count += n
			}
		}

<br/>

		store.commit('increment', 10)

载荷最好是一个对象，这样可以包含多个字段并且会更易读：

		// ...
		mutations: {
			increment (state, payload) {
				state.count += payload.amount
			}
		}
		
<br/>

		store.commit('increment', {
			amount: 10
		})

2. 对象风格的提交方式

		store.commit({
			type: 'increment',
			amount: 10
		})

使用对象风格的提交方式，整个对象都作为载荷传给 mutation 函数，因此 handler 保持不变。

## Mutation 需遵守的规则

###  Vue 的响应规则

Vuex 的 store 中的状态也是响应式的，当我们变更状态时，监视状态的 Vue 组件也会自动更新。这也意味着 Vuex 中的 mutation 也需要与使用 Vue 一样遵守一些注意事项：

1. 最好提前在你的 store 中初始化好所有所需属性。
2. 当需要在对象上添加新属性时，应该：使用 `Vue.set(obj, 'newProp', 123)`, 或者用新对象替换老对象。例如：

		state.obj = { ...state.obj, newProp: 123 }

### Mutation 必须是同步函数

因为如果 mutation 执行了异步函数，当 mutation 触发的时候，异步回调函数还没有被调用，devtools 不知道什么时候异步回调函数实际上被调用，所以异步回调函数中进行的状态的改变就没办法被跟踪。

## 在组件中提交 Mutation

在组件中可以使用 `this.$store.commit('xxx')` 提交 mutation，或者使用 mapMutations 辅助函数将组件中的 methods 映射为 store.commit 调用（需要在根节点注入 store）。

	import { mapMutations } from 'vuex'

	export default {
		// ...
		methods: {
			...mapMutations([
				'increment', // 将 `this.increment()` 映射为 `this.$store.commit('increment')`

				// `mapMutations` 也支持载荷：
				'incrementBy' // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
			]),
			...mapMutations({
				add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
			})
		}
	}

## 使用常量替代 Mutation 事件类型(推荐的最佳实践)

使用常量替代 mutation 事件类型在各种 Flux 实现中是很常见的模式。这样可以使 linter 之类的工具发挥作用，同时把这些常量放在单独的文件中可以让你的开发合作者对整个 app 包含的 mutation 一目了然：

	// mutation-types.js
	export const PRODUCTS = {
		SET_PRODUCTS:'setProducts'
	}

<br>

	// product.js
	import { PRODUCTS } from './mutation-types'

	const state = {
		//...
	}

	// mutations
	const mutations = {
		[PRODUCTS.SET_PRODUCTS] (state, products) {
			// mutate state
		}
	}



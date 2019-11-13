# Module

为了解决应用复杂时，store 对象就可能变得相当臃肿，Vuex 允许将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、以及嵌套子模块（从上至下进行同样方式的分割）。

	const moduleA = {
		state: { ... },
		mutations: { ... },
		actions: { ... },
		getters: { ... }
	}

	const moduleB = {
		state: { ... },
		mutations: { ... },
		actions: { ... }
	}

	const store = new Vuex.Store({
		modules: {
			a: moduleA,
			b: moduleB
		}
	})

	store.state.a // -> moduleA 的所有状态数据
	store.state.b // -> moduleB 的所有状态数据

## 模块的局部状态

对于模块内部的 mutation 和 getter，接收的第一个参数是**模块的局部状态对象**；对于模块内部的 action，局部状态通过 `context.state` 暴露出来，根节点状态则为 `context.rootState`

	const moduleA = {
		state: { count: 0 },
		mutations: {
			increment (state) {
				// 这里的 `state` 对象是模块的局部状态
				state.count++
			}
		},

		getters: {
			doubleCount (state) {
				return state.count * 2
			}
		},

		actions: {
			incrementIfOddOnRootSum ({ state, commit, rootState }) {
				if ((state.count + rootState.count) % 2 === 1) {
					commit('increment')
				}
			}
		}
	}

对于模块内部的 getter，getters和根节点状态分别会作为第二个和第三个参数暴露出来：

	const moduleA = {
		// ...
		getters: {
			sumWithRootCount (state, getters, rootState) {
				return state.count + rootState.count
			}
		}
	}

## 模块的命名空间

默认情况下，模块内部的 action、mutation 和 getter 是注册在全局命名空间的。模块开启命名空间的后，它的所有 getter、action 及 mutation 都会自动根据模块注册的路径调整命名。

例如：

	const store = new Vuex.Store({
		modules: {
			account: {
				namespaced: true,

				// 模块内容（module assets）
				state: { ... }, // 模块内的状态已经是嵌套的了，使用 `namespaced` 属性不会对其产生影响
				getters: {
					isAdmin () { ... } // -> getters['account/isAdmin']
				},
				actions: {
					login () { ... } // -> dispatch('account/login')
				},
				mutations: {
					login () { ... } // -> commit('account/login')
				},

				// 嵌套模块
				modules: {
					// 继承父模块的命名空间
					myPage: {
						state: { ... },
						getters: {
								profile () { ... } // -> getters['account/profile']
						}
					},

					// 进一步嵌套命名空间
					posts: {
						namespaced: true,

						state: { ... },
						getters: {
							popular () { ... } // -> getters['account/posts/popular']
						}
					}
				}
			}
		}
	})

**推荐所有模块开启命名空间（添加 `namespaced: true`），这样可以避免不同模块之间的命名冲突，但是应尽量避免模块嵌套。**

## 带命名空间的辅助函数

当使用 mapState, mapGetters, mapActions 和 mapMutations 这些辅助函数来绑定带命名空间的模块时，需要指定相应模块的空间路径。

1. 分别指定路径

		computed: {
			...mapState({
				a: state => state.moduleA.a,
				b: state => state.moduleA.b
			})
		},
		methods: {
			...mapActions({
				'moduleA/foo', // -> 调用方法：this['moduleA/foo']()
				'moduleA/bar' // -> 调用方法：this['moduleA/bar']()
			})
		}

1. 将模块空间路径字符串作为第一个参数传递给辅助函数

		computed: {
			...mapState('moduleA', {
				a: state => state.a,
				b: state => state.b
			})
		},
		methods: {A
			...mapActions('moduleA', [
				'foo', // -> this.foo(),
				'bar', // -> this.bar()
			])

1. 使用 createNamespacedHelpers 创建基于某个命名空间的辅助函数

		import { createNamespacedHelpers } from 'vuex'

		const { mapState, mapActions } = createNamespacedHelpers('moduleA')

		export default {
			computed: {
				// 在 `moduleA` 中查找
				...mapState({
					a: state => state.a,
					b: state => state.b
				})
			},
			methods: {
				// 在 `moduleA` 中查找
				...mapActions([
					'foo',
					'bar'
				])
			}
		}


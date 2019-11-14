# Module

为了解决应用复杂时 store 对象就可能变得相当臃肿，Vuex 允许将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、以及嵌套子模块（从上至下进行同样方式的分割）。

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
				// 开启命名空间
				namespaced: true,

				// 模块内容（module assets）
				state: { ... }, 
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

**推荐所有模块开启命名空间，这样可以避免不同模块之间的命名冲突，但是应尽量避免模块嵌套。**

### 带命名空间的辅助函数

当使用 mapState, mapGetters, mapActions 和 mapMutations 这些辅助函数来绑定带命名空间的模块时，需要指定相应模块的空间路径。

1. 分别指定路径

		computed: {
			...mapState({
				a: state => state.moduleA.a,
				b: state => state.moduleA.b
			})
		},
		methods: {
			...mapActions([
				'moduleA/foo', // -> 调用方法：this['moduleA/foo']()
				'moduleA/bar' // -> 调用方法：this['moduleA/bar']()
			])
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

### 在带命名空间的模块内访问全局内容

如果要使用全局 state 和 getter，可以在 **getters 和 actions** 中使用。rootState 和 rootGetters 会作为第三和第四参数传入 **getter** ，也会通过 context 对象的属性传入 action。

如果要在全局命名空间内分发 action 或提交 mutation，将 `{ root: true }` 作为第三个参数传给 dispatch 或 commit 即可。

	modules: {
		foo: {
			namespaced: true,

			getters: {
				// 在这个模块的 getter 中，`getters` 被局部化了
				// 你可以使用 getter 的第四个参数来调用 `rootGetters`
				someGetter (state, getters, rootState, rootGetters) {
					getters.someOtherGetter // -> 'foo/someOtherGetter'
					rootGetters.someOtherGetter // -> 'someOtherGetter'
				},
				someOtherGetter: state => { ... }
			},

			actions: {
				// 在这个模块中， dispatch 和 commit 也被局部化了
				// 他们可以接受 `root` 属性以访问根 dispatch 或 commit
				someAction ({ dispatch, commit, getters, rootGetters }) {
					getters.someGetter // -> 'foo/someGetter'
					rootGetters.someGetter // -> 'someGetter'

					dispatch('someOtherAction') // -> 'foo/someOtherAction'
					dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'

					commit('someMutation') // -> 'foo/someMutation'
					commit('someMutation', null, { root: true }) // -> 'someMutation'
				},
				someOtherAction (ctx, payload) { ... }
			}
		}
	}

### 在带命名空间的模块注册全局 action

若需要在带命名空间的模块注册全局 action，你可添加 `root: true`，并将这个 action 的定义放在函数 handler 中。例如：

	{
		actions: {
			someOtherAction ({dispatch}) {
				dispatch('someAction')
			}
		},
		modules: {
			foo: {
				namespaced: true,

				actions: {
					someAction: {
						root: true,
						handler (namespacedContext, payload) { ... } // -> 'someAction'
					}
				}
			}
		}
	}

## 模块动态注册

在 store 创建之后，可以使用 store.registerModule 方法注册模块。动态卸载模块使用 store.unregisterModule(moduleName) 。注意，你不能使用此方法卸载静态模块（即创建 store 时声明的模块）。

	// 注册模块 `myModule`
	store.registerModule('myModule', {
		// ...
	})
	// 注册嵌套模块 `nested/myModule`
	store.registerModule(['nested', 'myModule'], {
		// ...
	})

模块动态注册功能使得其他 Vue 插件可以通过在 store 中附加新模块的方式来使用 Vuex 管理状态。

#### 保留 state

<!-- 
1. 加班费计算 	1.5倍工资   1.5倍工资 （和经济补偿1.5个月工资二选一）
1. 年假计算 		4天				 	5天
1. 离职结薪日期	11.21			  12.20
1. 经济补偿计算 2个月工资		 1.5个月工资
1. 社保缴至月份 12月份 			 1月份
 -->


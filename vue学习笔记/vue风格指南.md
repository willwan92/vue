# Vue风格指南

这是官方的 Vue 特有代码的风格指南，分为4个类别，对应4个优先级。如下：

- 优先级 A：必要的，规避错误

- 优先级 B：强烈推荐，增强可读性

- 优先级 C：推荐，将选择和认知成本最小化

- 优先级 D：谨慎使用，有潜在危险

## 优先级 A ：必要的规则 (规避错误)

- 组件名应该是多个单词的，根组件 App 除外。

- 组件的 data 必须是一个函数(除了 new Vue 外)。

- Prop 定义应该尽量详细。

		// 更好的做法！
		props: {
			status: {
					type: String,
					required: true,
					validator: function (value) {
							return [
									'syncing',
									'synced',
									'version-conflict',
									'error'
							].indexOf(value) !== -1
					}
			}
		}

- 为 v-for 设置键值（key）

- 永远不要把 v-if 和 v-for 同时用在同一个元素上。

在两种常见的情况下我们可能会把 v-if 和 v-for 同时用在同一个元素上：

1. 为了过滤一个列表中的项目。在这种情形下，可以将列表替换为一个计算属性，让其返回过滤后的列表。
2. 为了避免渲染本应该被隐藏的列表。这种情形下，请将 v-if 移动至父级容器元素上。

- 为组件样式设置作用域

不一定要使用 scoped 特性。设置作用域也可以通过 [CSS Modules](https://vue-loader.vuejs.org/zh-cn/features/css-modules.html)

- 私有属性名

*在插件、混入等扩展中始终为自定义的私有属性使用 `$_` 前缀。并附带一个命名空间以回避和其它作者的冲突 (比如 $_yourPluginName_)。*

	var myGreatMixin = {
		// ...
		methods: {
			$_myGreatMixin_update: function () {
					// ...
			}
		}
	}

## 优先级 B ：强烈推荐的规则 (增强可读性)

- 把每个组件单独分成文件
- 组件的文件名应该要么始终是单词大写开头 (PascalCase)，要么始终是横线连接 (kebab-case)
- 基础组件名以特定的前缀开头

		components/
		|- AppButton.vue
		|- AppTable.vue
		|- AppIcon.vue

- 单例的组件应该以 The 前缀命名，以示其唯一性。

	单例组件每个页面只使用一次，这些组件永远不接受任何 prop，因为它们是为你的应用定制的。

		components/
		|- TheHeading.vue
		|- TheSidebar.vue

- 和父组件紧密耦合的子组件应该以父组件名作为前缀命名。
- 组件名应该以高级别的 (通常是一般化描述的) 单词开头，以描述性的修饰词结尾。

		components/
		|- SearchButtonClear.vue
		|- SearchButtonRun.vue
		|- SearchInputQuery.vue
		|- SearchInputExcludeGlob.vue
		|- SettingsCheckboxTerms.vue
		|- SettingsCheckboxLaunchOnStartup.vue

- 没有内容的组件应该是自闭合的，但在 DOM 模板里不要自闭合。
- 组件名最好是 PascalCase 的，但是在 DOM 模板中要使用 kebab-case 的。
- 组件名应该用完整单词而不是缩写。
- 声明 prop 时，应该使用 camelCase，在模板和 JSX 中应该使用 kebab-case。
- 多个特性的元素应该分多行撰写，每个特性一行。
- 板应该只包含简单的表达式，复杂的表达式则应该重构为计算属性或方法。
- 应该把复杂计算属性分割为尽可能多的更简单的属性。

		computed: {
			basePrice: function () {
					return this.manufactureCost / (1 - this.profitMargin)
			},
			discount: function () {
					return this.basePrice * (this.discountPercent || 0)
			},
			finalPrice: function () {
					return this.basePrice - this.discount
			}
		}

- HTML 特性值应该始终带引号
- 指令缩写保持一致 (例：用 @ 表示 v-on:) 应该要么都用要么都不用

## 优先级 C ：推荐的规则

- 组件/实例的选项应该有统一的顺序。

下面推荐的组件选项默认顺序：

1.	副作用 (触发组件外的影响)

	- el

2.	全局感知 (要求组件以外的知识)

	- name
	- parent

3.	组件类型 (更改组件的类型)

	- functional

4.	模板修改器 (改变模板的编译方式)

	- delimiters
	- comments

5.	模板依赖 (模板内使用的资源)

	- components
	- directives
	- filters

6.	组合 (向选项里合并属性)

	- extends
	- mixins

7.	接口 (组件的接口)

	- inheritAttrs
	- model
	- props/propsData

8.	本地状态 (本地的响应式属性)

	- data
	- computed

9.	事件 (通过响应式事件触发的回调)

	- watch

10.	生命周期钩子 (按照它们被调用的顺序)

	- beforeCreate
	- created
	- beforeMount
	- mounted
	- beforeUpdate
	- updated
	- activated
	- deactivated
	- beforeDestroy
	- destroyed

11.	非响应式的属性 (不依赖响应系统的实例属性)

	- methods

12.	渲染 (组件输出的声明式描述)

	- template/render
	- renderError

- 元素 (包括组件) 的特性应该有统一的顺序。

推荐的默认顺序：

1.	定义 (提供组件的选项)

	- is

1.	列表渲染 (创建多个变化的相同元素)

	- v-for

1.	条件渲染 (元素是否渲染/显示)

	- v-if
	- v-else-if
	- v-else
	- v-show
	- v-cloak

1.	渲染方式 (改变元素的渲染方式)

	- v-pre
	- v-once

1.	全局感知 (需要超越组件的知识)

	- id

1.	唯一的特性 (需要唯一值的特性)

	- ref
	- key
	- slot

1.	双向绑定 (把绑定和事件结合起来)

	- v-model

1.	其它特性 (所有普通的绑定或未绑定的特性)

1.	事件 (组件事件监听器)

	- v-on

1.	内容 (覆写元素的内容)

	- v-html
	- v-text

- 单文件组件应该总是让 `<script>`、`<template>` 和 `<style>` 标签的顺序保持一致，且 `<style>` 要放在最后

## 优先级 D ：谨慎使用的

- 如果一组 v-if + v-else 的元素类型相同，最好使用 key
- 元素选择器应该避免在 scoped 中出现
- 应该优先通过 prop 和事件进行父子组件之间的通信，而不是 this.$parent 或改变 prop
- 应该优先通过 Vuex 管理全局状态，而不是通过 this.$root 或一个全局事件总线


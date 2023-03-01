<template>
	<view class="left-window-style">
		<view class="second-menu">
			<!-- <keep-alive> -->
			<component v-bind:is="active" :hasLeftWin="hasLeftWin" :leftWinActive="leftWinActive"></component>
			<!-- </keep-alive> -->
		</view>
	</view>
</template>

<script>
	import websites from '@/pages/tabBar/websites/websites.nvue'
	import home from '@/pages/tabBar/home/home.nvue'
	import codePage from '@/pages/tabBar/code/code.nvue'
	import calculationPage from '@/pages/tabBar/calculation/index'
	import mediaPage from '@/pages/tabBar/media/media.nvue'
	import documentPage from '@/pages/tabBar/document/document'
	import {
		mapMutations,
		mapState
	} from 'vuex'
	let isPcObserver, isPhoneObserver
	export default {
		data() {
			return {
				nav: [
					'code',
					'document',
					'media',
					'calculation',
					'websites',
					'home',
					'code'
				],
				isPC: false
			}
		},
		components: {
			codePage,
			documentPage,
			mediaPage,
			calculationPage,
			websites,
			home,
			codePage
		},
		computed: {
			...mapState({
				active: state => state.active,
				hasLeftWin: state => !state.noMatchLeftWindow,
				leftWinActive: state => state.leftWinActive.split('/')[3],
			})
		},
		mounted() {
			isPcObserver = uni.createMediaQueryObserver(this)
			isPhoneObserver = uni.createMediaQueryObserver(this)
			//this.setActive('code')
			isPcObserver.observe({
				minWidth: 768
			}, matched => {
				this.isPC = matched
				const pageUrl = this.$route.path
				if (!pageUrl) return
				const pageName = this.$route.path.split('/')[4]
				if (pageUrl === '/' || this.nav.includes(pageName)) {
					const tabbarUrl = pageName ? (pageName === 'component' ? '/' : `/pages/tabBar/${pageName}/${pageName}`) : '/'
					if (pageUrl === '/' || pageUrl === tabbarUrl) {
						uni.switchTab({
							url: pageUrl,
						})
					}
				} else {
					uni.reLaunch({
						url: pageUrl
					})
				}
			})

			isPhoneObserver.observe({
				maxWidth: 767
			}, matched => {
				this.isPC = !matched
				if (matched) {
					const pageUrl = this.$route.path
					const tabbarName = this.$route.path.split('/')[2]
					const tabbarUrl = tabbarName && (tabbarName === 'component' ? '/' : `/pages/tabBar/${tabbarName}/${tabbarName}`)
					uni.switchTab({
						url: tabbarUrl,
						success(e) {
							uni.navigateTo({
								url: pageUrl
							})
						}
					})
				}

			})
		},
		unmounted() {
			isPcObserver.disconnect()
			isPhoneObserver.disconnect()
		},
		watch: {
			isPC: {
				immediate: true,
				handler(newMatches) {
					this.setMatchLeftWindow(newMatches)
					// this.handlerRoute(this.$route)
				}
			},
			// #ifndef VUE3
			$route: {
				immediate: true,
				handler(newRoute) {
					this.handlerRoute(newRoute)
				}
			},
			// #endif
			// #ifdef VUE3
			$route(newRoute) {
				this.handlerRoute(newRoute)
			}
			// #endif
		},

		methods: {
			...mapMutations(['setMatchLeftWindow', 'setActive', 'setLeftWinActive']),

			handlerRoute(newRoute) {
				if (this.isPC) {
					if (newRoute.path === '/') {
						console.log('left window redircTo','/pages/code/list2detail-detail/list2detail-detail' )
						uni.redirectTo({
							url: '/pages/code/list2detail-detail/list2detail-detail'
						})
						this.setActive('codePage')
					} else if (!newRoute.matched.length) {
						uni.redirectTo({
							url: '/pages/error/404'
						})
					} else {
						console.log('newRoute.path',newRoute.path)
						this.setLeftWinActive(newRoute.path)
						let active = newRoute.path.split('/')[2]
						console.log(active)
						if (this.nav.includes(active)) {
							if (active === 'media') {
								active = 'mediaPage'
							}
							if (active === 'document') {
								active = 'documentPage'
							}

							if (active === 'code') {
								active = 'codePage'
							}

							if (active === 'calculation') {
								active = 'calculationPage'
							}

							console.log('active', active)
							this.setActive(active)
						}
					}
				}
			},

			switchTab() {

			}
		}
	}
</script>

<style>
	@import '../common/uni-nvue.css';

	.left-window-style {
		min-height: calc(100vh - var(--top-window-height));
		background-color: #f8f8f8;
	}

	.second-menu {
		width: 350px;
		background-color: #F8F8F8;
	}

	.icon-image {
		width: 30px;
		height: 30px;
	}
</style>

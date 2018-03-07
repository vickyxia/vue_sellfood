<template>
  <div class="shopcart">
  	<!-- toggleList()来回折叠购物车详情 -->
   <div class="content" @click="toggleList">
   	<!-- 购物车左侧部分开始 -->
   	 <div class="content-left">
   	 	<!-- 放ｌｏｇｏ的部分开始 -->
   	 	<div class="logo-wrapper">
   	 		<!-- 放购物车图标的div -->
   	 		<div class="logo" :class="{'highlight':totalCount>0}">
   	 			<span class="cart" >
   	 				<!-- 使用font-awesome里面的fa-shopping-cart,要先npm install font-awesome,再在main.js引入font-awesome.css -->
   	 				<!-- 判断数量大于０就绑定＆.highLight样式高亮 -->
   	 				<i class="fa fa-shopping-cart fa-2x" :class="{'highlight':totalCount>0}"></i>
   	 			</span>
   	 		</div>
   	 		<!-- 购物车左上角的数量小表示，v-show用来控制是否显示，totalCount>0时显示这个div -->
   	 		<div class="num" v-show="totalCount>0">{{totalCount}}</div>
   	 	</div>
   	 	<!-- 放ｌｏｇｏ的部分结束 -->
		<!-- 放总价的部分，用:class添加高亮样式，价格大于０时添加&.highlight -->
   	 	<div class="price" :class="{'highlight':totalPrice>0}">￥{{totalPrice}}</div>
   	 	<!-- 配送费 -->
   	 	<div class="desc">另需配送费{{deliveryPrice}}</div>
   	 </div>
   	 <!-- 购物车左侧部分结束 -->
   	 <!-- 购物车右侧部分开始 -->
   	 <div class="content-right">
   	 	<!-- payDesc分３种情况，对应２不同样式payClass -->
   	 	<div class="pay" :class="payClass">
   	 		{{payDesc}}
   	 	</div>
   	 </div>
   	  <!-- 购物车右侧部分结束 -->
   </div>
   <!-- 小球飞入购物车 -->
   <!-- 两层布局fixed -->
   	
   <!-- 购物车详情 -->
   <transition name="fold">
   <div class="shopcart-list" v-show="listShow">
   	<div class="list-header">
   		<div class="title">购物车</div>
   		<div class="empty">清空</div>
   	</div>
   	<div class="list-content">
   		<ul class="food">
   			<li class="food" v-for="food in selectFoods">
   				<span class="name">{{food.name}}</span>
   				<div class="price">
   					<span>￥{{food.price * food.count}}</span>
   				</div> 
   				<div class="cartcontrol-wrapper">
   					<!-- 这里关联了food这个值 -->
   					<cartcontrol :food="food"></cartcontrol>
   				</div>
   			</li>
   		</ul>
   	</div>
   </div>
</transition>
  </div>
</template>

<script type="text/ecmascript-6">
import cartcontrol from '../cartcontrol/cartcontrol.vue';
export default {
	props: {    // 父组件向字子组件传递数据
		// 购物车状态由selectFoods这个数组决定(自定义)，数组每个值表示当前该商品的总价（price*count）
		// selectFoods默认值为return:[];
		selectFoods: {
			type: Array,
			default() {
				return [  // 这个是测试数据，可以注释掉
				// {
				// 	price: 5,
				// 	count: 2
				// }

				];  // 数组，每个值表示商品总价
			}
		},
		deliveryPrice: {   // 配送费,默认值可以自己设置default
			type: Number,
			default: 3
		},
		minPrice: {       // 起送价钱
			type: Number,
			default: 10
		}
	},
	data() {
		return {
			balls: [
			{
				show: false
			},
			{
				show: false
			},
			{
				show: false
			},
			{
				show: false
			},
			{
				show: false
			}
			],
			dropBalls: [],   // 已经下落的小球
			fold: true   // 用来控制购物车列表详情是否展开
		};
	},
	computed: {    // 计算属性
		totalPrice() {      // 根据selectFoods数组计算总价
			let total = 0;   // 定义总价total = 0 
			this.selectFoods.forEach((food) => {  // 遍历数组，对每个food的food.price和food.count相乘求和
				total += food.price * food.count;
			});
			return total;      //  返回总价
		},
		totalCount() {       // 根据selectFoods数组计算数量
			let count = 0;
			this.selectFoods.forEach((food) => {
				count += food.count;   // 每种food的count相加
			});
			return count;     // 返回总数量
		},
		payDesc() {   // 购物车右侧描述详情
			// 用反引号（波浪线下面）不用拼接字符串，返回{}
			if (this.totalPrice === 0) {       // 当前总价为0显示：￥${this.minPrice}元起送
				return `￥${this.minPrice}元起送`;
			} else if (this.totalPrice < this.minPrice) {
				let diff = this.minPrice - this.totalPrice;  // 计算距离起送价的差价
				return `还差￥${diff}元起送`;     // 返回还差￥${diff}元起送
			} else {
				return `去结算`;    // 满足起送价格条件显示去结算
			}
		},
		payClass() {
			if (this.totalPrice < this.minPrice) {
				return 'not-enough';    // 小于起送价，返回not-enough,:class="not-enough"
			} else {
				return 'enough';    // 返回enough,:class="enough"
			}
		},
		listShow() {
			if (!this.totalCount) {
				this.fold = true;
				return false;
			} 
				let show = !this.fold;
				return show;
		}
	},
	methods: {
		drop(el) {
			// console.log(el);
			// 遍历balls
			for (let i = 0; i < this.balls.length; i++) {
				let ball = this.balls[i];
				if (!ball.show) {
					ball.show = true;  // 找到一个可以显示的小球,触发动画
					ball.el = el;  // 保存这个element
					this.dropBalls.push(ball);
					return;
				}
			}
		},
		// 改变购物车详情显示/隐藏效果
		toggleList() {
			if (!this.totalCount) {
				console.log('no-click');
				return;
			}
			this.fold = !this.fold;
			console.log('click');
		}
	},
	transitions: {
		drop: {
			// 三个钩子,参数的当前执行transition的ＤＯＭ对象
			beforeEnter(el) {
				// 找到设为true的小球
				let count = this.balls.length;
				while (count--) {
					let ball = this.balls[count];
					if (ball.show) {
						let rect = ball.el.getBoundingClientRect(); 
						// 获得小球的视口偏移top和left
						let x = rect.left - 32;
						let y = -(window.innerHeight - rect.top - 22); // 获得小球要跑的xy长度
						// 设置它的初始位置
						el.style.display = '';
						el.style.webkitTransform = `translate3d(0,${y}px,0)`;
						el.style.transform = `translate3d(0,${y}px,0)`;
						let inner = el.getElementsByClassName('inner-hook')[0];
						inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
						inner.style.transform = `translate3d(${x}px,0,0)`;
					}
				}
			},
			enter(el) {
			/* eslint-disable no-unused-vars */
			let rf = el.offsetHeight;
			this.$nextTick(() => {
				// 恢复原来的样子,重绘才有用
				el.style.webkitTransform = 'translate3d(0,0,0)';
				el.style.transform = 'translate3d(0,0,0)';
				let inner = el.getElementsByClassName('inner-hook')[0];
				inner.style.webkitTransform = 'translate3d(0,0,0)';
				inner.style.transform = 'translate3d(0,0,0)';
				});
			},
			afterEnter(el) {
				// 动画结束要把ball变回false
				let ball = this.dropBalls.shift();
				if (ball) {
					ball.show = false;
					el.style.display = 'none';
				}
			}

		}
	},
	components: {
		cartcontrol
	}
};
</script>

<style lang="stylus" ref="stylesheet/stylus">
.shopcart
	position: fixed
	z-index: 50
	left: 0
	bottom: 0
	height: 40px
	background: black
	width: 100%
	.content
		display: flex
		background: #141d27
		.content-left
			flex: 1
			.logo-wrapper
				display: inline-block
				position: relative
				top: -10px
				margin: 0 12px
				padding: 6px
				width: 56px
				height:56px
				box-sizing: border-box
				border-radius: 50%
				vertical-align: center
				background: #141d27
				.logo
					width: 100%
					height: 100%
					border-radius: 50%
					background: #2b343c
					&.highlight
						background: rgb(0, 160, 220)
					.fa
						margin-left: 4px 
						margin-top: 5px
						color: #898989
						&.highlight
							color: #fff
				.num
					position: absolute
					top: 0
					left: 0
					width: 24px
					height: 16px
					line-height: 16px
					text-align: center
					background: rgb(240, 20, 20)
					color: #fff
					font-size: 9px
					font-weight: 700
					box-shadow: 0px 4px 8px 0px rgba(0, 0, 0, 0.4)
					border-radius: 6px
			.price
				display: inline-block
				line-height: 24px
				font-size: 12px
				color: rgb(255, 255, 255, 0.4)
				vertical-align: top
				padding-right: 12px
				box-sizing: border-box
				border-right: 1px solid rgba(255, 255, 255, 0.1)
				font-weight: 700
				padding-top: 8px
				&.highlight
					color: #fff
					font-size: 16px
			.desc
				flex: 0 0 100px
				display: inline-block
				height: 50px
				line-height: 24px
				font-size: 12px
				font-weight: 700
				text-align: center
				width: 100px
				color: rgba(255, 255, 255, 0.4)
				
		.content-right
			flex: 0 0 110px
			width: 120px
			.pay
				height: 48px
				line-height: 48px
				text-align: center
				font-size: 16px
				font-weight: 700
				color: rgba(255, 255, 255, 0.4)
				background: #2b333b
				&.not-enough
					background: #2b333b
				&.enough
					background: #00b43c
					color: #fff
	.ball-container
		.ball
			position: fixed
			left: 32px
			bottom: 22px
			z-index: 200
			&.drop-transition
				transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
				.inner
					width: 16px
					height: 16px
					border-radius: 50%
					background: rgb(0, 160, 220)
					transition: all 0.4s
	.shopcart-list
		position: absolute
		top: 0
		left: 0
		z-index: -1
		width: 100%
		&.fold-transition
			transition: all 0.5s ease
			transform: translate3d(0, -100%, 0)
		&.fold-enter, &.fold-leave
			transform: translate3d(0, 0, 0)



</style>

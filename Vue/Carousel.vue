<template>
   <div 
      v-if="bannerData !== null"
      class="carousel"
      :style="{ height }"
      @mouseover="handleBoxMouseover"
      @mouseout="handleBoxMouseout"
   >
      <ul class="img-container" @click="handleBannerClick">
         <li 
            v-for="(img, index) in bannerData"
            :key="index"
            :index="index"
            :data-id="img.targetId"
            class="imgLi"
            :class="img.currClass"
         >
            <div class="imgWrap">
               <img 
                  draggable="false"
                  :data-class="img.currClass"
                  :src="img.imageUrl"
                  :style="{ borderRadius }"
               />
               <slot :item="img"></slot>
            </div>
         </li>
      </ul>

      <!-- dots -->
      <ul class="dot-wrap">
         <li 
            v-for="(item, index) in dotList"
            :key="item.index"
            :index="index"
            class="dot"
            :class="{ active: item.isActive }"
            :style="{
               backgroundColor: item.isActive ? dotColor : ''  
            }"
            @mouseenter="handleDotEnter"
         ></li>
      </ul>

      <!-- 左按钮 -->
      <span 
         class="switch-dot left-0"
         :style="{ visibility: canBtnShow }"
         @click="handleLeftBtnClick"
      >
         <i class="iconfont icon-pre"></i>
         
      </span>
      <!-- 右按钮 -->
      <span 
         class="switch-dot right-0"
         :style="{ visibility: canBtnShow }"
         @click="handleRightBtnClick"
      >
         <i class="iconfont icon-next"></i>
      </span>

      <!-- 左右切换面板 -->
      <div class="click-board left-0" @click="handleLeftBtnClick"></div>

      <div class="click-board right-0" @click="handleRightBtnClick"></div>
   </div>
</template>

<script>

export default {
   name: "Carousel",
   props: {
      bannerData: {
         type: Array,
         require: true,
      },
      initial: {
         type: Number,
         default: 0
      },
      interval: {
         type: Number,
         default: 4000
      },
      autoplay: {
         type: Boolean,
         default: true
      },
      loop: {
         type: Boolean,
         default: true
      },
      height: {
         type: String,
         default: '240px'
      },
      dotColor: {
         type: String,
         default: 'cornflowerblue'
      },
      borderRadius: {
         type: String,
         default: '30px'
      }
   },
   data() {
      return {
         canBtnShow: 0,
         classContainer: [],
         dotList: [],
         currDotPosi: 0,
         moveTimer: null,
         isDotActive: false,
      };
   },
   created() {
      const banner = this.bannerData;
      const lastBannerIndex = JSON.parse(sessionStorage.getItem("carouselIndex"));

      // 生成对应的 class 列表 
      this.classContainer = this.genClassList(banner.length, this.initial);

      // 给 banner 每一项设置 currClass, 用来后续操纵 class
      banner.forEach((item, index) => {
         this.$set(item, "currClass", this.classContainer[index]);
      });

      // 生成底部小圆点列表, isActive 为真表示当前对应得 banner 项
      this.dotList = banner.map((item, index) => ({
         isActive: index === this.initial,
      }));

      this.currDotPosi = lastBannerIndex > -1 ? lastBannerIndex : this.initial;
   },
   mounted() {
      const banner = this.bannerData;
      const list = this.classContainer;

      // 通过预先生成得 classList, 给对应得 banner 项初始化样式
      list.forEach((item, index) => {
         banner[index].currClass = item;
      })

      if (this.autoplay) {
         this.initTimer(this.interval);
      }
      this.handleTabsChange();
   },
   beforeDestroy() {
      clearTimeout(this.moveTimer);
   },
   watch: {
      $route() {
         sessionStorage.setItem(
            "carouselIndex",
            this.currDotPosi
         );
      },
   },
   methods: {
      // 动态生成 classList
      genClassList(length = 0, initialIndex = 1) {
         if (initialIndex < 0 || initialIndex >= length) initialIndex = 1;

         const list = new Array(length).fill(0);

         const classList = list.map((item, index) => {
            if (initialIndex === 0) {
               switch (index) {
                  case initialIndex: return 'list-item2';
                  case 1: return 'list-item3';
                  case length - 1: return 'list-item1';
                  default: return 'list-back';
               }
            } else if (initialIndex === length - 1) {
               switch (index) {
                  case initialIndex: return 'list-item2';
                  case 0: return 'list-item3';
                  case initialIndex - 1: return 'list-item1';
                  default: return 'list-back';
               }
            } else {
               switch (index) {
                  case initialIndex - 1: return 'list-item1';
                  case initialIndex: return 'list-item2';
                  case initialIndex + 1: return 'list-item3';
                  default: return 'list-back';
               }
            }
         });

         return classList;
      },
      handleBoxMouseover() {
         this.canBtnShow = 'visible';
         clearInterval(this.moveTimer);
         this.moveTimer = null;
      },
      handleBoxMouseout() {
         this.canBtnShow = 'hidden';
         if (this.autoplay) {
            clearInterval(this.moveTimer);
            this.initTimer(this.interval);
         }
      },
      handleLeftBtnClick() {
         if (!this.loop && this.currDotPosi === 0) return;
         this.moveToLeft();
      },
      handleRightBtnClick() {
         if (!this.loop && this.currDotPosi === this.dotList.length - 1) return;
         this.moveToRight();
      },
      dotMove() {
         const len = this.dotList.length;
         const posi = this.currDotPosi;

         // 先使所有底部小圆点失效
         this.dotList.forEach(dot => dot.isActive = false);
         // 处理边界情况
         this.currDotPosi = 
            posi >= len ? 0:
            posi < 0 ? len - 1: 
            posi;
         
         // 为当前小圆点添加样式
         this.dotList[this.currDotPosi].isActive = true;
      },
      // 处理当鼠标悬停在小圆点上时, 使得 banner 切换到对应位置
      handleDotEnter(e) {
         const dotPosi = e.target.getAttribute("index");
         const itemHeadPosi = this.classContainer.indexOf("list-item2");
         // 计算出悬停位置与当前在最前方 banner item 位置的差值
         let diff = Math.max(dotPosi, itemHeadPosi) - Math.min(dotPosi, itemHeadPosi);

         while (diff--) {
            if (dotPosi > itemHeadPosi) {
               // 悬停位置在当前右侧
               this.moveToRight()
            } else {
               // 悬停位置在当前左侧
               this.moveToLeft()
            }
         }
      },
      moveToRight(e) {
         const banner = this.bannerData;
         const list = this.classContainer;
         
         // 将队列最后一个删除, 并添加到队头
         const last = list.pop();
         list.unshift(last);

         // 重置 banner 每一项
         list.forEach((item, i) => banner[i].currClass = item);

         // 小圆点当前位置同时 + 1, 并向右移动一位
         this.currDotPosi++;
         this.dotMove();

         if (typeof this.$listeners.change === 'function') {
            this.$listeners.change(this.currDotPosi, banner[this.currDotPosi]);
         }
      },
      moveToLeft() {
         const banner = this.bannerData;
         const list = this.classContainer;

         // 同理, 删除队头项并添加到队尾
         const first = list.shift();
         list.push(first);

         // 重置 banner 每一项
         list.forEach((item, i) => banner[i].currClass = item);

         // 小圆点当前位置同时 - 1, 并向左移动一位
         this.currDotPosi--;
         this.dotMove();

         if (typeof this.$listeners.change === 'function') {
            this.$listeners.change();
         }
      },
      initTimer(interval) {
         // 开启自动轮播
         this.moveTimer = setInterval(() => {
            if (!this.loop && this.currDotPosi === this.dotList.length - 1) return;
            this.moveToRight();
         }, interval);
      },
      // 获取 document 上关于页面是否被隐藏的属性名
      // 未用, 暂时保留
      getHiddenProp() {
         let prefixes = ["webkit", "moz", "ms", "o"];

         if ("hidden" in document) return "hidden";

         for (const value of prefixes) {
            const prop = value + "Hidden";
            if (prop in document) {
               return prop;
            }
         }

         return '';
      },
      // 判断当前 Tab 是否不可见
      // 未用, 暂时保留
      isHidden() {
         const hiddenProp = this.getHiddenProp();
         if (!hiddenProp) return '';

         return document[hiddenProp];
      },
      // 当页面不可见时, 关闭轮播
      handleTabsChange() {
         let evtName = '';
         // 获取对应浏览器的控制页面显隐的事件名
         // 处理  Safari
         evtName = 'pagehide' in document ? 'pagehide' : 'visibilitychange';
         document.addEventListener(evtName, (e) => {
            const visiable = document.visibilityState;
            if (visiable !== 'visible') {
               clearInterval(this.moveTimer);
            } else {
               // if (this.moveTimer) clearInterval(this.moveTimer);
               this.initTimer(this.interval);
            }
         });
      },
      handleBannerClick(e) {
         const currPosiFlag = e.target.dataset.class;

         // 触发 headItemClick 事件
         if (currPosiFlag === 'list-item2') {
            if (typeof this.$listeners.headItemClick === 'function') {
               this.$listeners.headItemClick(e, this.bannerData[this.currDotPosi]);
            }
         }
      },
   },
};
</script>

<style scope>
@import "https://at.alicdn.com/t/font_2956590_jmm5iqzca.css";

.carousel {
   position: relative;
   height: 200px;
   margin: auto;
   user-select: none;
   width: 100%;
   box-sizing: border-box;
}
.img-container {
   position: relative;
   width: 58%;
   height: 100%;
   left: 50%;
   transform: translateX(-50%);
   padding-bottom: 20px;
}

.imgLi {
   position: absolute;
   width: 100%;
   height: 100%;
   transition-duration: 0.8s;
}

.imgWrap {
   position: relative;
   width: 100%;
   height: 100%;
}

.imgWrap img {
   width: 100%;
   height: 100%;
   object-fit: cover;
}

.list-item1 {
   transform: translate(-42%, 0px) scale(0.8);
   z-index: 1;
}
.carousel .list-item2 {
   z-index: 2;
   cursor: pointer;
}
.list-item3 {
   transform: translate(42%, 0px) scale(0.8);
   z-index: 1;
}

.list-back {
   transform: scale(0.6);
   z-index: 0;
}

.click-board {
   position: absolute;
   height: 60%;
   width: 10%;
   top: 20%;
   z-index: 0;
}

.dot-wrap {
   position: absolute;
   margin: 15px 0px;
   left: 50%;
   transform: translateX(-50%);
}

.dot {
   background-color: rgba(0, 0, 0, 0.1);
   float: left;
   width: 6px;
   height: 6px;
   border-radius: 3px;
}

.dot:not(:last-child) {
   margin-right: 10px;
}

.dot.active {
   background-color: #ec4141;
}

.switch-dot {
   position: absolute;
   transition: opacity 0.4s;
   background-color: rgb(0, 0, 0);
   text-align: center;
   height: 25px;
   width: 25px;
   line-height: 25px;
   border-radius: 50%;
   color: #fff;
   cursor: pointer;
   top: 50%;
   transform: translateY(-50%);
   z-index: 100;
   opacity: 0.3;
}

.switch-dot i {
   text-align: center;
}

.switch-dot:hover {
   opacity: 0.7;
}

.left-0 {
   left: 5%;
}

.right-0 {
   right: 5%;
}
</style>
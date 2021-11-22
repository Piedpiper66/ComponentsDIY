# Carousel  走马灯

> bannerData 每一项至少需要一项 imageUrl 属性

## Carousel	Attributes

| 参数         | 说明                     | 类型    | 默认值           |
| ------------ | ------------------------ | ------- | ---------------- |
| bannerData   | 轮播图数据               | Array   | [ ]              |
| initial      | 初始索引位置             | number  | 0                |
| interval     | 轮播间隔时间             | number  | 4000             |
| autoplay     | 是否自动播放             | boolean | true             |
| loop         | 是否循环播放             | boolean | true             |
| height       | 轮播图高度               | string  | '240px'          |
| dotColor     | 轮播图下方滚动小球的颜色 | string  | 'cornflowerblue' |
| borderRadius | 图片圆角大小             | string  | '10px'           |

## Carousel Events

| 事件名称      | 说明                       | 回调参数                               |
| ------------- | -------------------------- | -------------------------------------- |
| change        | 轮播图切换时触发           | 当前轮播图索引，对应索引位置的数据对象 |
| headItemClick | 点击轮播图最上层图片时触发 | 事件对象，对应索引位置的数据对象       |

## Carousel slot

**设置了一个默认的作用域插槽**

**以防每一项的最上面还需要加一些元素**

```vue
// 可以通过默认独占插槽中的 item 属性获取对应数据对象
// 数据对象为 bannerData 中的各项
<Carousel v-slot="slotProps">
    // exmple
	<span>{{ slotProps.item.title }}</span>
</Carousel>
```


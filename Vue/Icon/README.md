# Svg-Icon  svg图标

## Icon Attributes

| 参数      | 说明                                             | 类型    | 默认值         |
| --------- | ------------------------------------------------ | ------- | -------------- |
| name      | svg文件的文件名                                  | string  | 无             |
| className | svg 元素的类名                                   | string  | ""             |
| width     | 元素宽度                                         | string  | 1.25rem        |
| height    | 元素高度                                         | string  | 1.25rem        |
| size      | 元素大小 : width&& height && (width === hetight) | string  | ""             |
| fill      | svg元素所填充的颜色                              | string  | "currentColor" |
| spin      | 元素是否旋转                                     | boolean | false          |

## 集成 SVG 文件

```js
// utils/svg.js

const requireAll = (requireContext) => requireContext.keys().map(requireContext);
const req = require.context("@/assets/svg", false, /\.svg$/);

requireAll(req);

// main.js
import "./utils/svg";
```

```js
// vue.config.js
module.exports = {
    chainWebpack: (config) => {
    // svg 规则
    const svgRule = config.module.rule("svg");
    // 清除已有的loader, 如果不清除会在原有loader之后再使用当前loader规则
    svgRule.uses.clear();
    // 正则匹配排除node_modules目录
    svgRule.exclude.add(/node_modules/);
    // 添加svg新的loader处理
    svgRule
      .test(/\.svg$/)
      .use("svg-sprite-loader")
      .loader("svg-sprite-loader")
      .options({
        symbolId: "icon-[name]",
      });

    // 去除 svg 指定的冗余属性
    svgRule
      .test(/\.svg$/)
      .use("svgo-loader")
      .loader("svgo-loader")
      .options({
        plugins: [
          {
            name: "removeAttrs",
            params: {
              attrs: [
                "fill",
                "fill-rule",
                "class",
                "height",
                "width",
                "opacity",
              ],
            },
          },
        ],
      });

    // 修改images loader，添加svg处理
    const imagesRule = config.module.rule("images");
    imagesRule.exclude.add(resolve("src/assets/svg"));
    config.module.rule("images").test(/\.(png|jpe?g|gif|svg)(\?.*)?$/);
}
```


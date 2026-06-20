## wechat-article-style-skill

> 公众号文章排版样式规范与装饰性卡片模板。提供完整的排版参数（字号、行高、字间距、颜色）和可复用的装饰性卡片 HTML 代码。当用户需要：(1) 创建或编辑公众号文章，(2) 了解公众号排版规范，(3) 使用装饰性卡片样式，(4) 生成公众号格式的 HTML 内容时使用此技能。


# 公众号文章排版样式

## 概述

本技能提供公众号文章的完整排版规范和装饰性卡片模板，基于对南信大海科院公众号文章的深度分析。包含字号、行高、字间距、颜色等核心参数，以及信息卡片、标题标签、引用块等装饰性元素的完整 HTML 代码。

## 快速参考

### 核心排版参数

| 参数 | 正文 | 图注 | 标题 | 强调 |
|------|------|------|------|------|
| 字号 | 15px | 14px | 16-18px | 15px |
| 行高 | 1.8 | 1.6 | 1.8 | 1.8 |
| 字间距 | 1.6px | 0px | 1px | 1.6px |
| 对齐 | justify | center | justify | justify |
| 颜色 | rgb(62,62,62) | rgb(62,62,62) | rgb(62,62,62) | rgb(62,62,62) |
| 首行缩进 | 2.2133em | 0em | - | - |

### 颜色规范

| 用途 | 颜色值 | 说明 |
|------|--------|------|
| 正文 | rgb(62, 62, 62) | 深灰色，非纯黑 |
| 主色调-蓝 | rgb(33, 150, 243) | 强调、标签背景 |
| 主色调-青蓝 | rgb(95, 156, 239) | 装饰、边框 |
| 主色调-浅蓝 | rgb(120, 185, 226) | 分隔线、装饰 |
| 深蓝-1 | rgb(22, 93, 155) | 边框、重点 |
| 深蓝-2 | rgb(16, 76, 117) | 边框 |
| 浅蓝背景-1 | rgb(199, 225, 247) | 卡片背景 |
| 浅蓝背景-2 | rgb(241, 245, 255) | 卡片背景 |
| 浅蓝背景-3 | rgb(244, 250, 255) | 卡片背景 |
| 浅蓝背景-4 | rgb(245, 249, 252) | 卡片背景 |
| 浅蓝背景-5 | rgb(249, 253, 254) | 卡片背景 |
| 浅蓝背景-6 | rgb(242, 247, 252) | 卡片背景 |
| 浅蓝背景-7 | rgb(225, 240, 253) | 卡片背景 |
| 白色 | rgb(255, 255, 255) | 文字、背景 |
| 浅灰 | rgb(245, 245, 245) | 背景 |
| 边框灰 | rgb(189, 203, 212) | 虚线边框 |
| 绿色-1 | rgb(75, 143, 120) | 装饰点 |
| 绿色背景 | rgb(236, 249, 243) | 卡片背景 |
| 青色-1 | rgb(43, 158, 228) | 渐变起始 |
| 青色-2 | rgb(0, 210, 192) | 渐变结束 |
| 青色-3 | rgb(121, 181, 229) | 渐变起始 |
| 青色-4 | rgb(72, 211, 208) | 渐变结束 |

## 正文排版

### 标准正文段落

```html
<section style="text-align: justify; font-size: 15px; line-height: 1.8; letter-spacing: 1.6px; box-sizing: border-box;">
    <p style="text-indent: 2.2133em; white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
        <span leaf="">这里是正文内容，首行缩进。</span>
    </p>
</section>
```

### 强调文字

```html
<strong style="box-sizing: border-box;">
    <span leaf="">强调内容</span>
</strong>
```

### 特殊首行缩进

```html
<!-- 2.2em 缩进（部分文章使用） -->
<p style="text-indent: 2.2em; ...">
```

## 分隔线装饰

### 1. 双横线+圆点分隔线

适用于：章节分隔、视觉装饰

```html
<section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: 10px 0px; box-sizing: border-box;">
    <section style="display: inline-block; vertical-align: middle; width: auto; flex: 100 100 0%; height: auto; padding: 0px 10px 0px 0px; align-self: center; box-sizing: border-box;">
        <section style="margin: 5px 0px; box-sizing: border-box;">
            <section style="background-color: rgb(120, 185, 226); height: 1px; box-sizing: border-box;">
                <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: middle; width: auto; align-self: center; flex: 0 0 auto; min-width: 5%; max-width: 100%; height: auto; padding: 0px; line-height: 0; box-sizing: border-box;">
        <section style="text-align: center; box-sizing: border-box;">
            <section style="display: inline-block; width: 16px; height: 16px; vertical-align: top; overflow: hidden; border-width: 0px; border-radius: 100%; border-style: none; border-color: rgb(62, 62, 62); background-color: rgba(120, 185, 226, 0.2); box-sizing: border-box;">
            </section>
        </section>
        <section style="text-align: center; margin: -6px 0px 0px; box-sizing: border-box;">
            <section style="display: inline-block; width: 18px; height: 18px; vertical-align: top; overflow: hidden; border-width: 0px; border-radius: 100%; border-style: none; border-color: rgb(62, 62, 62); background-color: rgba(120, 185, 226, 0.2); box-sizing: border-box;">
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: middle; width: auto; align-self: center; padding: 0px 0px 0px 10px; flex: 100 100 0%; height: auto; box-sizing: border-box;">
        <section style="margin: 5px 0px; box-sizing: border-box;">
            <section style="background-color: rgb(120, 185, 226); height: 1px; box-sizing: border-box;">
                <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
            </section>
        </section>
    </section>
</section>
```

### 2. 左右对称短横线

适用于：标题上方装饰

```html
<section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: 10px 0px 0px; box-sizing: border-box;">
    <section style="display: inline-block; vertical-align: top; width: 50%; align-self: flex-start; flex: 0 0 auto; height: auto; line-height: 0; padding: 0px 0px 0px 4px; box-sizing: border-box;">
        <section style="display: flex; width: 100%; flex-flow: column; box-sizing: border-box;">
            <section style="z-index: 1; box-sizing: border-box;">
                <section style="margin: 0px; box-sizing: border-box;">
                    <section style="display: inline-block; width: 20px; height: 4px; vertical-align: top; overflow: hidden; background-color: rgb(33, 150, 243); margin: 0px; box-sizing: border-box;">
                        <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
                    </section>
                </section>
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: top; width: 50%; align-self: flex-start; flex: 0 0 auto; height: auto; line-height: 0; padding: 0px 4px 0px 0px; box-sizing: border-box;">
        <section style="display: flex; width: 100%; flex-flow: column; box-sizing: border-box;">
            <section style="z-index: 1; box-sizing: border-box;">
                <section style="text-align: right; margin: 0px; box-sizing: border-box;">
                    <section style="display: inline-block; width: 20px; height: 4px; vertical-align: top; overflow: hidden; background-color: rgb(33, 150, 243); margin: 0px; box-sizing: border-box;">
                        <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
                    </section>
                </section>
            </section>
        </section>
    </section>
</section>
```

### 3. 渐变背景分隔线

适用于：分隔线装饰

```html
<section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: 10px 0px; width: 100%; align-self: flex-start; background-image: linear-gradient(90deg, rgba(83, 209, 232, 0.16) 13%, rgba(37, 72, 255, 0.1) 88%); padding: 20px 10px; box-sizing: border-box;">
    <section style="text-align: justify; width: 100%; box-sizing: border-box;">
        <p style="text-indent: 2.2133em; white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
            <span leaf="">内容</span>
        </p>
    </section>
</section>
```

## 装饰性卡片

### 1. 标准重点信息卡片（蓝色强调）

适用于：重要信息、新闻公告

```html
<section style="margin: 10px 0px; isolation: isolate; text-align: center; justify-content: center; display: flex; flex-flow: row; box-sizing: border-box;">
    <section style="display: inline-block; width: 100%; vertical-align: top; background-color: rgb(245, 249, 252); margin: 0px; flex: 0 0 auto; height: auto; padding: 16px 23px; border-style: solid; border-width: 1px 6px 6px 1px; border-color: rgb(22, 93, 155); align-self: flex-start; box-sizing: border-box;">
        <section style="text-align: justify; box-sizing: border-box;">
            <p style="text-indent: 2.2133em; white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
                <span leaf="">重要信息内容</span>
            </p>
        </section>
    </section>
</section>
```

### 2. 双层边框引用块

适用于：技术说明、详细解释

```html
<section style="margin-top: 10px; margin-bottom: 10px; box-sizing: border-box;">
    <section style="border: 1px solid rgb(120, 185, 226); padding: 3px; box-sizing: border-box;">
        <section style="border-color: rgb(120, 185, 226); border-width: 3px; border-style: solid; background-color: rgb(245, 249, 252); padding: 16px 10px; box-sizing: border-box;">
            <section style="box-sizing: border-box;">
                <p style="text-indent: 2.2133em; white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
                    <span leaf="">引用内容</span>
                </p>
            </section>
        </section>
    </section>
</section>
```

### 3. 三层嵌套卡片（外实内虚）

适用于：研究方法、技术细节

```html
<section style="margin-top: 10px; margin-bottom: 10px; box-sizing: border-box;">
    <section style="border: 3px solid rgb(95, 156, 239); padding: 3px; box-sizing: border-box;">
        <section style="border-color: rgb(98, 161, 247); border-width: 1px; border-style: dashed; padding: 10px; background-color: rgba(255, 255, 255, 0); box-sizing: border-box;">
            <section style="font-size: 15px; line-height: 1.8; letter-spacing: 1.6px; box-sizing: border-box;">
                <p style="text-indent: 2.2133em; white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
                    <span leaf="">内容</span>
                </p>
            </section>
        </section>
    </section>
</section>
```

### 4. 带阴影的卡片

适用于：会议通知、活动公告

```html
<section style="margin-top: 10px; margin-bottom: 10px; box-sizing: border-box;">
    <section style="display: inline-block; width: 100%; border: 1px solid rgb(95, 156, 239); box-shadow: rgb(204, 204, 204) 0.2em 0.2em 0.3em; padding: 10px 15px; box-sizing: border-box;">
        <section style="font-size: 15px; letter-spacing: 1.6px; line-height: 1.8; box-sizing: border-box;">
            <p style="text-indent: 2.2133em; white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
                <span leaf="">卡片内容</span>
            </p>
        </section>
    </section>
</section>
```

### 5. 圆角阴影卡片（带顶部装饰条）

适用于：会议报道、活动通知

```html
<section style="text-align: center; justify-content: center; display: flex; flex-flow: row; margin: 0px 0px 10px; box-sizing: border-box;">
    <section style="display: inline-block; width: 97%; vertical-align: top; align-self: flex-start; flex: 0 0 auto; background-color: rgb(244, 250, 255); height: auto; border-radius: 10px; overflow: hidden; box-shadow: rgb(199, 225, 247) 0px 5px 13px -4px; padding: 16px 21px; border-style: solid; border-width: 0px; border-color: rgb(199, 225, 247); margin: 0px; box-sizing: border-box;">
        <!-- 顶部装饰条 -->
        <section style="display: flex; flex-flow: row; text-align: left; justify-content: flex-start; margin: 10px 0px; box-sizing: border-box;">
            <section style="display: inline-block; vertical-align: middle; width: auto; align-self: center; flex: 100 100 0%; height: auto; box-sizing: border-box;">
                <section style="font-size: 0px; margin: 10px 0%; text-align: justify; justify-content: flex-start; display: flex; flex-flow: row; width: 100%; align-self: flex-start; background-color: rgb(171, 215, 247); box-sizing: border-box;">
                    <section style="margin: 0px 0%; width: 100%; box-sizing: border-box;">
                        <section style="border-top: 3px dashed rgb(250, 254, 255); box-sizing: border-box;">
                            <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
                        </section>
                    </section>
                </section>
            </section>
            <section style="display: inline-block; vertical-align: middle; width: 34px; align-self: center; flex: 0 0 auto; height: auto; box-sizing: border-box;">
                <section style="text-align: center; margin: 0px; line-height: 0; opacity: 0.76; box-sizing: border-box;">
                    <section style="max-width: 100%; vertical-align: middle; display: inline-block; line-height: 0; width: 26px; height: auto; box-sizing: border-box;" nodeleaf="">
                        <img class="rich_pages wxw-img __bg_gif" style="vertical-align: middle; max-width: 100%; width: 26px !important; box-sizing: border-box; height: auto !important;" src="图标URL" alt="">
                    </section>
                </section>
            </section>
            <section style="display: inline-block; vertical-align: middle; width: auto; align-self: center; flex: 100 100 0%; height: auto; box-sizing: border-box;">
                <section style="font-size: 0px; margin: 10px 0%; text-align: justify; justify-content: flex-start; display: flex; flex-flow: row; width: 100%; align-self: flex-start; background-color: rgb(171, 215, 247); box-sizing: border-box;">
                    <section style="margin: 0px 0%; width: 100%; box-sizing: border-box;">
                        <section style="border-top: 3px dashed rgb(250, 254, 255); box-sizing: border-box;">
                            <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
                        </section>
                    </section>
                </section>
            </section>
        </section>
        <section style="text-align: justify; line-height: 1.8; letter-spacing: 1.6px; font-size: 15px; box-sizing: border-box;">
            <p style="text-indent: 2.2em; white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
                <span leaf="">正文内容</span>
            </p>
        </section>
    </section>
</section>
```

### 6. 标签式标题（带三角形装饰）

适用于：论文引用、作者简介、项目资助

```html
<section style="display: flex; width: 100%; flex-flow: column; box-sizing: border-box;">
    <section style="z-index: 1; box-sizing: border-box;">
        <section style="text-align: center; justify-content: center; display: flex; flex-flow: row; margin: 10px 0px -17px; box-sizing: border-box;">
            <section style="display: inline-block; vertical-align: bottom; width: auto; align-self: flex-end; flex: 0 0 0%; height: auto; padding: 0px 3px 0px 0px; box-sizing: border-box;">
                <section style="display: inline-block; width: 0px; height: 0px; vertical-align: top; overflow: hidden; border-style: solid; border-width: 13px 8px; border-color: rgb(120, 185, 226) rgb(120, 185, 226) rgb(120, 185, 226) rgba(255, 255, 255, 0); box-sizing: border-box;">
                </section>
            </section>
            <section style="display: inline-block; vertical-align: bottom; width: auto; align-self: flex-end; flex: 0 0 auto; background-color: rgb(120, 185, 226); margin: 0px; padding: 8px 10px 8px 14px; min-width: 5%; max-width: 100%; height: auto; box-sizing: border-box;">
                <section style="text-align: justify; color: rgb(245, 249, 252); box-sizing: border-box;">
                    <p style="white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
                        <strong style="box-sizing: border-box;"><span leaf="">标签标题</span></strong>
                    </p>
                </section>
            </section>
            <section style="display: inline-block; vertical-align: bottom; width: auto; flex: 0 0 0%; height: auto; align-self: flex-end; padding: 0px 0px 0px 3px; box-sizing: border-box;">
                <section style="transform: perspective(0px); transform-style: flat; box-sizing: border-box;">
                    <section style="transform: rotateY(180deg); box-sizing: border-box;">
                        <section style="display: inline-block; width: 0px; height: 0px; vertical-align: top; overflow: hidden; border-style: solid; border-width: 13px 8px; border-color: rgb(120, 185, 226) rgb(120, 185, 226) rgb(120, 185, 226) rgba(255, 255, 255, 0); box-sizing: border-box;">
                        </section>
                    </section>
                </section>
            </section>
        </section>
    </section>
</section>
```

### 7. 引用内容区

配合标签式标题使用

```html
<section style="justify-content: flex-start; display: flex; flex-flow: row; margin: 0px 0px 10px; width: 100%; background-color: rgb(255, 246, 246); align-self: flex-start; padding: 20px; border-bottom: 3px solid rgb(222, 54, 54); box-sizing: border-box;">
    <section style="margin: 0px; width: 100%; box-sizing: border-box;">
        <section style="padding: 0px; line-height: 1.8; letter-spacing: 1.8px; font-size: 15px; width: 100%; box-sizing: border-box;">
            <p style="white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
                <span leaf="">引用内容</span>
            </p>
        </section>
    </section>
</section>
```

### 8. 作者卡片

适用于：作者简介

```html
<section style="text-align: center; justify-content: center; display: flex; flex-flow: row; box-sizing: border-box;">
    <section style="display: inline-block; width: 97%; vertical-align: top; align-self: flex-start; flex: 0 0 auto; height: auto; background-color: rgb(241, 245, 255); padding: 19px 10px; box-sizing: border-box;">
        <section style="line-height: 0; box-sizing: border-box;">
            <section style="max-width: 100%; vertical-align: middle; display: inline-block; line-height: 0; box-sizing: border-box;" nodeleaf="">
                <img class="rich_pages wxw-img" style="vertical-align: middle; max-width: 100%; width: 100%; box-sizing: border-box;" src="头像URL" alt="作者照片">
            </section>
        </section>
        <section style="text-align: justify; box-sizing: border-box;">
            <p style="text-indent: 2.2133em; white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
                <span leaf="">作者介绍内容...</span>
            </p>
        </section>
    </section>
</section>
```

### 9. 图片+图注组合

适用于：包含图表的研究文章

```html
<section style="text-align: center; justify-content: center; display: flex; flex-flow: row; margin: 0px; box-sizing: border-box;">
    <section style="display: inline-block; width: auto; vertical-align: top; align-self: flex-start; flex: 100 100 0%; border-style: dashed; border-width: 1px; border-color: rgb(95, 156, 239); padding: 15px; height: auto; margin: 0px; border-radius: 10px; overflow: hidden; box-sizing: border-box;">
        <section style="line-height: 0; box-sizing: border-box;">
            <section style="max-width: 100%; vertical-align: middle; display: inline-block; line-height: 0; box-sizing: border-box;" nodeleaf="">
                <img class="rich_pages wxw-img" data-ratio="0.984" data-type="png" data-w="634" style="vertical-align: middle; max-width: 100%; width: 100%; box-sizing: border-box;" src="图片URL" alt="图片说明">
            </section>
        </section>
        <section style="text-align: center; font-size: 14px; line-height: 1.6; letter-spacing: 0px; box-sizing: border-box;">
            <p style="margin: 0px; padding: 0px; box-sizing: border-box;">
                <span leaf="">图1. 图片说明文字</span>
            </p>
        </section>
    </section>
</section>
```

### 10. 数字标号卡片

适用于：步骤说明、时间线、活动安排

```html
<section style="display: flex; flex-flow: row; margin: 10px 0%; justify-content: flex-start; box-sizing: border-box;">
    <section style="display: inline-block; vertical-align: middle; width: 60px; flex: 0 0 auto; height: auto; align-self: center; box-sizing: border-box;">
        <section style="text-align: left; box-sizing: border-box;">
            <section style="display: inline-block; width: 40px; height: 40px; vertical-align: top; overflow: hidden; border-top-right-radius: 14px; background-color: rgb(33, 150, 243); box-sizing: border-box;">
                <section style="margin: 22px 0% 0px; box-sizing: border-box;">
                    <section style="color: rgb(232, 235, 238); font-size: 18px; text-align: center; line-height: 0; box-sizing: border-box;">
                        <p style="margin: 0px; padding: 0px; box-sizing: border-box;">
                            <strong style="box-sizing: border-box;"><span leaf="">01</span></strong>
                        </p>
                    </section>
                </section>
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: middle; width: auto; align-self: center; min-width: 10%; max-width: 100%; flex: 0 0 auto; height: auto; box-sizing: border-box;">
        <section style="margin: 0px 0%; text-align: left; transform: translate3d(-1px, 0px, 0px); box-sizing: border-box;">
            <section style="font-size: 18px; line-height: 1.8; letter-spacing: 1px; color: rgb(120, 185, 226); padding: 0px; box-sizing: border-box;">
                <p style="margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf=""><br></span></p>
            </section>
        </section>
    </section>
</section>
<section style="justify-content: flex-start; display: flex; flex-flow: row; box-sizing: border-box;">
    <section style="display: inline-block; width: 100%; vertical-align: top; border-left: 2px dashed rgb(189, 203, 212); border-bottom-left-radius: 0px; align-self: flex-start; flex: 0 0 auto; box-sizing: border-box;">
        <section style="margin: 0px 0% 20px; box-sizing: border-box;">
            <section style="font-size: 15px; line-height: 1.8; letter-spacing: 1.6px; padding: 0px 20px; box-sizing: border-box;">
                <p style="text-indent: 2.2133em; margin: 0px; padding: 0px; box-sizing: border-box;">
                    <span leaf="">内容</span>
                </p>
            </section>
        </section>
    </section>
</section>
```

### 11. 引用装饰（双侧边线+四角圆点）

适用于：重要引用、核心观点

```html
<section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: 10px 0px; box-sizing: border-box;">
    <section style="display: inline-block; vertical-align: top; width: 10px; align-self: flex-start; flex: 0 0 auto; height: auto; border-left: 4px solid rgb(22, 93, 155); margin: 16px 0px 0px; line-height: 0; box-sizing: border-box;">
        <section style="display: flex; width: 100%; flex-flow: column; box-sizing: border-box;">
            <section style="z-index: auto; box-sizing: border-box;">
                <section style="text-align: center; margin: -6px 0px 50px; box-sizing: border-box;">
                    <section style="display: inline-block; width: 6px; height: 6px; vertical-align: top; overflow: hidden; background-color: rgb(22, 93, 155); margin: 0px; box-sizing: border-box;">
                        <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
                    </section>
                </section>
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: top; width: 50px; align-self: flex-start; flex: 0 0 auto; height: auto; border-top: 4px solid rgb(22, 93, 155); margin: 0px 0px 0px 6px; box-sizing: border-box;">
        <section style="display: flex; width: 100%; flex-flow: column; box-sizing: border-box;">
            <section style="z-index: auto; box-sizing: border-box;">
                <section style="margin: 0px; transform: translate3d(-6px, 0px, 0px); box-sizing: border-box;">
                    <section style="display: inline-block; width: 6px; height: 6px; vertical-align: top; overflow: hidden; background-color: rgb(22, 93, 155); margin: 0px; box-sizing: border-box;">
                        <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
                    </section>
                </section>
            </section>
        </section>
    </section>
</section>
<section style="display: flex; width: 100%; flex-flow: column; box-sizing: border-box;">
    <section style="z-index: 1; box-sizing: border-box;">
        <section style="padding: 0px 20px; font-size: 15px; line-height: 1.8; letter-spacing: 1.6px; box-sizing: border-box;">
            <p style="text-indent: 2.2133em; white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
                <span leaf="">引用内容</span>
            </p>
        </section>
    </section>
</section>
```

### 12. 渐变角标卡片

适用于：总结、结语

```html
<section style="margin: 10px 0px; display: inline-block; width: 100%; box-sizing: border-box;">
    <section style="display: flex; width: 100%; flex-flow: column; box-sizing: border-box;">
        <section style="z-index: auto; box-sizing: border-box;">
            <section style="justify-content: center; display: flex; flex-flow: row; margin: 0px 0px 1px; box-sizing: border-box;">
                <section style="display: inline-block; vertical-align: top; width: auto; min-width: 5%; max-width: 100%; flex: 0 0 auto; height: auto; box-sizing: border-box;">
                    <section style="text-align: left; box-sizing: border-box;">
                        <section style="display: inline-block; width: 61px; height: 61px; vertical-align: top; overflow: hidden; background-image: linear-gradient(rgb(121, 181, 229) 13%, rgb(72, 211, 208) 88%); box-sizing: border-box;">
                        </section>
                    </section>
                </section>
                <section style="display: inline-block; width: auto; vertical-align: top; align-self: flex-start; flex: 100 100 0%; border-style: solid; border-width: 1px; border-color: rgb(205, 221, 241); height: auto; margin: 3px -58px; background-color: rgb(245, 249, 252); z-index: 1; padding: 19px 20px; box-sizing: border-box;">
                    <section style="text-align: justify; font-size: 15px; line-height: 1.8; letter-spacing: 1.6px; box-sizing: border-box;">
                        <p style="text-indent: 2.2133em; white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
                            <span leaf="">总结内容</span>
                        </p>
                    </section>
                </section>
                <section style="display: inline-block; vertical-align: bottom; width: auto; min-width: 5%; max-width: 100%; flex: 0 0 auto; height: auto; align-self: flex-end; box-sizing: border-box;">
                    <section style="text-align: left; box-sizing: border-box;">
                        <section style="display: inline-block; width: 61px; height: 61px; vertical-align: top; overflow: hidden; background-image: linear-gradient(rgb(121, 181, 229) 13%, rgb(72, 211, 208) 88%); box-sizing: border-box;">
                        </section>
                    </section>
                </section>
            </section>
        </section>
    </section>
</section>
```

### 13. 嵌套引用块

适用于：多层次内容

```html
<section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: 10px 0px 0px; box-sizing: border-box;">
    <section style="display: inline-block; width: 100%; vertical-align: top; align-self: flex-start; flex: 0 0 auto; background-color: rgb(225, 240, 253); padding: 0px; box-sizing: border-box;">
        <section style="margin: 10px 0px 0px; box-sizing: border-box;">
            <section style="background-color: rgb(120, 185, 226); height: 1px; box-sizing: border-box;">
                <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
            </section>
        </section>
        <section style="justify-content: flex-start; display: flex; flex-flow: row; margin: -10px 0px; box-sizing: border-box;">
            <section style="display: inline-block; width: auto; vertical-align: top; align-self: flex-start; flex: 100 100 0%; border-left: 1px solid rgb(120, 185, 226); border-right: 1px solid rgb(120, 185, 226); height: auto; margin: 0px 10px; padding: 30px 10px; box-sizing: border-box;">
                <section style="text-align: justify; font-size: 15px; line-height: 1.8; letter-spacing: 1.6px; padding: 0px; box-sizing: border-box;">
                    <p style="text-indent: 2.2133em; white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
                        <span leaf="">内容</span>
                    </p>
                </section>
            </section>
        </section>
        <section style="margin: 0px 0px 10px; box-sizing: border-box;">
            <section style="background-color: rgb(120, 185, 226); height: 1px; box-sizing: border-box;">
                <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
            </section>
        </section>
    </section>
</section>
```

## END 结束标识

### 1. 标准 END 标签

```html
<section style="display: flex; flex-flow: row; margin: 10px 0%; text-align: center; justify-content: center; isolation: isolate; box-sizing: border-box;">
    <section style="display: inline-block; vertical-align: middle; width: auto; flex: 100 100 0%; height: auto; align-self: center; box-sizing: border-box;">
        <section style="margin: 0px 0%; box-sizing: border-box;">
            <section style="background-color: rgb(120, 185, 226); height: 1px; box-sizing: border-box;">
                <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: top; width: auto; flex: 0 0 0%; align-self: stretch; height: auto; line-height: 0; padding: 0px; box-sizing: border-box;">
        <section style="transform: perspective(0px); transform-style: flat; box-sizing: border-box;">
            <section style="transform: rotateY(180deg); box-sizing: border-box;">
                <section style="display: inline-block; width: 0px; height: 0px; vertical-align: top; overflow: hidden; border-style: solid; border-width: 4px 2px; border-color: rgb(120, 185, 226) rgba(255, 255, 255, 0) rgba(255, 255, 255, 0) rgb(120, 185, 226); box-sizing: border-box;">
                </section>
            </section>
        </section>
        <section style="transform: perspective(0px); transform-style: flat; box-sizing: border-box;">
            <section style="transform: rotateX(180deg) rotateY(180deg); box-sizing: border-box;">
                <section style="display: inline-block; width: 0px; height: 0px; vertical-align: top; overflow: hidden; border-style: solid; border-width: 4px 2px; border-color: rgb(120, 185, 226) rgba(255, 255, 255, 0) rgba(255, 255, 255, 0) rgb(120, 185, 226); box-sizing: border-box;">
                </section>
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: top; width: auto; flex: 0 0 auto; align-self: stretch; min-width: 10%; max-width: 100%; height: auto; background-color: rgb(120, 185, 226); border-width: 0px; box-sizing: border-box;">
        <section style="line-height: 1.4; color: rgb(255, 255, 255); padding: 0px 10px; font-size: 11px; box-sizing: border-box;">
            <p style="margin: 0px; padding: 0px; box-sizing: border-box;">
                <strong style="box-sizing: border-box;"><span leaf="">END</span></strong>
            </p>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: top; width: auto; flex: 0 0 0%; align-self: stretch; height: auto; line-height: 0; padding: 0px; box-sizing: border-box;">
        <section style="box-sizing: border-box;">
            <section style="display: inline-block; width: 0px; height: 0px; vertical-align: top; overflow: hidden; border-style: solid; border-width: 4px 2px; border-color: rgb(120, 185, 226) rgba(255, 255, 255, 0) rgba(255, 255, 255, 0) rgb(120, 185, 226); box-sizing: border-box;">
            </section>
        </section>
        <section style="transform: perspective(0px); transform-style: flat; box-sizing: border-box;">
            <section style="transform: rotateX(180deg); box-sizing: border-box;">
                <section style="display: inline-block; width: 0px; height: 0px; vertical-align: top; overflow: hidden; border-style: solid; border-width: 4px 2px; border-color: rgb(120, 185, 226) rgba(255, 255, 255, 0) rgba(255, 255, 255, 0) rgb(120, 185, 226); box-sizing: border-box;">
                </section>
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: middle; width: auto; flex: 100 100 0%; height: auto; align-self: center; box-sizing: border-box;">
        <section style="margin: 0px 0%; box-sizing: border-box;">
            <section style="background-color: rgb(120, 185, 226); height: 1px; box-sizing: border-box;">
                <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
            </section>
        </section>
    </section>
</section>
```

### 2. 带描边的 END 文字

```html
<section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: 10px 0px; box-sizing: border-box;">
    <section style="display: inline-block; vertical-align: middle; width: auto; flex: 100 100 0%; height: auto; align-self: center; padding: 0px 10px 0px 0px; box-sizing: border-box;">
        <section style="margin: 0.5em 0px; box-sizing: border-box;">
            <section style="border-top: 1px dashed rgb(15, 76, 117); box-sizing: border-box;">
                <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: middle; width: auto; align-self: center; flex: 0 0 auto; min-width: 5%; max-width: 100%; height: auto; padding: 0px 10px; box-sizing: border-box;">
        <section style="margin: 0px; box-sizing: border-box;">
            <section style="color: rgb(255, 255, 255); letter-spacing: 4px; text-shadow: rgb(22, 93, 155) 1px -1px 0px, rgb(22, 93, 155) 1px 1px 0px, rgb(22, 93, 155) -1px 1px 0px, rgb(22, 93, 155) -1px -1px 0px, rgb(22, 93, 155) 1px 0px 0px, rgb(22, 93, 155) 0px 1px 0px, rgb(22, 93, 155) -1px 0px 0px, rgb(22, 93, 155) 0px -1px 0px; text-align: center; box-sizing: border-box;">
                <p style="margin: 0px; padding: 0px; box-sizing: border-box;">
                    <strong style="box-sizing: border-box;"><span leaf="">END</span></strong>
                </p>
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: middle; width: auto; align-self: center; flex: 100 100 0%; height: auto; padding: 0px 0px 0px 10px; box-sizing: border-box;">
        <section style="margin: 0.5em 0px; box-sizing: border-box;">
            <section style="border-top: 1px dashed rgb(15, 76, 117); box-sizing: border-box;">
                <svg viewBox="0 0 1 1" style="float: left; line-height: 0; width: 0px; vertical-align: top;"></svg>
            </section>
        </section>
    </section>
</section>
```

### 3. 圆形标签 END

```html
<section style="display: flex; width: 100%; flex-flow: column; box-sizing: border-box;">
    <section style="z-index: auto; box-sizing: border-box;">
        <section style="text-align: center; justify-content: center; display: flex; flex-flow: row; margin: 0px; box-sizing: border-box;">
            <section style="display: inline-block; width: auto; vertical-align: top; align-self: flex-start; flex: 0 0 auto; background-color: rgb(22, 93, 155); border-radius: 100%; overflow: hidden; min-width: 5%; max-width: 100%; height: auto; padding: 3px 15px; box-sizing: border-box;">
                <section style="text-align: justify; font-size: 11px; color: rgb(255, 255, 255); letter-spacing: 1px; box-sizing: border-box;">
                    <p style="white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;">
                        <span leaf="">END</span>
                    </p>
                </section>
            </section>
        </section>
    </section>
</section>
```

## 底部信息卡

### 1. 标准底部信息卡

```html
<section style="text-align: center; justify-content: center; display: flex; flex-flow: row; margin: 15px 0px; box-sizing: border-box;">
    <section style="display: inline-block; width: auto; vertical-align: top; align-self: flex-start; flex: 100 100 0%; border-style: solid; border-width: 1px; border-color: rgb(189, 203, 212); padding: 0px 6px; height: auto; box-sizing: border-box;">
        <section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: -7px 0px; width: 100%; align-self: flex-start; background-color: rgb(245, 245, 245); padding: 6px 0px; box-sizing: border-box;">
            <section style="justify-content: flex-start; display: flex; flex-flow: row; width: 100%; align-self: flex-start; border-top: 1px solid rgb(189, 203, 212); border-bottom: 1px solid rgb(189, 203, 212); padding: 15px 20px; box-sizing: border-box;">
                <section style="font-size: 14px; line-height: 1.6; width: 100%; box-sizing: border-box;">
                    <p style="margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">图文 | 作者</span></p>
                    <p style="margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">编辑 | 编辑者</span></p>
                    <p style="margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">责编 | 责任编委</span></p>
                    <p style="margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">审核 | 审核人</span></p>
                </section>
            </section>
        </section>
    </section>
</section>
```

### 2. 带左侧蓝条的底部信息卡

```html
<section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: 0px 0px 10px; width: 100%; align-self: flex-start; border-style: solid; border-width: 1px; border-color: rgb(95, 156, 239); padding: 5px; box-sizing: border-box;">
    <section style="justify-content: flex-start; display: flex; flex-flow: row; width: 100%; align-self: flex-start; padding: 20px 10px; background-color: rgb(242, 247, 252); box-sizing: border-box;">
        <section style="text-align: justify; font-size: 14px; width: 100%; box-sizing: border-box;">
            <p style="white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">图文 | 作者</span></p>
            <p style="white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">编辑 | 编辑者</span></p>
            <p style="white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">责编 | 责任编委</span></p>
            <p style="white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">审核 | 审核人</span></p>
        </section>
    </section>
</section>
```

### 3. 带四角装饰的底部信息卡

```html
<section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: 0px; box-sizing: border-box;">
    <section style="display: inline-block; vertical-align: top; width: 50%; box-sizing: border-box;">
        <section style="box-sizing: border-box;">
            <section style="display: inline-block; width: 7px; height: 7px; vertical-align: top; overflow: hidden; padding: 0px; border-style: solid; border-width: 0px; border-radius: 82px; background-color: rgb(75, 143, 120); box-sizing: border-box;">
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: top; width: 50%; align-self: flex-start; flex: 0 0 auto; box-sizing: border-box;">
        <section style="text-align: right; box-sizing: border-box;">
            <section style="display: inline-block; width: 7px; height: 7px; vertical-align: top; overflow: hidden; padding: 0px; border-style: solid; border-width: 0px; border-radius: 82px; background-color: rgb(75, 143, 120); box-sizing: border-box;">
            </section>
        </section>
    </section>
</section>
<section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: -4px 0px; box-sizing: border-box;">
    <section style="display: inline-block; width: auto; vertical-align: top; align-self: flex-start; flex: 100 100 0%; border-style: solid; border-width: 1px; border-color: rgba(75, 143, 120, 0.49); height: auto; margin: 0px 3px; background-color: rgb(236, 249, 243); padding: 24px; box-sizing: border-box;">
        <section style="text-align: justify; font-size: 14px; box-sizing: border-box;">
            <p style="white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">图文 | 作者</span></p>
            <p style="white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">编辑 | 编辑者</span></p>
            <p style="white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">责编 | 责任编委</span></p>
            <p style="white-space: normal; margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">审核 | 审核人</span></p>
        </section>
    </section>
</section>
<section style="text-align: left; justify-content: flex-start; display: flex; flex-flow: row; margin: 0px; box-sizing: border-box;">
    <section style="display: inline-block; vertical-align: top; width: 50%; box-sizing: border-box;">
        <section style="box-sizing: border-box;">
            <section style="display: inline-block; width: 7px; height: 7px; vertical-align: top; overflow: hidden; padding: 0px; border-style: solid; border-width: 0px; border-radius: 82px; background-color: rgb(75, 143, 120); box-sizing: border-box;">
            </section>
        </section>
    </section>
    <section style="display: inline-block; vertical-align: top; width: 50%; align-self: flex-start; flex: 0 0 auto; box-sizing: border-box;">
        <section style="text-align: right; box-sizing: border-box;">
            <section style="display: inline-block; width: 7px; height: 7px; vertical-align: top; overflow: hidden; padding: 0px; border-style: solid; border-width: 0px; border-radius: 82px; background-color: rgb(75, 143, 120); box-sizing: border-box;">
            </section>
        </section>
    </section>
</section>
```

### 4. 虚线边框底部信息卡

```html
<section style="margin: 10px 0px; text-align: center; box-sizing: border-box;">
    <section style="display: inline-block; width: 100%; border: 2px dashed rgb(189, 203, 212); padding: 10px; background-color: rgb(242, 247, 252); border-radius: 0.7em; box-sizing: border-box;">
        <section style="font-size: 14px; width: 100%; box-sizing: border-box;">
            <p style="margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">图文 | 作者</span></p>
            <p style="margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">编辑 | 编辑者</span></p>
            <p style="margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">责编 | 责任编委</span></p>
            <p style="margin: 0px; padding: 0px; box-sizing: border-box;"><span leaf="">审核 | 审核人</span></p>
        </section>
    </section>
</section>
```

## 边框与圆角

### 边框样式

| 类型 | CSS |
|------|-----|
| 实线边框 | `border: 1px solid rgb(95, 156, 239)` |
| 虚线边框 | `border: 1px dashed rgb(189, 203, 212)` |
| 重点卡片边框 | `border-width: 1px 6px 6px 1px; border-color: rgb(22, 93, 155)` |
| 侧边线 | `border-left: 4px solid rgb(22, 93, 155)` |
| 底部线 | `border-bottom: 3px solid rgb(222, 54, 54)` |
| 顶部线 | `border-top: 1px dashed rgb(15, 76, 117)` |

### 圆角样式

| 类型 | CSS |
|------|-----|
| 小圆角 | `border-radius: 0.7em` |
| 中圆角 | `border-radius: 10px` |
| 大圆角 | `border-radius: 14px` |
| 圆形 | `border-radius: 100%` |
| 胶囊形 | `border-radius: 82px` |

### 阴影样式

| 类型 | CSS |
|------|-----|
| 柔和阴影 | `box-shadow: rgb(204, 204, 204) 0.2em 0.2em 0.3em` |
| 蓝色阴影 | `box-shadow: rgb(199, 225, 247) 0px 5px 13px -4px` |

## 渐变背景

### 渐变样式

```css
/* 横向渐变 */
background-image: linear-gradient(90deg, rgba(83, 209, 232, 0.16) 13%, rgba(37, 72, 255, 0.1) 88%);

/* 蓝青渐变（角标用） */
background-image: linear-gradient(rgb(43, 158, 228) 13%, rgb(0, 210, 192) 88%);
background-image: linear-gradient(rgb(121, 181, 229) 13%, rgb(72, 211, 208) 88%);
```

## 公众号特有 Class

| Class | 用途 |
|-------|------|
| `rich_pages wxw-img` | 图片 |
| `js_underline_content` | 编辑器内容区 |
| `autoTypeSetting24psection` | 自动排版设置 |
| `rich_media_content` | 文章内容区 |
| `__bg_gif` | 背景 GIF 图标 |
| `ProseMirror` | 编辑器内容区 |
| `ProseMirror-trailingBreak` | 尾部换行符 |
| `ProseMirror-separator` | 图片分隔符 |

## 注意事项

1. **所有 section 必须闭合**，嵌套层次清晰
2. **必须包含 `box-sizing: border-box`**，确保尺寸计算正确
3. **SVG 占位符**：装饰性 section 内通常包含空的 SVG，保持结构完整
4. **避免使用纯黑色文字**，使用 `rgb(62, 62, 62)` 更柔和
5. **图片宽度**：使用 `max-width: 100%; width: 100%` 自适应
6. **图注字号**：使用 `14px`，行高 `1.6`，字间距 `0px`
7. **首行缩进**：使用 `2.2133em` 或 `2.2em`
8. **visibility: visible**：确保元素可见
9. **leaf 属性**：文本内容应放在 `<span leaf="">` 标签内
10. **三角形绘制**：使用 CSS border 技术，左右需要镜像翻转

## 更新日志

- 2026-04-24: 初始版本
- 2026-04-24 更新：加入更多卡片模板、END 标识、底部信息卡
- 2026-04-24 更新：基于5个实际发布范本扩展，新增分隔线、数字标号卡片、引用装饰、渐变角标卡片等12种新模板

---
> Source: [makabaka11/wechat-article-style.skill](https://github.com/makabaka11/wechat-article-style.skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->

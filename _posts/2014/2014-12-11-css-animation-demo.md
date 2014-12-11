---
layout: post
title: 纯CSS3实现的动画
categories:
- 编程与开发
tags:
- CSS
---

CSS3的animation是一个比较强大的功能，通过指定不同的帧，浏览器绘制引擎会自动使用插值算法将动画效果表现出来。下面分享一个有意思的效果：是一个旋转的八卦，使用纯CSS3实现，需要使用支持标准CSS3的浏览器浏览。

<style>
    .outter-box {
        position: relative;
        margin: 5px auto;
        width: 300px;
        height: 300px;
    }
    @-webkit-keyframes rotate {
      0% {
        transform: rotate(0deg);
      }
      50% {
        transform: rotate(180deg);
      }
      100% {
        transform: rotate(360deg);
      }
    }
    .eight-diagrams {
      box-sizing: initial;
      position: absolute;
      top: 100px;
      left: 100px;
      width: 96px;
      height: 48px;
      background: #eee;
      border-style: solid;
      border-width: 2px 2px 48px 2px;
      border-color: red;
      border-radius: 100%;
    }
    .eight-diagrams:before, .eight-diagrams:after {
      box-sizing: initial;
      content: "";
      position: absolute;
      background: #eee;
      border: 18px solid red;
      border-radius: 100%;
      width: 12px;
      height: 12px;
    }
    .eight-diagrams:before {
      top: 50%;
      left: 0;
      border-color: red;
    }
    .eight-diagrams:after {
      top: 50%;
      left: 50%;
      border-color: #eee;
    }
    .eight-diagrams:hover {
      -webkit-animation: rotate 2s linear infinite;
    }
</style>

<div class="outter-box">
    <div class="eight-diagrams"></div>
</div>

其CSS代码如下：

    @-webkit-keyframes rotate {
      0% {
      transform: rotate(0deg);
      }
      50% {
        transform: rotate(180deg);
      }
      100% {
      transform: rotate(360deg);
      }
    }
    .eight-diagrams {
      position: absolute;
      top: 100px;
      left: 100px;
      width: 96px;
      height: 48px;
      background: #eee;
      border-style: solid;
      border-width: 2px 2px 48px 2px;
      border-color: red;
      border-radius: 100%;
    }
    .eight-diagrams:before, .eight-diagrams:after {
      content: "";
      position: absolute;
      background: #eee;
      border: 18px solid red;
      border-radius: 100%;
      width: 12px;
      height: 12px;
    }
    .eight-diagrams:before {
      top: 50%;
      left: 0;
      border-color: red;
    }
    .eight-diagrams:after {
        top: 50%;
        left: 50%;
        border-color: #eee;
    }
    .eight-diagrams {
      -webkit-animation: rotate 2s linear infinite;
    }

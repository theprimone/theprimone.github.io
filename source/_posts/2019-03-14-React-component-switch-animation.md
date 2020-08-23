---
layout: post
title:  React 组件切换动画有感
categories:
  - JavaScript
  - React
tags:
  - 组件
  - 动画
author: zpr1g
date: 2019-3-14 20:00:00
updated: 2019-4-8 09:58:34
---

![](https://cloud.githubusercontent.com/assets/100200/23694858/180a1a78-0391-11e7-92a4-99acec2280f6.gif)

今天又拿出了搁置已久的 [eler](https://www.easylearner.app) 项目，好不容易写到现在──用户登录注册了。之前做到了需要实现一个页面内部组件切换的动画来切换输入邮箱、输入验证码、输入注册信息的三步走的 fade 动画。因此，折腾了好久怎么使用 TransitionGroup 来切换。

<!-- more -->

做出来的效果都很尴尬，第一次尝试使用数组存储三个组件，根据步骤取出一个组件来渲染，这样做会使一瞬间父组件变宽（没整明白咋回事……），即将消失和即将进入的组件同时发生，由此高度也会一瞬间变高，体验极差。第二次便去 google 了一下，提到一个用绝对定位的方式来避免高度上的异常，测试发现绝对定位会让组件脱离文档流……就没继续了，体验好像也不对头啊。

终于在第三次，我仔细研究了一下 google 的登录逻辑，发现它是通过路由来做到输入邮箱动画转到输入密码的组件，打开 Rendering 工具也确实验证了这个思路应该是没问题的，毕竟我之前都做了页面切换动画和左右布局页面的右侧路由切换动画。所以，综上所述，某个组件内部的组件切换动画应该也是采用路由切换动画的方式来实现。

这么说来，页面及组件切换动画都充分利用了 React 的 diff 算法啊，这才反应过来啊，23333。话说 google 页面不是用的 React 的话，那么是不是得挖掘更深层次的原理啊，我好方，留个坑先。

现在也不早了（22:29），明天再验证这个想法吧。天知道我当时是怎么想的，第一反应居然是去使用 TransitionGroup 直接实现组件切换动画。

#### 2019-03-17 续

```jsx
class AppSignUp extends Component {

    render() {
        const timeout = { enter: 225, exit: 195 };
        return (
            <AppSignUpCompt>
                <TransitionGroup>
                    <CSSTransition
                        key={transitionKey}
                        classNames="fade"
                        appear
                        timeout={timeout}
                    >
                        <Switch location={location}>
                            {/* {routes.map((route) => <RouteWithSubRoutes key={route.path} {...route} />)} */}
                            <Route exact path='/user/sign-up/identifier' render={() => <EmailCheck />} />
                            <Route exact path='/user/sign-up/verify-code' render={() => <VerifyCodeCheck />} />
                            <Route exact path='/user/sign-up/setup' render={() => <SignUpSetup />} />
                        </Switch>
                    </CSSTransition>
                </TransitionGroup>
            </AppSignUpCompt>
        );
    }
}
```

书接上回，说干就干，开始研究基于路由的组件切换动画了，验证了一番，代码如上，如果使用嵌套路由配置会导致页面一片空白，打开控制台能看到一直在刷新页面_(:3J∠)_，没整明白，手动配置一番方才页面正常。可惜，最终还是出现了以前路由切换页面的迷之 bug ，后来莫名就好了，这还导致我都不知道怎么描述，折腾来折腾去，还是回到 [github 官方仓库](https://github.com/ReactTraining/react-router)寻找答案，issue 中搜索一下 transition group ，终于发现了具体描述跟 [Animating with \<Switch\> using previous pathname](https://github.com/ReactTraining/react-router/issues/4351) 一样的情况，但是这都是 17 年初的 issue 了啊 :expressionless:

貌似解决方法是[这个](https://github.com/ReactTraining/react-router/issues/4351#issuecomment-281196606)，但是折腾了两三天的我实在不想再折腾了，感觉头都要炸了。于是乎，先略过动画切换吧（没动画效果切换一切正常），先把逻辑搞完先。

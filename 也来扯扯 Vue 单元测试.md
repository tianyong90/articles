# 也来扯扯 Vue 单元测试

![u happy](./images/we-vue-coverage.jpg)

从使用 Vue 写出第一个 `Hello world` 到现在已经有近两年时间了，期间利用业余时间折腾了一套组件 we-vue，起初是出于实践学到的新知识，更多的是玩的意思，不过后来维护的过程中渐渐积累了一些经验，并开始享受这种过程。

在 we-vue 更新到 v2.0 的时候，开始全面地编写单元测试。起先使用 karma + mocha + chrome-headless 这种组合完成的行级覆盖率达到 96% 的测试。但最近，我又放弃了这种组合，转而使用 Jest。在这连番的折腾中，入过不少坑（当然，很多时候是自己挖坑自己跳），也解锁了不少新姿势。

本文主要扯一扯自己在完成这些单元测试，以及迁移到 Jest 过程中的一些收获。文中并不会涉及非常具体的测试写法，因为这些教程官方文档已经做得很好了。希望文中的一些内容对于正准备做 Vue （其实也不仅限于 Vue） 单元测试的人能有所帮助。

## 为什么要做单元测试

作为一个程序员，单元测试或许是一个绕不开的坎。我大致总结了一下当初决定做单元测试的原因：

1. 有单元测试的项目，显得更专业，更有 B 格。真的，认真回忆下，这确实是当初自己的首要动机，可能这算是动机不纯吧？ :smile:

2. 懒！受不了每次调整之后，得不断地检查代码，甚至查看页面源码是否符合预期。不断修改各种参数并刷新以测试不同情况下的结果。而这里面的一大部分工作其实可以让单元测试来完成。所以说，懒人让世界更美好！

3. 单元测试能避免出现一些代码运行结果与预期不符的错误，通常是一些比较低级但又难以发现的问题。

4. 单元测试能够避免在升级更新、修复 BUG 的时候引入一些意料之外的问题。**有时候自以为小修改小优化无大碍，其实不然！**

5. 单元测试对提高代码质量很有帮助。因为，好的代码一般是便于测试的。如果在进行单元测试过程中发现自己的一些代码不方便进行测试，那么你可能需要重新审视这些代码，看是否有一些设计上不合理或者可以优化的地方。**当然，这也并不是说代码应该“迁就”于单元测试，如果这样就有点儿本末倒置了。**

总之，单元测试能提高程序的可靠性，让开发者在发布时更有底气，让使用者更有安全感。虽然编写单元测试需要花费一些时间，但相比于它所带来的优势，这些时间和精力上的花费还是值得的。

另外值得注意的是，单元测试并不能完全代替功能测试，因为程序本身设计的逻辑错误或者其它的一些环境因素所造成的影响，单元测试可能无能为力。所以，单元测试只是保证你想让程序模块输出一只猪，它不会整出一头驴来。至于进一步的功能测试或者说“肉测”，仍然是有必要的。

## 用到的一些工具（软件）

#### 趁手的家伙 -- WebStorm / Visual Studio Code

去打群架，得操个趁手的家伙，板砖极好，大木头棒子也不错。

而写代码，一个好用的 IDE 或者编辑器，能让效率飞升。就我个人而言，做前端时大部分时间使用 WebStorm，其本身对 Vue.js 就有很好的支持（内置了相关的插件）同时也支持的各种测试框架，适当的配置之后，可以很方便的进行断点、查看规模之类的调试工作。

此外，Visual Studio Code 也是个不错的选择，目前已不有少 Vue.js 开发和测试相关的插件了，只需要搜索加点击但可安装。

> 当然，这里我无意挑起各种 IDE 或者编辑器流派之争端，提到的两个只是自己个人喜好。大家可自行选择合适的，甚至只要自己喜欢且觉得方便，用记事本开干也没问题。

![u happy](./images/ni-kai-xin-jiu-hao.jpg)

#### 用成熟好用的测试工具库 -- vue-test-utils

[vue-test-utils](https://github.com/vuejs/vue-test-utils) 是 Vue 生态圈中的一个开源项目，其前身是 [avoriaz](https://github.com/eddyerburgh/avoriaz)，avoriaz 也是一个不错的包，但其 README 中有说明，当 vue-test-utils 正式发布的时候， 它将会被废弃。

vue-test-utils 能极大地简化 Vue.js 单元测试。

例如，网上一搜 Vue 单元测试，得到的例子一般是像下面这样的（包括 vue-cli 提供的模板里默认也是这样）：

```js
import Vue from 'vue'
import HelloWorld from '@/components/HelloWorld'

describe('HelloWorld.vue', () => {
  it('should render correct contents', () => {
    const Constructor = Vue.extend(HelloWorld)
    const vm = new Constructor().$mount()
    expect(vm.$el.querySelector('.hello h1').textContent)
      .toEqual('Welcome to Your Vue.js App')
  })
})
```

使用 vue-test-utils 后，你可以像下面这样

```js
import { shallow } from '@vue/test-utils'
import HelloWorld from '@/components/HelloWorld'

describe('HelloWorld.vue', () => {
  it('should render correct contents', () => {
    const wrapper = shallow(HelloWorld, {
      attachToDocument: ture
    })

    expect(wrapper.find('.hello h1').text()).to.equal('Welcome to Your Vue.js App')
  })
})
```

可以看到代码更加简洁了。wrapper 内含许多有用的方法，上面的例子中所使用的 find() 其中最简单不过的一个。vue-test-utils 还有 createLocalVue() 等方法以及 stub 之类的功能，基本上可以完成绝大部分情况下的测试用例。

> 需要注意的是，截至日前（2018-03-21）仍然处于 Beta 阶段。在正式版发布之前可能会有大的更改，例如新增或废弃一些方法。同时也可能存在一些 BUG（自己就曾[修复过一个](https://github.com/vuejs/vue-test-utils/issues/263) :smile:）。但目前总体来说已趋于稳定，推荐使用，需要留意其最新更改。

#### 选择一个好用的断言库

通常是 chai，有时候结合 sinon 一起使用。chai 是一个优秀的库，里面的方法十分完善。网上相关的教程更是不计其数，这也反映出它很受欢迎。

不过个人并不太中意 chai 的语法，比如比较常用的 `to.be.ok`,`to.not.be.ok`,`expect({a: 1, b: 2}).to.be.an('object').that.has.all.keys('a', 'b')`，为啥要那么长？为啥要那么多句点？顺序忘了怎么弄？

![are you ok](./images/are-you-ok.jpg)

所以一开始我就选择了 expect.js (expect 是 Jest 的一部分，可以单独安装使用)，主要是它的语法更符合我的口味，这也为后期迁移到 Jest 省了不少事。

#### 一个合适测试框架 -- Jest

这里只提到了 Jest，当然也是个人喜好而已，这也是自己最终决定的方案。当然此前使用的 karma + mocha + chai + chrome... 那一套也有其适用场景和可取之处。后面将会提到 Jest 的一些优点和缺点。

## 利用 CI 服务自动进行单元测试、构建以及发布

现在已经有不少平台提供 CI 服务，例如 TravisCI 和 CircleCI。对于开源的项目，能免费使用这些平台的服务持续集成一些日常构建、测试工作。

 自己目前使用 CircleCI，具体原因就不多说了，使用哪个取决于自身喜好和具体业务情况，甚至可以考虑自己搭建 CI 服务器。

## 为自己的项目加入测试覆盖率徽标

在自己开源项目的 README 中加入一个显示单元测试覆盖率的徽标，会增进用户的第一印象。高覆盖率的徽标，会使项目显得更专业可靠，也能让用户进一步了解整个项目并最终选用。

CodeCov 能提供这种服务，并可以结合前面提到的 CI 使用，通过 CI 在代码推送后自动执行单元测试，通过后将代码覆盖率相关数据发送给 CodeCov，这样，在 README 中加入的覆盖率徽标就能自动更新了。

为此，你需要一个 codecov 账号（通常用 GitHub 账号登录即可）并安装 `codecov` 包

```shell
$ yarn add -D codecov
```

然后在 CI 的任务配置里加入上传代码测试覆盖率数据的步骤，例如 CircleCI 配置如下：

```yaml
steps:
    ## 前面部分任务省略

    # run tests!
    - run: yarn test

    # update codecov stats
    - run: ./node_modules/.bin/codecov
```

最后在 README 里加入微标图片就可以了,就像这样。

![code cov](https://img.shields.io/codecov/c/github/tianyong90/we-vue/master.svg)


## Jest 相对于 karma + mocha + Chrome 组合的优缺点

前面提到，我最终转向了使用 Jest，这并非一时脑热，而是经过多次权衡和尝试之后才作的决定。主要是由于 Jest 相对于之前的方案有着不少优势，一些特性让测试变得更轻松愉快，更有效率。

我大致做了下对比，粗略总结如下：

### 优点

1. 一站式的解决方案

    在使用 Jest 之前，我需要一个测试框架（mocha），需要一个测试运行器（karma），需要一个断言库（chai），需要一个用来做 spies/stubs/mocks 的工具（sinon 以及 sinon-chai 插件），一个用于测试的浏览器环境（可以是 Chrome 浏览器，也可以用 PhantomJS）。

    而使用 Jest 后，只要安装它，全都搞定了。

2. 全面的官方文档，易于学习和使用

    Jest 的官方文档很完善，对着文档很快就能上手。而在之前，我需要学习好几个插件的用法，至少得知道 mocha 用处和原理吧 我得学会 karma 的配置和命令，chai 的各种断言方法……，经常得周旋于不同的文档站之间，其实是件很烦也很低效的事。

3. 配置简单方便

4. 更直观明确的测试信息提示

5. 方便的命令行工具

    全局安装 Jest 后，可以在命令行执行单元测试，配合各种命令参数，可以方便地实现执行单个测试、监视文件变化并自动执行等功能。特别是对于监视文件变化并执行，它提供多种模式，可以只执行修改过的测试。记得初次读到这部分文档时，我不禁仰天长啸，竟然有这么骚气凌人的操作？

    > Jest 甚至提供了 jest-codemods 这一工具，用来将使用其它包的测试迁移为使用 Jest

### 缺点

1. jsdom 的一些局限性
    
    因为 Jest 是基于 jsdom 的，jsdom 毕竟不是真实的浏览器环境，它在测试过程中其实并不真正的“渲染”组件。这会导致一些问题，例如，如果组件代码中有一些根据实际渲染后的属性值进行计算（比如元素的 clientWidth）就可能出问题，因为 jsdom 中这些参数通常默认是 0.
    
    所以有些情况下，测试中可能要施以一些骚操作，比如自行 mock（实例上就是伪造，但合理地伪造）一些中间值，来满足测试用例。如果你的项目中这样的情况很多，还是建议使用 karma + mocha + chrome 这一组合。

2. 周边相关的包可能还不完善

    例如 vue-jest，目前的版本并不能完全实现 vue-loader 的功能。比如，使用 sass，postcss 之类的功能，它会抛出警告信息。代码中直接 import 实际的 css 文件，则有可能报错，这时则需要使用 mock 来模拟 css 文件。这些问题，在使用 karma-mocha Chrome 的时候是没有的，因为测试运行于真实的浏览器环境中。

## ChromeHeadless vs. PhantomJS?

较新版本的 Chrome 支持以 headless 模式运行，这对于测试这种不需要显示界面的任务来说是很合适了（其实也可以使用常规模式，只不过执行测试的时候 Chrome 会弹出窗口）。

而在 Chrome 推出 headless 模式功能之前。我们通常用 PhantomJS 的 headless WebKit 环境来进行测试，但它有着一些久未解决的问题，而且更新进度越来越慢。不欠前（2018-03-05），因为开发组内部意见不合，PhantomJS 项目已经封存了代码暂停开发了。

Chrome headless 对于 PhantomJS 来说算是一个致命的打击，特别是 
Chrome 官方推出的 puppeteer 在短时间内已经被广泛接受和使用。但其实 PhantomJS 还是有一些适用场景的，例如一些服务器并不支持 Chrome，这种情况下 PhantomJS 就有用武之地了。不过目前看来，对手的碾压以及自身维护团队的涣散，让我有理由放弃它了。

## 后记

实践总是最有效率的学习方式，不停地折腾才能不断的进步，特别是对于编程这事上，每天都有新的东西出现。

编写单元测试可能比较枯燥，因为它并不像做新功能一样让人兴奋。但只要耐心调试，当全部测试用例都通过，当最后测试覆盖率慢慢提升时，那种成就感也不亚于开发出了新功能！

## 广告

最后，为自己的 we-vue 打个小广告，虽然目前不成气候，也还有不少需要完善的地方。但自己会做下去，也希望能有更多的人支持和参与。里面可以看到一些觉的组件测试套路，目前组件部分的单元测试覆盖率已经超过 99%。

项目地址: [https://github.com/tianyong90/we-vue](https://github.com/tianyong90/we-vue)  
在线文档: [https://wevue.org](https://wevue.org)  
在线示例: [https://demo.wevue.org](https://demo.wevue.org)


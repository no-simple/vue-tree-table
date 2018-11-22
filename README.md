# tree table

> 基于vue和element-ui中table实现的tree table，传入树形接口数据即可。可无限层级。

# 详细说明文档

> [详细说明文档,已发表在掘金](https://juejin.im/post/5be797456fb9a04a0378bb91)

# demo

> 预览地址：[tree table](https://no-simple.github.io/vue-tree-table/)

# 前言
 如何在table上实现一个可折叠展开子节点的table？先看下最终实现效果图：


![](https://user-gold-cdn.xitu.io/2018/11/11/16700adbccdfe9a8?w=758&h=453&f=gif&s=117653)
> 其实这个项目在两个月以前就以上上传在**github**了，但当时没有写详细的实现过程。自己前几天发表的一篇技术贴[当下拉列表数据过大时，该如何应对？](https://juejin.im/post/5be534086fb9a049ef2615e2)得到大家的不少支持，犹如歌曲《红日》里的歌词**像红日之火 点燃真的我**~  所以继续为掘金社区做奉献自己微不足道的力量啦 ~~

> 在周末听着嗨歌写文还是很nice的~  嗨起来 ~ ~


*  **项目地址：**： [源码传送门](https://github.com/no-simple/vue-tree-table)
*  **demo**： [demo传送门](https://no-simple.github.io/vue-tree-table/)
 * 顺便推一下：[为何要再封装 AJAX？](https://juejin.im/post/5be8d9026fb9a049a81ed6d6)

**技术栈**：
* **vue**
* **javascript**


# 动手实现
为有兴趣的同学先准备一下大纲目录预览，为了不让同学们看入迷，让大纲为你**指明前行的道路**！
## 大纲预览
1. 明确需求
2. 树形结构数据准备
3. **数据扁平化**（重点）
4. 层级展示
5. 折叠展开功能实现
## 1. 明确需求
> 实现一个可折叠展开的table，看上去很迷茫的样子。但实际上就是：
1. 在**table**上点击时对其**子集**进行**隐藏**或**显示**
2. 通过缩进的距离来表现层级关系

在代码里很东西其实都是伪装出来的，例如我们要实现的这个可无限折叠的table。但在用户操作的时候看来就是那么回事咯 ~ ~

## 2. 树形结构数据准备
这里已经准备好了树形结构的数据，存放于**data.js**的文件中，节点通过`Children`连接。如标题所说，可无限折叠，无论这里的树形数据有多少层级，都能给它办妥当了。先把牛吹在这，接下来看看如何实现。

![](https://user-gold-cdn.xitu.io/2018/11/11/16700e1b39e3a4d5?w=949&h=604&f=png&s=68729)
## 3. 数据扁平化
### 条件梳理
树形结构数据扁平化并不难，难点在于在扁平化的时候要做的处理。这里深入梳理一下：
> 1. 我们将一个树节点的所有父节点称为**family**，好比我们要知道的自己的大领导、二领导、直系领导...因为他们的命令咱们都得执行。在这个表单中就是当父节点或更高的祖辈节点发命令时，做一个懂事的子节点当然要听话办事。`__family`字段就是用来存放所有所有的父节点的数组。在扁平化数据是通过**数组字段**`__family`收集其所有**父节点**。这样一来，就像族谱一样，就能清晰知道该节点的所有父节点。**至于如和折叠收缩呢？**我们通过一个`foldList`数组来记录要**折叠的节点**，也称为**死亡名单**。如此一来，只要**包含父节点标识**的**节点**就乖乖的隐藏吧。

> 2. 每个成员都因该是唯一的，那么我们在遍历时都应该为每个成员添加一个唯一标识`__identity`。正是上一个步骤中说的**节点标志**。

> 3. 为了明确知道各个节点之间的层级关系，通过`__level`明确层级关系。那么我们规定`__level`的值越小等级越高。`__level=0`代表首层节点。

> 4. 既然是无限层级，肯定是用递归来实现了。`formatConversion()`方法实现递归。**那何时跳出当前遍历的递归呢**？当前节点没有子集时`Children.length = 0`的时候跳出本次递归，进行下一次递归。该方法可能需要花点时间理解，代码里注释已写的比较详细，有问题欢迎留言沟通~~
### 数据扁平化
```
    /*********************************
      ** Fn: formatConversion
      ** Intro: 将树形接口数据扁平化
      ** @params: parent 为当前累计的数组  也是最后返回的数组
      ** @params: children 为当前节点仍需继续扁平子节点的数据
      ** @params: index 默认等于0， 用于在递归中进行累计叠加 用于层级标识
      ** @params: family 装有当前包含元素自身的所有父级 身份标识
      ** @params: elderIdentity 父级的  唯一身份标识
      ** Author: zyx
    *********************************/
    formatConversion (parent, children, index = 0, family = [], elderIdentity = 'x') {
      // children如果长度等于0，则代表已经到了最低层
      // let page = (this.startPage - 1) * 10
      if (children.length > 0) {
        children.map((x, i) => {
          // 设置 __level 标志位 用于展示区分层级
          Vue.set(x, '__level', index)
          // 设置 __family 为家族关系 为所有父级，包含本身在内
          Vue.set(x, '__family', [...family, elderIdentity + '_' + i])
          // 本身的唯一标识  可以理解为个人的身份证咯 一定唯一。
          Vue.set(x, '__identity', elderIdentity + '_' + i)
          parent.push(x)
          // 如果仍有子集，则进行递归
          if (x.Children.length > 0) this.formatConversion(parent, x.Children, index + 1, [...family, elderIdentity + '_' + i], elderIdentity + '_' + i)
        })
      } return parent
    }
```
我们来对比一下扁平话的数据
* 原数据

![](https://user-gold-cdn.xitu.io/2018/11/11/1670127720f21baa?w=576&h=468&f=png&s=30068)
* 扁平化后的数据

![](https://user-gold-cdn.xitu.io/2018/11/11/1670126bd540cc3c?w=590&h=708&f=png&s=52645)
> 对比数据的前后，先明确（父节点： `Name: "App"`）,   （子节点：`Name: "企业查询"`）。我们先看看__level字段，分别对应0和1，没问题。`__family`包含了本身节点。`__identity`的个格式就是**前缀 x**加上在**数据中的位置**。例如节点**App**在数据中的位置是第一个节点，那对应的`__identity`即是`x_0`。**企业查询**位于**App**节点下的第一个元素，**则在父节点的标识的基础上追加一位0**，那对应的`__identity`即是`x_0_0`。其他依次类推。一切已准备妥当~~
## 4. 层级展示
如图所示：


![](https://user-gold-cdn.xitu.io/2018/11/11/1670138a32ede097?w=421&h=412&f=png&s=10595)
如果展示层级呢？利用css即可实现
* 字体图标准备： 这里先补充一点，这里涉及两个字体图标，图中红色框标注的，资源存放于src目录下的iconfont目录下中。**层级最低的字段时不需要展示该图标呢**，**该如何判断呢？** 通过判断当前节点是否还有子集`params.Children.length === 0`即可判断。若没有子集则不设置字体图标即可。**点击时**还需要对**图标进行切换**，这又如何实现呢？前台提到过的**死亡名单**`foldList`，如果该存在该名单中，代表该数据已经被折叠，那返回折叠图标。否则返回展开图标。如代码：
```
//    html
<i :class="toggleFoldingClass(scope.row)"></i>

//    js methods:

    /*********************************
      ** Fn: toggleFoldingClass
      ** Intro: 如果子集长度为0，则不返回字体图标。
      ** Intro: 如果子集长度为不为0，根据foldList是否存在当前节点的标识返回相应的折叠或展开图标
      ** Intro: 关于class说明：permission_placeholder返回一个占位符，具体查看class
      ** @params: params 当前行的数据对象
      ** Author: zyx
    *********************************/
    toggleFoldingClass (params) {
      return params.Children.length === 0 ? 'permission_placeholder' : (this.foldList.indexOf(params.__identity) === -1 ? 'iconfont icon-minus-square-o' : 'iconfont icon-plussquareo')
    },
```
* 层级展示： `__level`字段就是这时候发挥用处。`__level`值越大，则将`margin-left`值增大。如代码：
```
<p :style="`margin-left: ${scope.row.__level * 20}px;`">...
```
基本的样子已经有了，那接下来实现点击功能。
## 5. 折叠展开功能实现
### 记录点击的节点标识
通过**死亡名单**`foldList`来记录点击。点击事件在点击折叠展开图标时触发`toggleFoldingStatus(scope.row)`事件，前面也说过，`foldList`存在的标识，对于所有的子节点都要隐藏起来。如果`foldList`没有数据，则全部展开，万事大吉。代码如图所示，就是那么简单。
```
//    html
<i  @click="toggleFoldingStatus(scope.row)" class="permission_toggleFold" :class="toggleFoldingClass(scope.row)"></i>
//    js methods
    /*********************************
      ** Fn: toggleFoldingStatus
      ** Intro: 切换展开 还是折叠
      ** @params: params 当前点击行的数据
      ** Author: zyx
    *********************************/
    toggleFoldingStatus (params) {
      this.foldList.includes(params.__identity) ? this.foldList.splice(this.foldList.indexOf(params.__identity), 1) : this.foldList.push(params.__identity)
    },
```
### 折叠展开功能实现
万事俱备，只欠东风。最后一件要做的大事就是根据**死亡名单**`foldList`来进行显示和隐藏数据。
这里借助el-table中的`row-style`实现。同学们也可以自己实现。来看下官方的说明

![](https://user-gold-cdn.xitu.io/2018/11/11/1670161aef64144a?w=541&h=114&f=png&s=10525)
该方法就是为table中的每一行数据设置样式。
```
    /*********************************
      ** Fn: toggleDisplayTr
      ** Intro: 该方法会对每一行数据都做判断 如果foldList 列表中的元素 也存在与当前行的 __family列表中  则该行不展示
      ** @params:
      ** Author: zyx
    *********************************/
    toggleDisplayTr ({row, index}) {
      for (let i = 0; i < this.foldList.length; i++) {
        let item = this.foldList[i]
        // 如果foldList中元素存在于 row.__family中，则该行隐藏。  如果该行的自身标识等于隐藏元素，则代表该元素就是折叠点
        if (row.__family.includes(item) && row.__identity !== item) return 'display:none;'
      }
      return ''
    },
```
* 在来看看效果
![](https://user-gold-cdn.xitu.io/2018/11/11/167016505d5f276c?w=907&h=714&f=gif&s=224086)
# 锦上添花
如果数据太多的话，一个层级层级的去搜索确实也麻烦，所以那么我们再来添加两个按钮**全部折叠**和**全部展开**好了。
咨询分析一下，其实这个功能很简单。还是关于前面提到过的**死亡名单**`foldList`。
* **全部展开**： 如果`foldList`为空，则万事大吉，数据全部展开。
* **全部折叠**：我们的设计是只要存在于死亡名单的所有包含该标识的节点都要隐藏。那么只要将所有的首层节点添加进去就可以了。`this.foldList = this.tableListData.map(x => x.__identity)`即取出首层节点的标识。
# 结语
更复杂的需求是结合分页，当从其他页回到之前页时，之前页的折叠状态要依旧保持。这里就不在说明了，因为应用场景很少。
* 点赞给个鼓励哟~

For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).

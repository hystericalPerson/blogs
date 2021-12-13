---
title: antv x6+vue3 demo
date: 2021-12-10 11:21:23
tags:  [X6,vue3]
categories: 
    - [demo]
---

## 前言
一个自定义流程图编辑器，antv-X6 + vue3 实现。  
由于项目实现完挺久的，简单的记录一下
![demo页面](/images/img1.png)

## 说明
#### 项目地址
[项目地址](https://github.com/hystericalPerson/vue3-x6-editor-demo)
<https://github.com/hystericalPerson/vue3-x6-editor-demo>


#### 功能点
1. 左侧组件
   + 基本组件
   + 流程库
   + 节点库
2. 中间展示部分
3. 右侧关于节点的描述（双向绑定<code>node-data</code>中的数据）

#### 设计思路
按照页面布局，简单的划分3个组件。<code>graph</code>,<code>nodeDesc</code>,<code>dnd</code>

##### graph
1. 创建组件<code>graph</code>,并将其配置化代码存入<code>config.js</code>
```
editor.graph = new Graph({
    ...createDefaultSetting(props.graphId, `${props.graphId}map`)
})

createDefaultSetting为配置化文件的引入
```
2. 绑定需要的画布事件函数
```
// 绑定画布的键盘事件
const bindKey = () => {
    bindKeyCopy()
    bindKeyPaste()
    bindKeyShear()
    bindKeyUndo()
    bindKeyRedo()
    bindKeySelectAll()
    bindKeyDel()
}
```

##### dnd
在左侧组件部分选用<code>dnd</code>，而不是选用<code>step</code>  
<code>dnd</code>和<code>step</code>的优劣势对比：
+ <code>step</code>是内置封装好的组件，容易上手。但是不好拓展
+ <code>dnd</code>可定制化开发，更容易扩展。可根据选用的预研进行<code>vue</code>或<code>react</code>的开发

**重难点**
+ 在创建节点上，运用vue开发，因此必需使用<code>Graph.registerVueComponent</code>。如果只是采用<code>createNode</code>去进行创建，会在编辑情况下出现节点空白的BUG
+ 由于需要与右侧的节点数据展示关联以及vue3组件的data存入在<code>dnd.vue</code>中.因此每当点击节点的时候需要判断并处理在父页面创建的<code>cmptCellInfo</code>对象。
```
//判断在对象中是否有此节点，没有则引入

this.currentId = this.$el.closest('[data-shape=vue-shape]').getAttribute('data-cell-id')
if (!this.cmptCellInfo[this.currentId]) {
    this.cmptCellInfo[this.currentId] = cloneDeep(defaultConfig(item))
    cellNodeInfo.value && (this.cmptCellInfo[this.currentId] = cloneDeep(cellNodeInfo.value))
    !cellNodeInfo.value && (this.cmptCellInfo[this.currentId] = cloneDeep(defaultConfig(item)))
}
```
```
// 创建vue3组件
const registerV3Comp = () => {
    for (const item in nodeInfo) {
        console.log('into')
        Graph.registerVueComponent(`${item}`, {
            template: `<${item} :info="cmptCellInfo[currentId]"></${item}>`,
            components: {
                [item]: nodeCmpt[item]
            },
            data () {
                return {
                    cmptCellInfo: cmptCellInfo,
                    currentId: ''
                }
            },
            mounted () {
                this.currentId = this.$el.closest('[data-shape=vue-shape]').getAttribute('data-cell-id')
                if (!this.cmptCellInfo[this.currentId]) {
                    this.cmptCellInfo[this.currentId] = cloneDeep(defaultConfig(item))
                    cellNodeInfo.value && (this.cmptCellInfo[this.currentId] = cloneDeep(cellNodeInfo.value))
                    !cellNodeInfo.value && (this.cmptCellInfo[this.currentId] = cloneDeep(defaultConfig(item)))
                }
            }
        }, true)
    }
}
```
+ 由于直接给节点的data赋值覆盖会导致视图没有渲染，所以<code>cmptCellInfo</code>中的所有属性都指向节点中的data对象


1. 创建dnd并与画布绑定，将公共的配置化代码放入<code>config.js</code>中
```
const init = () => {
    createDnd()
    registerV3Comp()
}

const createDnd = () => {
editor.dnd = new Addon.Dnd({
    target: editor.graph,
    getDropNode: (node) => {
        const cloneNode = node.clone({ keepId: true })
        cloneNode.data = cmptCellInfo[cloneNode.id]
        // 清空选区
        nextTick(() => {
            editor.graph.resetSelection(cloneNode)
            cellClickCallback(cloneNode)
        })
        return cloneNode
    }
})
}
```
```
 // 创建vue3组件
const registerV3Comp = () => {
    for (const item in nodeInfo) {
        console.log('into')
        Graph.registerVueComponent(`${item}`, {
            template: `<${item} :info="cmptCellInfo[currentId]"></${item}>`,
            components: {
                [item]: nodeCmpt[item]
            },
            data () {
                return {
                    cmptCellInfo: cmptCellInfo,
                    currentId: ''
                }
            },
            mounted () {
                this.currentId = this.$el.closest('[data-shape=vue-shape]').getAttribute('data-cell-id')
                if (!this.cmptCellInfo[this.currentId]) {
                    this.cmptCellInfo[this.currentId] = cloneDeep(defaultConfig(item))
                    cellNodeInfo.value && (this.cmptCellInfo[this.currentId] = cloneDeep(cellNodeInfo.value))
                    !cellNodeInfo.value && (this.cmptCellInfo[this.currentId] = cloneDeep(defaultConfig(item)))
                }
            }
        }, true)
    }
}
```
2. 将组件绑定拖拽事件
```
const addCmptGraphNode = (type, info, e) => {
    if (onJudgeOnlyOneNode(type)) return
    cellNodeInfo.value = info
    const template = {
        ...nodeTypeConfig(type),
        ...{
            components: {
                [type]: nodeCmpt[type]
            }
        }
    }
    type === 'cmptProcessNode' && (template.width = 200)
    const node = editor.graph.createNode(template)
}
           
```
3. 在编辑时候需要逐个将下发的节点信息与<code>cmptCellInfo</code>绑定，不可直接用<code>tojson</code>进行绑定。此外，节点的信息必需优先于线。
```
// 清空数据 画布
editor.graph.clearCells()
for (const key in cmptCellInfo) {
    delete cmptCellInfo[key]
}
cells.forEach(cell => {
    if (cell.shape !== 'edge') {
        cmptCellInfo[cell.id] = cloneDeep(cell.data)
        editor.graph.addNode(cell)
    }
})
cells.forEach(cell => {
    if (cell.shape === 'edge') {
        editor.graph.addEdge(cell)
    }
})
```

##### nodeDesc
将数据绑定即可

**至此左中右的数据完成数据绑定。再根据业务需要自行处理**
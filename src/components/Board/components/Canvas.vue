/**
* Created by OXOYO on 2018/4/8.
*
* 画布组件
*/

<style  lang="less" rel="stylesheet/less">
  .xpe_canvas {
    display: inline-block;
    position: static;
    z-index: 2000;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;

    .canvas-box {
      display: inline-block;
      position: static;
      left: 50%;
      margin: 100px 0;
      background: #fff;

      &.active,
      &:hover {
        box-shadow: 0 0 2px 0 rgba(0, 0, 0, 0.1);
      }

      &.pc {
        width: 960px;
        height: 1000px;
        margin-left: -480px;
      }
      &.mobile {
        width: 360px;
        height: 1349px;
        margin-left: -180px;
      }

      .component-node {}

      .selection {
        display: none;
        position: static;
        border: 1px solid transparent;
        border-image: url("../../../assets/selection.gif");
        border-image-slice: 1;
        border-image-width: 1px;
        border-image-repeat: round;
      }
    }
  }
</style>

<template>
  <div
    class="xpe_canvas"
  >
  <!-- @contextmenu.stop.prevent="handleRightClickOnCanvas($event)" -->
    <div
      v-for="item in canvasMap"
      v-show="currentProject === item.id"
      :class="[ 'canvas-box', currentProject === item.id ? 'active' : '', item.type === 'pc' ? 'pc' : item.type === 'mobile' ? 'mobile' : '' ]"
      :style="{ 'width': item.width, 'height': item.height, transform: 'scale(' + item.scale.current + ',' + item.scale.current + ')' }"
      :key="item.id"
      :project-id="item.id"
      :name="item.name"
      :title="item.name"
      
      @drop.stop.prevent="handleDropOnCanvas(item, $event)"
      @dragover.stop.prevent
    >
      <!-- 选中虚线框效果 -->
      <!-- <div
        v-for="(selectionStyle, key) in item.selectionStyleMap"
        :key="'selection-' + key"
        :selection-id="'selection-' + key"
        class="selection"
        :style="selectionStyle"
      >
      </div> -->

      <template
        v-for="(node, index) in item.components"
      >
      <!-- @contextmenu.native.stop.prevent="handleRightClickOnNode(node, $event)" -->
        <component
          class="component-node"
          :is="node.component.name"
          :key="index"
          :node-id="node.id"
          v-bind="node.props"
          :style="node.style"
          
          @click.native.stop.prevent="handleComponentTrigger(node)"
          draggable="true"
          @dragstart.native.stop="handleDragStart(node, $event)"
          @dragend.native.stop="handleDragEnd(node, $event)"
          @mouseover.native.stop.prevent="handleMouseOverOnNode(node)"
          @mouseout.native.stop.prevent="handleMouseOutOnNode(node)"
          @drop.native.stop.prevent="handleDropOnNode(node, $event)"
          @dragover.stop.prevent
          @dragenter.native.stop.prevent="handleDropEnterNode(node, $event)"
          @dragleave.native.stop.prevent="handleDropLeaveNode(node, $event)"
        >
          {{ node.innerHTML}}
          <template
            v-if="node.children && node.children.length"
            v-for="(childNode, childIndex) in node.children"
          >
            <component
              class="component-node"
              :is="childNode.component.name"
              :key="childIndex"
              :node-id="childNode.id"
              v-bind="childNode.props"
              :style="childNode.style"
              @contextmenu.native.stop.prevent="handleRightClickOnNode(childNode, $event)"
              @click.native.stop.prevent="handleComponentTrigger(childNode)"
              draggable="true"
              @dragstart.native.stop="handleDragStart(childNode, $event)"
              @dragend.native.stop="handleDragEnd(childNode, $event)"
              @mouseover.native.stop.prevent="handleMouseOverOnNode(childNode)"
              @mouseout.native.stop.prevent="handleMouseOutOnNode(childNode)"
              @drop.stop.prevent="handleDropOnNode(childNode, $event)"
              @dragover.stop.prevent
              @dragenter.native.stop.prevent="handleDropEnterNode(childNode, $event)"
              @dragleave.native.stop.prevent="handleDropLeaveNode(childNode, $event)"
            >
            </component>
          </template>
        </component>
      </template>
    </div>
    <NestNoticeModal></NestNoticeModal>
  </div>
</template>

<script>
import XPEButton from '@/ui/Button.vue'
import utils from '@/global/utils'
import NestNoticeModal from './NestNoticeModal.vue'

export default {
  name: 'XPECanvas',
  components: {
    XPEButton,
    NestNoticeModal
  },
  data () {
    return {
      // 是否展开
      isExpand: true,
      // 画布列表
      canvasMap: {},
      // 当前激活对象
      currentProject: '',
      // 当前操作元素
      currentNode: '',
      // 缩放级别，默认1
      scale: {
        default: 1,
        current: 1,
        max: 10,
        min: 0.1,
        step: 0.01
      }
    }
  },
  methods: {
    // 画布右键点击
    handleRightClickOnCanvas: function (event) {
      let _t = this
      let xpeEl = document.querySelector('#xpe')
      let xVal
      let yVal
      if (xpeEl) {
        xVal = event.clientX - xpeEl.offsetLeft
        yVal = event.clientY - xpeEl.offsetTop
      } else {
        xVal = event.offsetX
        yVal = event.offsetY
      }
      // 菜单信息
      let contextMenuInfo = {
        isShow: true,
        x: xVal,
        y: yVal,
        target: 'XPE_canvas',
        list: [
          {
            name: 'expand',
            icon: {
              type: 'icon-expand',
              style: '',
              category: 'iconfont'
            },
            text: '展开',
            enable: _t.isExpand,
            action: {
              type: 'bus',
              handler: 'XPE/expand/toggle/all',
              params: false
            }
          },
          {
            name: 'fold',
            icon: {
              type: 'icon-fold',
              style: '',
              category: 'iconfont'
            },
            text: '折叠',
            enable: !_t.isExpand,
            action: {
              type: 'bus',
              handler: 'XPE/expand/toggle/all',
              params: true
            }
          },
          {
            name: 'showGuides',
            icon: {
              type: '',
              style: '',
              category: 'iconfont'
            },
            text: '显示 / 隐藏参考线',
            enable: true,
            action: {
              type: 'bus',
              handler: 'XPE/scale/guides/toggle'
            }
          },
          {
            name: 'showToolTip',
            icon: {
              type: '',
              style: '',
              category: 'iconfont'
            },
            text: '显示 / 隐藏参考线坐标',
            enable: true,
            action: {
              type: 'bus',
              handler: 'XPE/scale/guides/toolTip/toggle'
            }
          },
          {
            name: 'clearGuides',
            icon: {
              type: '',
              style: '',
              category: 'iconfont'
            },
            text: '清空当前项目参考线',
            enable: true,
            action: {
              type: 'bus',
              handler: 'XPE/scale/guides/clear',
              params: _t.currentProject
            }
          },
          {
            name: 'clear',
            icon: {
              type: '',
              style: '',
              category: 'iconfont'
            },
            text: '清空当前项目画布',
            enable: true,
            action: {
              type: 'bus',
              handler: 'XPE/canvas/clear',
              params: _t.currentProject
            }
          }
        ]
      }
      // 广播事件
      utils.bus.$emit('XPE/contextMenu/show', contextMenuInfo)
    },
    // 元素上右键
    handleRightClickOnNode: function (nodeInfo, event) {
      let _t = this
      let xpeEl = document.querySelector('#xpe')
      let xVal
      let yVal
      if (xpeEl) {
        xVal = event.clientX - xpeEl.offsetLeft
        yVal = event.clientY - xpeEl.offsetTop
      } else {
        xVal = event.offsetX
        yVal = event.offsetY
      }
      // 菜单信息
      let contextMenuInfo = {
        isShow: true,
        x: xVal,
        y: yVal,
        target: 'XPE_canvas_node',
        list: [
          {
            name: 'expand',
            icon: {
              type: 'icon-expand',
              style: '',
              category: 'iconfont'
            },
            text: '展开',
            enable: _t.isExpand,
            action: {
              type: 'bus',
              handler: 'XPE/expand/toggle/all',
              params: false
            }
          },
          {
            name: 'fold',
            icon: {
              type: 'icon-fold',
              style: '',
              category: 'iconfont'
            },
            text: '折叠',
            enable: !_t.isExpand,
            action: {
              type: 'bus',
              handler: 'XPE/expand/toggle/all',
              params: true
            }
          },
          {
            name: 'showGuides',
            icon: {
              type: '',
              style: '',
              category: 'iconfont'
            },
            text: '显示 / 隐藏参考线',
            enable: true,
            action: {
              type: 'bus',
              handler: 'XPE/scale/guides/toggle'
            }
          },
          {
            name: 'showToolTip',
            icon: {
              type: '',
              style: '',
              category: 'iconfont'
            },
            text: '显示 / 隐藏参考线坐标',
            enable: true,
            action: {
              type: 'bus',
              handler: 'XPE/scale/guides/toolTip/toggle'
            }
          },
          {
            name: 'clearGuides',
            icon: {
              type: '',
              style: '',
              category: 'iconfont'
            },
            text: '清空当前项目参考线',
            enable: true,
            action: {
              type: 'bus',
              handler: 'XPE/scale/guides/clear',
              params: _t.currentProject
            }
          },
          {
            name: 'clear',
            icon: {
              type: '',
              style: '',
              category: 'iconfont'
            },
            text: '清空当前项目画布',
            enable: true,
            action: {
              type: 'bus',
              handler: 'XPE/canvas/clear',
              params: _t.currentProject
            }
          },
          {
            name: 'edit',
            icon: {
              type: 'edit',
              style: '',
              category: 'iview'
            },
            text: '编辑',
            enable: true,
            action: {
              type: 'callback',
              handler: _t.handleComponentTrigger,
              params: nodeInfo
            }
          },
          {
            name: 'remove',
            icon: {
              type: 'android-remove',
              style: '',
              category: 'iview'
            },
            text: '删除',
            enable: true,
            action: {
              type: 'callback',
              handler: _t.handleComponentRemove,
              params: nodeInfo
            }
          }
        ]
      }
      // 广播事件
      utils.bus.$emit('XPE/contextMenu/show', contextMenuInfo)
    },
    // 处理组件点击
    handleComponentTrigger: function (nodeInfo) {
      let _t = this
      // 更新当前操作的节点
      _t.currentNode = nodeInfo.id
      // console.log('handleComponentTrigger', nodeInfo.component.name)
      // 广播事件，更新当前激活组件
      utils.bus.$emit('XPE/project/component/trigger', nodeInfo)
      _t.$nextTick(function () {
        _t.handleMouseOverOnNode(nodeInfo)
        _t.handleMouseOutOnNode()
      })
    },
    // 删除元素
    handleComponentRemove: function (nodeInfo) {
      let _t = this
      // 更新当前操作的节点
      _t.currentNode = ''
      let _canvas = _t.canvasMap[_t.currentProject];
      _canvas['components'] = _canvas['components']
          .filter(
            node => {
              if (node.id !== nodeInfo.id) {
                return true
              }
              return false
            }
          );
      console.log('canvasMap', _canvas)
      // 广播事件，更新当前激活组件
      utils.bus.$emit('XPE/project/component/remove', nodeInfo)
      _t.$nextTick(
          function () {
            _t.handleMouseOutOnNode()
          }
        );
    },
    // 元素drop 元素正在拖动时触发
    handleDropOnCanvas: function (item, event) {
      let _t = this;
      let canvasMap = _t.canvasMap;
      console.log('handleDropOnCanvas 放画布上',canvasMap);
      // 获取节点数据
      let nodeInfo = JSON.parse(event.dataTransfer.getData('node'));
      // if (nodeInfo.component.name=='FormItem') {
      //   return;
      // }
      nodeInfo = {
        id: '',
        components: {},
        props: {},
        slots: {},
        innerHTML: '',
        ...nodeInfo
      };
      let offsetX = event.offsetX;
      let offsetY = event.offsetY;
      let style = {
        // position: 'absolute',
        // 'pointer-events': 'auto',
        // left: offsetX + 'px',
        // top: offsetY + 'px'
      };
      // console.log('style', style)
      nodeInfo.style = {
        ...nodeInfo.style,
        ...style
      }
      let _components=canvasMap[item.id]['components'];
      // 更新当前画布下的组件数据
      if (_components.length) {

        _components=_components.filter(
            node => {
              if(node.children){
                node.children=node.children.filter(node2=>node2.id !== nodeInfo.id)
              }
              return node.id !== nodeInfo.id;
            }
          );

        // // 判断该节点是否已存在
        // let nodeIndex = _components.findIndex(node => node.id === nodeInfo.id);
        // if (nodeIndex > -1) {
        //   _components[nodeIndex] = nodeInfo;
        // } else {
        //   _components.push(nodeInfo);
        // }
      } 
      _components.push(nodeInfo);

      canvasMap[item.id]['components']=_components;

      _t.canvasMap = {
        ...canvasMap
      };
      // 更新当前操作的节点
      _t.currentNode = nodeInfo.id;
      // 广播事件，更新当前激活组件
      utils.bus.$emit('XPE/project/component/trigger', nodeInfo)
      _t.$nextTick(
        function () {
          _t.handleMouseOverOnNode(nodeInfo);
          _t.handleMouseOutOnNode();
        }
      );
    },
    handleDropEnterNode: function (parentNode, event) {
      let _t = this;
      console.log('handleDropEnterNode',`${parentNode.component.name}[${parentNode.id}]`, event);
    },
    handleDropLeaveNode: function (parentNode, event) {
      let _t = this;
      console.log('handleDropLeaveNode',`${parentNode.component.name}[${parentNode.id}]`, event);
    },
    handleDropOnNode: function (parentNode, event) {
      let _t = this;
      let canvasMap = _t.canvasMap;
      console.log('handleDropOnNode 放节点上',canvasMap);
      // 获取节点数据
      let nodeInfo = JSON.parse(event.dataTransfer.getData('node'))
      nodeInfo = {
        id: '',
        components: {},
        props: {},
        slots: {},
        innerHTML: '',
        ...nodeInfo
      };
      // 是否可以嵌套标识
      let isNest = false;
      let parentNest = parentNode.component.nest || {
        enable: false,
        parent: {
          allow: [],
          deny: []
        },
        children: {
          allow: [],
          deny: []
        }
      };
      let childNest = nodeInfo.component.nest || {
        enable: false,
        parent: {
          allow: [],
          deny: []
        },
        children: {
          allow: [],
          deny: []
        }
      };
      let parentName = parentNode.component.name;
      let childName = nodeInfo.component.name;
      // 判断目标节点是否支持嵌套
      if (parentNest.enable) {
        if (!parentNest.children.allow.length && !parentNest.children.deny.length && !childNest.parent.allow.length && !childNest.parent.deny.length) {
          isNest = true
        } else if (parentNest.children.allow.length) {
          isNest = parentNest.children.allow.includes(childName) && !parentNest.children.deny.includes(childName)
        } else if (parentNest.children.deny.length) {
          isNest = !parentNest.children.deny.includes(childName)
        } else if (childNest.parent.allow.length) {
          isNest = childNest.parent.allow.includes(parentName) && !childNest.parent.deny.includes(parentName)
        } else if (childNest.parent.deny.length) {
          isNest = !childNest.parent.deny.includes(parentName)
        }
      }
      console.log(parentNest);
      if (!isNest) {
        // TODO 弹窗提示不能嵌套
        utils.bus.$emit('XPE/canvas/nest/notice');
        return;
      }
      let offsetX = event.offsetX;
      let offsetY = event.offsetY;
      let style = {
        position: 'static',
        // 子元素可以声明pointer-events来解禁父元素的阻止鼠标事件限制。
        // 如果你对一个元素设置了click事件监听器，然后你移除了pointer-events样式声明，
        // 或把它的值改变为auto，监听器会重新生效。基本上，监听器会遵守pointer-events的设定。
        // 'pointer-events': 'auto',
        // left: offsetX + 'px',
        // top: offsetY + 'px'
      };
      // console.log('style', style)
      nodeInfo.style = {
        ...nodeInfo.style,
        ...style
      };
      let _canvas=canvasMap[_t.currentProject];
      // 1.删除当前节点
      _canvas['components'] = 
          _canvas['components']
          .filter(
            node => {
              if(node.children){
                node.children=node.children.filter(node2=>node2.id !== nodeInfo.id)
              }
              
              return node.id !== nodeInfo.id;
            }
          );
      // 2.给当前节点设置children
      _canvas['components']
          .map(
            node => {
              if (node.id === parentNode.id) {
                if (!node.hasOwnProperty('children')) {
                  node.children = []
                }
                node.children.push(nodeInfo)
              }
              return node
            }
          );
      _t.canvasMap = {
        ...canvasMap
      };
      _t.$nextTick(
        function () {
          _t.handleComponentTrigger(nodeInfo)
        }
      );
    },
    // 节点拖拽
    handleDragStart: function (nodeInfo, event) {
      console.log('handleDragStart',`${nodeInfo.component.name}[${nodeInfo.id}]`);
      // 拖拽的节点数据
      event.dataTransfer.setData('node', JSON.stringify(nodeInfo));
    },
    handleDragEnd: function (nodeInfo, event) {
      console.log('handleDragEnd',`${nodeInfo.component.name}[${nodeInfo.id}]` );
      // 拖拽的节点数据
    },
    // 处理options更新
    handleOptionsSet: function (nodeInfo) {
      let _t = this
      let canvasMap = _t.canvasMap
      canvasMap[_t.currentProject]['components'].map(node => {
        if (node.id === nodeInfo.id) {
          // console.log('node.id', node.id)
          node.props = nodeInfo.props
          node.slots = nodeInfo.slots
          node.innerHTML = nodeInfo.innerHTML
          node.style = nodeInfo.style
        }
        return node
      })
      _t.canvasMap = {
        ...canvasMap
      }
      // console.log('handleOptionsSet')
      _t.$nextTick(function () {
        _t.handleMouseOverOnNode(nodeInfo)
      })
    },
    // 清空画布
    clearCanvas: function (projectID) {
      let _t = this
      _t.canvasMap[projectID]['components'] = []
      _t.canvasMap[projectID]['selectionStyleMap'] = {}
      _t.canvasMap[projectID]['scale'] = {
        ..._t.scale
      }
    },
    handleMouseOverOnNode: function (nodeInfo) {
      let _t = this
      console.log('handleMouseOverOnNode',`${nodeInfo.component.name}[${nodeInfo.id}]`);
      let target = document.querySelector('[node-id=' + nodeInfo.id + ']')
      if (target) {
        let width = target.offsetWidth + 2
        let height = target.offsetHeight + 2
        // console.log('width', width, 'height', height)
        let style = {
          display: 'inline-block',
          width: width + 'px',
          height: height + 'px',
          margin: '-1px',
          left: nodeInfo.style.left,
          top: nodeInfo.style.top
        }
        _t.canvasMap[_t.currentProject]['selectionStyleMap'] = {
          ..._t.canvasMap[_t.currentProject]['selectionStyleMap'],
          [nodeInfo.id]: style
        }
      }
    },
    handleMouseOutOnNode: function () {
      let _t = this
      // console.log('handleMouseOutOnNode')
      if (Object.keys(_t.canvasMap[_t.currentProject]['selectionStyleMap']).length) {
        let selectionStyleMap = _t.canvasMap[_t.currentProject]['selectionStyleMap']
        Object.keys(selectionStyleMap).map(key => {
          if (key !== _t.currentNode) {
            delete selectionStyleMap[key]
          }
        })
        _t.canvasMap[_t.currentProject]['selectionStyleMap'] = {
          ...selectionStyleMap
        }
      }
    },
    handleZoom: function (params) {
      let _t = this
      let current
      let projectInfo = _t.canvasMap[_t.currentProject]
      if (!projectInfo) {
        return
      }
      switch (params.type) {
        case 'zoom-in':
          current = projectInfo.scale.current + projectInfo.scale.step
          break
        case 'zoom-out':
          current = projectInfo.scale.current - projectInfo.scale.step
          break
        case 'zoom-reset':
          current = projectInfo.scale.default
          break
      }
      if (current > projectInfo.scale.max) {
        current = projectInfo.scale.max
      } else if (current < projectInfo.scale.min) {
        current = projectInfo.scale.min
      } else if (!current) {
        current = projectInfo.scale.default
      }
      _t.canvasMap[_t.currentProject]['scale']['current'] = current
      // 执行回调
      params.callback(Math.ceil(current * 100) + '%')
    }
  },
  created: function () {
    let _t = this
    // 监听事件
    utils.bus.$on('XPE/expand/toggle/all', function (val) {
      _t.isExpand = val
    })
    // 监听事件
    utils.bus.$on('XPE/project/add/ok', function (projectInfo) {
      // console.log('canvasMap', projectInfo.name)
      _t.canvasMap = {
        ..._t.canvasMap,
        [projectInfo.id]: {
          id: '',
          name: '',
          type: 'pc',
          width: 0,
          height: 0,
          components: [],
          selectionStyleMap: {},
          scale: {
            ..._t.scale
          },
          ...projectInfo
        }
      }
      // 更新当前激活项目
      _t.currentProject = projectInfo.id
    })
    utils.bus.$on('XPE/project/trigger', function (projectID) {
      // 更新当前激活项目
      _t.currentProject = projectID
    })
    utils.bus.$on('XPE/project/remove', function (projectID) {
      // 删除当前操作对象数据
      delete _t.canvasMap[projectID]
    })
    utils.bus.$on('XPE/project/component/options/set', function (nodeInfo) {
      _t.handleOptionsSet(nodeInfo)
    })
    utils.bus.$on('XPE/canvas/clear', function (projectID) {
      _t.clearCanvas(projectID)
    })
    utils.bus.$on('XPE/canvas/zoom', function (params) {
      _t.handleZoom(params)
    })
  }
}
</script>

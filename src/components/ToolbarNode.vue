<script setup lang="ts">
import { MarkerType, Position, useVueFlow, Handle } from '@vue-flow/core'
import { NodeToolbar } from '@vue-flow/node-toolbar'
import { Plus, Link, Minus } from '@element-plus/icons-vue'
import { getNewEdge, getNewNode, getNewCondition } from '@/components/inital-elements'

interface Props {
  id: string
  data: any
  position: any
  label?: any
}

const props = defineProps<Props>()

const { nodes, addNodes, addEdges, getNodes, removeNodes, getEdges, removeEdges } = useVueFlow()

const addNode = () => {
  const id = nodes.value.length + 1
  const parentId = +props.id

  const newNode = getNewNode(id, parentId, props.position)
  const newEdge = getNewEdge(id, parentId)

  addNodes([newNode])
  addEdges([newEdge])
}

const addCondition = () => {
  const id = nodes.value.length + 1
  const parentId = +props.id

  const newNode = getNewCondition(id, parentId, props.position)
  const newEdge = getNewEdge(id, parentId)

  addNodes([newNode])
  addEdges([newEdge])

  addNodesWhenAddCondition(newNode)
}

const addNodesWhenAddCondition = (parentNode: any) => {
  const id = nodes.value.length

  for (let index = 1; index <= 2; index++) {
    const newNode = {
      id: `${id + index}`,
      type: 'toolbar',
      label: `Node ${id + index}`,
      position: {
        x: index === 1 ? parentNode.position.x + 10 : parentNode.position.x + 200,
        y: index === 1 ? parentNode.position.y + 200 : parentNode.position.y + 38.2,
      },
      data: { toolbarPosition: Position.Bottom },
      style: {
        border: '1px solid #fff',
        background: '#740fad',
        color: 'white',
        borderRadius: '99px',
      },
      parentNode: '' + id,
    }

    const newEdge = {
      id: `e${id}-${id + index}`,
      source: `${id}`,
      target: `${id + index}`,
      type: 'smoothstep',
      label: index === 1 ? 'false' : 'true',
      labelShowBg: true,
      labelBgStyle: { fill: '#fcf6ff' },
      markerEnd: MarkerType.Arrow,
      style: { stroke: '#10b981' },
    }

    addNodes([newNode])
    addEdges([newEdge])
  }
}

const removeNode = (id: any) => {
  const nodes = getNodes.value;
  const edges = getEdges.value;
  let deleteNodes = [];
  let deleteEdges = [];

  const addNodesToList = (id: any) => {
    for(var node of nodes) {
      if(node.parentNode === id) {
        deleteNodes.push(node.id);
        addNodesToList(node.id);
      }
    }
  }

  addNodesToList(id);

  for(var edge of edges) {
    for(var node of deleteNodes) {
      if(edge.source === node.id || edge.target === node.id) {
        deleteEdges.push(edge);
      }
    }
  }

  removeEdges(deleteEdges);
  removeNodes(deleteNodes, false);
}
</script>

<template>
  <node-toolbar
    style="display: flex; gap: 0.5rem; align-items: center"
    :is-visible="data.toolbarVisible"
    :position="data.toolbarPosition"
  >
    <el-button type="primary" :icon="Plus" circle @click="addNode" />
    <el-button type="primary" :icon="Link" circle @click="addCondition" />
    <el-button type="danger" :icon="Minus" circle @click="removeNode(id)" />
  </node-toolbar>

  <div :style="{ padding: '10px 20px' }">
    {{ label }}
  </div>

  <Handle type="target" :position="Position.Left" class="tw-opacity-0" />
  <Handle type="source" :position="Position.Right" class="tw-opacity-0" />
</template>

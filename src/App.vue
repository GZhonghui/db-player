<script setup>
import { ref } from 'vue'
import { NTree, NButton, NTabs, NTab, NCard, NTabPane } from 'naive-ui'

const libraryData = {
  a: {
    a1: {
      _f: ['x.flac']
    },
    _f: []
  },
  b: {
    _f: []
  },
  _f: ['1.mp3', '2.mp3']
}

// 转换为 n-tree 格式 + 加上 fullPath
function convertToTree(data, parentKey = '', pathParts = []) {
  const result = []

  for (const key in data) {
    if (key === '_f') {
      const files = data[key].map((file, index) => ({
        label: file,
        key: `${parentKey}/f-${index}`,
        isLeaf: true,
        fileName: file,
        fullPath: [...pathParts, file].join('/')
      }))
      result.push(...files)
    } else {
      const node = {
        label: key,
        key: `${parentKey}/${key}`,
        children: convertToTree(data[key], `${parentKey}/${key}`, [...pathParts, key])
      }
      result.push(node)
    }
  }

  return result
}

const treeData = ref(convertToTree(libraryData))
const checkedKeys = ref([])

function flattenTree(tree) {
  const result = []
  function recurse(nodes) {
    for (const node of nodes) {
      result.push(node)
      if (node.children) {
        recurse(node.children)
      }
    }
  }
  recurse(tree)
  return result
}

function showSelectedFiles() {
  const allNodes = flattenTree(treeData.value)
  const selectedFiles = allNodes
    .filter(node => node.isLeaf && checkedKeys.value.includes(node.key))
    .map(node => node.fullPath)

  if (selectedFiles.length === 0) {
    alert('你没有选择任何文件')
  } else {
    alert(`你选中的文件路径为：\n\n${selectedFiles.join('\n')}`)
  }
}
</script>

<template>
  <div id="library">
    <n-tabs type="line" animated>
      <n-tab-pane name="tab-library" tab="音乐">
        <n-tree
          :data="treeData"
          :default-expand-all="false"
          block-line
          show-line
          checkable
          cascade
          v-model:checked-keys="checkedKeys"
        />
      </n-tab-pane>
      <n-tab-pane name="tab-playlist" tab="列表">
        Hey Jude
      </n-tab-pane>
    </n-tabs>
    <div style="margin-top: 16px;">
      <n-button type="primary" @click="showSelectedFiles">
        显示选中的完整路径
      </n-button>
    </div>
  </div>
</template>

<style scoped>
#library {
  margin: 16px;
  padding: 16px;
  border: 1px solid #ccc;
  border-radius: 8px;
}
</style>

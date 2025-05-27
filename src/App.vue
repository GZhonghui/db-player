<script setup>
import { ref } from 'vue'
import { NTree, NButton, NTabs, NTab, NCard, NTabPane, NList, NListItem } from 'naive-ui'

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

const songs = ref([
  { artist: '万能青年旅店', title: '杀死那个石家庄人' },
  { artist: '朴树', title: '平凡之路' },
  { artist: '李志', title: '梵高先生' }
])

const play = (song) => {
  console.log('播放歌曲:', song)
}
</script>

<template>
  <div id="player">
    <p>播放状态</p>
  </div>
  <div id="library">
    <n-tabs type="line" animated>
      <n-tab-pane name="tab-library" tab="音乐">
        <n-button type="primary" style="margin-bottom: 8px;" @click="showSelectedFiles">
          添加至播放列表
        </n-button>
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
        <n-button>清空</n-button>
        <n-list hoverable clickable>
          <n-list-item v-for="(song, index) in songs" :key="index">
            <div class="song-item">
              <span class="song-text">{{ song.artist }} - {{ song.title }}</span>
              <n-button
                type="primary"
                class="play-button"
                @click="play(song)"
                quaternary
              >
                播放
              </n-button>
              <n-button type="primary" class="play-button">删除</n-button>
            </div>
          </n-list-item>
        </n-list>
      </n-tab-pane>
    </n-tabs>
  </div>
</template>

<style scoped>
#player, #library {
  margin: 16px;
  padding: 16px;
  border: 1px solid #ccc;
  border-radius: 8px;
}

.song-item {
  position: relative;
  padding: 8px 0;
}

.song-text {
  display: inline-block;
}

.play-button {
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0;
  transition: opacity 0.2s;
  z-index: 1;
}

/* 当鼠标悬浮到 .song-item 上时，显示按钮 */
.song-item:hover .play-button {
  opacity: 1;
}

.song-item:hover .song-text {
  opacity: 0.3;
}
</style>

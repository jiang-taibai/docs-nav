<script>
import {NNotificationProvider, NButton, NImage, NInput, NSelect, NTag, useNotification} from 'naive-ui'

export default {
  name: 'MainView',
  components: {
    NNotificationProvider, NButton, NImage, NInput, NSelect, NTag,
  },
  data() {
    return {
      notification: useNotification(),
      originalDocuments: [],
      selectedTags: [],
      selectedSortBy: 'publicationTime',
      sortOrder: 'desc',
      inputKeyword: '',
      allTags: [],
      allSortBy: [
        {label: '按发布时间', value: 'publicationTime'},
        {label: '按更新时间', value: 'lastModifiedTime'},
        {label: '按标签拟合', value: 'matchedTags'},
      ],
    }
  },
  methods: {
    /**
     * 获取json数据
     *
     * @returns {Promise}
     */
    fetchJsonData() {
      return new Promise((resolve, reject) => {
        fetch('data/documents.json?timestamp=' + new Date().getTime())
            .then(response => {
              resolve(response.json())
            })
            .catch(error => {
              reject(error)
            });
      })
    },

    /**
     * 更改排序规则
     */
    changeSortOrder() {
      if (this.sortOrder === 'asc') {
        this.sortOrder = 'desc'
      } else {
        this.sortOrder = 'asc'
      }
    },

    /**
     * 处理动画结束
     *
     * @param document 文档
     */
    handleAnimationEnd(document) {
      document.entered = true;
    },

    /**
     * 打开链接
     * @param link 链接
     */
    openLink(link) {
      window.open(link)
    }
  },
  computed: {
    DocumentViewport() {
      const filteredDocuments = []
      for (const document of this.originalDocuments) {
        if (this.inputKeyword !== '' && (
            !document.title.includes(this.inputKeyword) &&
            !document.description.includes(this.inputKeyword))
        ) {
          continue
        }
        if (this.selectedTags.length === 0) {
          filteredDocuments.push(document)
        } else {
          let matchedTags = 0
          for (const tag of this.selectedTags) {
            if (document.tags.includes(tag)) {
              matchedTags += 1
            }
          }
          if (matchedTags > 0) {
            filteredDocuments.push({
              ...document,
              matchedTags,
            })
          }
        }
      }
      return filteredDocuments.sort((a, b) => {
        if (this.selectedSortBy === null) {
          return 0
        }
        if (this.sortOrder === 'asc') {
          return a[this.selectedSortBy] > b[this.selectedSortBy] ? 1 : -1
        } else {
          return a[this.selectedSortBy] < b[this.selectedSortBy] ? 1 : -1
        }
      })
    }
  },
  mounted() {
    this.fetchJsonData().then((data) => {
      this.originalDocuments = data
      this.allTags = data.reduce((acc, cur) => {
        return [...acc, ...cur.tags]
      }, []).filter((tag, index, self) => {
        return self.indexOf(tag) === index
      }).map(tag => {
        return {
          label: tag,
          value: tag,
        }
      })
      this.notification.success({
        content: '共获取到' + data.length + '条数据！',
        duration: 3000,
      })
    }).catch((error) => {
      console.error(error)
      this.notification.error({
        content: '获取数据失败！',
        duration: 5000,
      })
    })
  },
}
</script>

<template>
  <div id="container">

    <div id="container-title">Where is the Public Document 🔎📚🚀🚀</div>

    <div id="toolbar">
      <span>Tags:</span>
      <n-select style="flex: 1" v-model:value="selectedTags"
                placeholder="请选择标签" :consistent-menu-width="false"
                multiple :options="allTags" clearable/>
      <span>Sort:</span>
      <n-select style="width: 140px" v-model:value="selectedSortBy"
                placeholder="请选择排序规则"
                :options="allSortBy"/>
      <n-button secondary strong @click="changeSortOrder">
        <template #icon>
          {{ sortOrder === 'asc' ? '🔼' : '🔽' }}
        </template>
        {{ sortOrder === 'asc' ? '升序' : '降序' }}
      </n-button>
      <n-input style="width: 250px" v-model:value="inputKeyword" placeholder="Keywords"/>
    </div>

    <div id="document-list">
      <TransitionGroup name="list" tag="div">
        <div v-for="document in DocumentViewport" :key="document.id">
          <div class="document-item">
            <div class="document-card">
              <div class="document-card-cover">
                <n-image lazy width="200" height="200"
                         object-fit="cover"
                         :src="document.cover"/>
              </div>
              <div class="document-card-content">
                <div class="document-card-title" :title="document.title">
                  {{ document.title }}
                </div>
                <div class="document-card-description multi-line-truncate" :title="document.description">
                  {{ document.description }}
                </div>
                <div class="document-card-footer">
                  <div class="document-card-author">
                    <div>Author(s):</div>
                    <n-tag :bordered="false" size="small" type="success">
                      {{ document.author }}
                    </n-tag>
                  </div>
                  <div class="document-card-creation-time">
                    <div>Publication:</div>
                    <n-tag :bordered="false" size="small">
                      {{ document.publicationTime }}
                    </n-tag>
                  </div>
                  <div class="document-card-last-modified-time">
                    <div>Updated:</div>
                    <n-tag :bordered="false" size="small">
                      {{ document.lastModifiedTime }}
                    </n-tag>
                  </div>
                  <div class="document-card-link">
                    <n-button type="info" @click="openLink(document.link)">
                      查看
                    </n-button>
                  </div>
                </div>
              </div>
            </div>
            <div class="document-card-tags">
              <div class="document-card-tags-label">Tags:</div>
              <n-tag v-for="tag in document.tags" :key="tag" size="small"
                     :bordered="false" type="info">{{ tag }}
              </n-tag>
            </div>
          </div>
        </div>
      </TransitionGroup>
    </div>

  </div>
</template>

<style scoped>
.list-move, /* 对移动中的元素应用的过渡 */
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-leave-to {
  opacity: 0;
  transform: translateX(100%);
}

.list-enter-from {
  opacity: 0;
  transform: translateY(100%);
}

/* 确保将离开的元素从布局流中删除
  以便能够正确地计算移动的动画。 */
.list-leave-active,
.list-enter-active {
  position: absolute;
}

#container {
  width: 60vw;
  margin: 0 auto;
  min-width: 600px;
  padding: 0 20px;
}

#container-title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
}

#toolbar {
  margin-top: 20px;
  display: flex;
  gap: 8px;
  align-items: center;
}

#document-list {
  margin-top: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background-color: rgba(255, 255, 255, 0.8);
  position: relative;
}

.document-item {
  border: 1px solid #ccc;
  border-radius: 5px;
  margin: 10px;
  transition: all 0.2s ease-in-out;
  background-color: #FFFFFF;
}

.document-item:hover {
  box-shadow: 0 0 10px #ccc;
}

.document-card {
  display: flex;
  flex-direction: row;
  overflow: hidden;
  height: 200px;
}

.document-card-content {
  display: flex;
  flex-direction: column;
  padding: 10px;
  width: 100%;
  border-bottom: 1px solid #ccc;
}

.document-card-title {
  font-size: 20px;
  font-weight: bold;
  text-align: left;
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.document-card-description {
  font-size: 16px;
  color: #666;
  text-align: left;
  flex: 1;
  text-indent: 2em;
  padding-right: 4px;
}

.multi-line-truncate {
  display: -webkit-box;
  -webkit-line-clamp: 4;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.document-card-footer {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  max-width: 500px;
}

.document-card-author, .document-card-creation-time, .document-card-last-modified-time, .document-card-tags-label {
  font-size: 14px;
  color: #666;
}

.document-card-tags {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  align-items: center;
  gap: 4px;
  padding: 8px;
  flex-wrap: wrap;
}
</style>

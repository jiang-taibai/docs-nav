# Docs Nav

效果展示：https://doc.coderjiang.com/whut/

## 运行

你需要安装 Node.js、npm，然后执行以下命令：

```bash
npm install
npm run dev
```

## 修改

### 数据源

数据源在 `public/data/documents.json`，允许你在打包后不需要重新构建项目就可以修改文档。

数据源是一个 JSON 数组，每个元素是一个文档对象，包含以下字段：

```json
{
  "id": "唯一性 ID",
  "entered": false,
  "title": "名称",
  "author": "作者名称，如果有多个可以用逗号分开",
  "description": "简介",
  "cover": "封面图片，相对路径或者是 URL",
  "link": "查看文档的 URL",
  "publicationTime": "2024-03-27",
  "lastModifiedTime": "2024-03-27",
  "tags": [
    "标签一",
    "标签二",
    "标签三"
  ]
}
```

### 配置

排序规则在 `src/view/MainView.vue` 中的 `data` 里的 `allSortBy` 字段，`label` 是显示的名称，`value` 是排序的字段。

```javascript
allSortBy: [
    {label: '按发布时间', value: 'publicationTime'},
    {label: '按更新时间', value: 'lastModifiedTime'},
    {label: '按标签拟合', value: 'matchedTags'},
]
```

但具体的过滤和排序规则在 `computed` 里的 `DocumentViewport` 实现，你可以根据自己的需求修改。以下代码仅供参考：

```javascript
DocumentViewport()
{
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
```

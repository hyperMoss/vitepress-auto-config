# vitepress-auto-config
一个帮助vitepress根据文件目录和内容，自动生成标题导航的思路
## 思路
通过遍历文件，构造文件树，并读取文件中的第一个#标题作为菜单名，利用文件夹名区分文章内容

针对不同的文件目录，使用不同的配置传参

详情参考config.js 文件

```js
// 获取目标目录
function addFileDir(dir) { return path.resolve(__dirname, `../docs/${dir}`) }

// 收集目录下的文件
function collectFileDir(dir, callback) {
  let fileTree = {}
  const rootDir = dir
  let arr = [dir]
  let current = null
  let index = 0
  while (current = arr[index++]) {
    // 读取当前文件，并做一个判断，文件目录分别处理
    let stat = fs.statSync(current)
    //如果文件是目录
    if (stat.isDirectory()) {
      //读取当前目录，拿到所有文件和目录
      let files = fs.readdirSync(current)
      // 将非根目录的文件添加到文件树
      if (current !== dir) {
        fileTree[path.basename(current)] = files.map(file => {
          try {
            let data = fs.readFileSync(path.resolve(current, file), 'utf8')
            let headling = /^(\#{1})([^\#\n]+.)$/m
            let title = headling.exec(data)[0].replace(/^(\#)/,'')
            return { title, fileName: path.basename(file, path.extname(file)) }
          } catch (err) {
            console.error(err)
          }
        })
      }
      arr = arr.concat(files.map(file => path.resolve(current, file)))
    }
  }
  callback && callback(fileTree)
}


function sidebarFactory(dirName) {
  let res = []
  collectFileDir(addFileDir(dirName), (tree) => {
    res = Object.keys(tree).map(key => ({
      text: key,
      children: tree[key].map(({ title, fileName }) => {
        return { text: title, link: `/docs/${dirName}/${key}/${fileName}` }
      })
    }))
  })
  return res
}
```

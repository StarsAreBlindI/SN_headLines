## 一、效果图
内容管理界面：
![](../img/1.png)
发布文章界面：

![](../img/2.png)

## 二、预览网址
项目网址：[SN头条数据管理平台](https://starsareblindi.github.io/SN_headLines)

## 三、相关技术
### 1.基于Bootstrap搭建网站标签和样式
关键代码：
```html
// html部分
<link rel="stylesheet" 
  href="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/5.2.3/css/bootstrap.min.css">
<link rel="stylesheet"
  href="https://at.alicdn.com/t/c/font_4001111_4jrdiaeyvuy.css?spm=a313x.7781069.1998910419.52&file=font_4001111_4jrdiaeyvuy.css">
<link rel="stylesheet" 
  href="https://cdn.bootcdn.net/ajax/libs/bootstrap-icons/1.10.3/font/bootstrap-icons.min.css">

// js部分
<script src="https://cdn.bootcdn.net/ajax/libs/twitter-bootstrap/5.2.3/js/bootstrap.min.js"></script>
```
### 2.集成wangEditor插件实现富文本编辑器
关键代码：
```html
<head>
  <!-- 引入wangEditor样式文件 -->
  <link rel="stylesheet" href="https://cdn.bootcdn.net/ajax/libs/wangeditor5/5.1.23/css/style.min.css">
</head>
<body>
<!-- 富文本编辑器位置 -->
<div id="editor—wrapper">
<div id="toolbar-container"><!-- 工具栏 --></div>
<div id="editor-container"><!-- 编辑器 --></div>
</body>
```
### 3.使用原生JavaStript完成增删改查等业务
回显文章和保存文章关键代码：
```javascript

; (function () {
  // 发布文章页面接收参数判断（共用同一套表单）
  const paramsStr = location.search
  const params = new URLSearchParams(paramsStr)
  params.forEach(async (value, key) => {
    // 当前有要编辑的文章 id 被传入过来
    if (key === 'id') {
      // 修改标题和按钮文字
      document.querySelector('.title span').innerHTML = '修改文章'
      document.querySelector('.send').innerHTML = '修改'
      // 获取文章详情数据并回显表单
      const res = await axios({
        url: `/v1_0/mp/articles/${value}`
      })
      console.log(res)
      // 组织我仅仅需要的数据对象，为后续遍历回显到页面上做铺垫
      const dataObj = {
        channel_id: res.data.channel_id,
        title: res.data.title,
        rounded: res.data.cover.images[0], // 封面图片地址
        content: res.data.content,
        id: res.data.id
      }
      // 遍历数据对象属性，映射到页面元素上，快速赋值
      Object.keys(dataObj).forEach(key => {
        if (key === 'rounded') {
          // 封面设置
          if (dataObj[key]) {
            // 有封面
            document.querySelector('.rounded').src = dataObj[key]
            document.querySelector('.rounded').classList.add('show')
            document.querySelector('.place').classList.add('hide')
          }
        } else if (key === 'content') {
          // 富文本内容
          editor.setHtml(dataObj[key])
        } else {
          // 用数据对象属性名，作为标签 name 属性选择器值来找到匹配的标签
          document.querySelector(`[name=${key}]`).value = dataObj[key]
        }
      })
    }
  })
})();

document.querySelector('.send').addEventListener('click', async e => {
  // 判断按钮文字，区分业务（因为共用一套表单）
  if (e.target.innerHTML !== '修改') return
  // 修改文章逻辑
  const form = document.querySelector('.art-form')
  const data = serialize(form, { hash: true, empty: true })
  // 调用编辑文章接口，保存信息到服务器
  try {
    const res = await axios({
      url: `/v1_0/mp/articles/${data.id}`,
      method: 'PUT',
      data: {
        ...data,
        cover: {
          type: document.querySelector('.rounded').src ? 1 : 0,
          images: [document.querySelector('.rounded').src]
        }
      }
    })
    console.log(res)
    myAlert(true, '修改文章成功')
  } catch (error) {
    myAlert(false, error.response.data.message)
  }
})
```
### 4.使用axios拦截器进行权限判断及前台线上接口交互
关键代码：
```javascript
// axios 公共配置
// 基地址
axios.defaults.baseURL = 'http://geek.itheima.net'

// 添加请求拦截器
axios.interceptors.request.use(function (config) {
  // 统一携带 token 令牌字符串在请求头上
  const token = localStorage.getItem('token')
  token && (config.headers.Authorization = `Bearer ${token}`)
  return config;
}, function (error) {
  // 对请求错误做些什么
  return Promise.reject(error);
});

axios.interceptors.response.use(function (response) {
  // 2xx 范围内的状态码都会触发该函数
  // 直接返回服务器的响应结果对象
  const result = response.data
  return result;
}, function (error) {
  // 超出 2xx 范围内的状态码都会触发该函数
  if (error?.response?.status === 401){
    alert('身份验证失败，请重新登录')
    localStorage.clear()
    location.href = '../login/index.html'
  }
  return Promise.reject(error);
});
```

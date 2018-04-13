---
title: 封装layui的组件
date: 2018-04-13 22:52:47
tags:
categories:
---

## byselect
---
这是针对下拉列表的封装，可以支持本地数据源和远程请求的数据源，支持枚举表和sql查询。

### 基本用法
---
```html
<select name="approvalStatus" by-filter="select" by-label="审核状态" by-enutype="approvalStatus" by-default="0"></select>
<script>
// ...
// 默认全部 by-filter="select" 的渲染
byselect.render();
</script>
```
```js
// 渲染下拉列表
byselect.render({
    elem: $('#select-id')
    , label: "审核状态" // 对应 by-label, 显示名称
    , type: "approvalStatus" // 对应by-enutype， 枚举类型
    , defaultValue: "0" // 默认值，对应by-default
    , async: true // 是否异步，默认为true， false时为同步方式
    , param: {} // 去后台请求时的参数对象
    , refresh: false // 强制刷新标识，默认为false，当为true时，每次执行的枚举都从后台获取，不走缓存
});
```
```js
// 获取对应枚举值的枚举名称，先从缓存中获取，没有则去后台获取
byselect.getEnuName(type, value);
``` 

### 注意事项
1. 请求路径的问题，是在 `config.js` 中的 `enuUrl`配置的。
2. 内部包含`form.render()`重新渲染，所以只需要将`<select>`放在`class="layui-form"`的标签内部即可，一定要！
3. 获取后台枚举时，可以支持单枚举和多枚举获取，多枚举即type使用`@`分割即可，但是注意返回值的json格式不同。
```json
// 单枚举， data是object
{
    "code":200,
    "msg":"",
    "data": {
        "enus": [{
            "name": "男",
            "value": "1"
        },{
            "name": "女",
            "value": "1"
        }],
        "defaultValue": "2"
    }
}
// 多枚举，data是array
{
    "code":200,
    "msg":"",
    "data": [{
        "enus": [{
            "name": "男",
            "value": "1"
        },{
            "name": "女",
            "value": "1"
        }],
        "defaultValue": "2"
    },{
        "enus": [{
            "name": "已读",
            "value": "1"
        },{
            "name": "未读",
            "value": "2"
        }],
        "defaultValue": "1"
    }]
}
```

## request

### request请求方法
```js
// 封装自ajax
request.get(url, param).done(function(){});
// 多种方式，同步异步
request.getSync/post/postSync/put/del/jsonp/html/htmlSync
```
### request.url(url)
封装相对路径为实际请求绝对路径，需配合`config.js`中`apiPath`使用。

### request.getPKs(rows, key)
获取行主键，之所以放到request中，是因为很多时候是在put和delete行数据时才会用到获取主键，而此时肯定是要用到request发送请求的，是为了方便。

### 注意事项
1. request已封装进token。
2. url有两种方式，`普通写法`和`指定请求方式`的写法。可以满足更多种情况，更加灵活。
```
// front!url，此时告诉request这是一个前台url，不需要组合apiPath，传入的url原样请求
request.get('front!http://ihave.news')
// 虽然调用的post方法，但是指定了put!，则实际请求为put
request.post('put!expo/list')
```

## bymenu

### 只有侧边栏
```js
// menus 请求后台拿到的menus数据
// parent 侧边栏ul节点
bymenu.showLeft(menus, parent);
```

### 顶栏和侧边栏
```js
// menus 数据
// topNode 顶栏ul节点
// leftNode 侧边栏ul节点
bymenu.showTopLeft(menus, topNode, leftNode);
```

## byprogress

### byprogress.render(opts)
```js
// 进度条layer
byprogress.render({
    title: "上传进度" // layer标题
    , id: "pro" // 用来标识lay-filter
    , url: "expo/progress" // 向后台请求进度的url
    , done: function() {
        // 进度条达到100%后的回调
    }
});
// 注：默认500ms请求间隔
```

## byform
其实也可以理解为layer和form的组合，一种配合html模板生成通用的模态窗口的组件。

### byform.render
```js
byform.render({
    title: "title" // 模态框标题
    , area: ['400px','300px'] // 模态框宽、高
    , template: 'pages/customer/cus-template.html' // 模板文件路径，支持读取html文件
    , data: {} // 用来回填form表单的数据
    , dataId: "customerId" // 用来指定PK字段
    , onOpen: function() {
        // 打开后，渲染完毕后执行的回调
    }
    , onSubmit: function() {
        // 点击确定，执行完提交后的回调
    }
    , tools: [{  // 工具栏
        title: '备忘录',
        icon: '&#xe63c;',
        type: 'prompt',
        column: 'content',
        data: {
            customerId: rows[0].customerId,
            noteType: '财务',
            title: '维护财务信息'
        },
        url: 'post!crm/note'
    }]
    , config: {} // layer原生配置项
});
```

## bytable
以上说的种种组件，其实都是为了最后这个大boss准备的。因为在一个管理系统中，最常用、最常出现的页面元素无非就是`form`,`select`,`table`,`toolbar`等，并且业务的开展都围绕了一个数据表格来进行。所以bytable是集成了各组件后的集成性组件。

### bytable.render
```js
var tableIns = bytable.render({
    elem: '#merchant',
    // table后台数据源地址
    url: 'crm/customer/general',
    where: {
        applyStatus: "2",
        customerStatus: "0"
    },
    // 查询条件
    search: [{
        label: '请输入单位名称',  // input的placeholder
        column: 'companyName',  // input name
        type: 'input',          // input类型
        disabled: true,         // 不可编辑
        width: 200              // width: 0 即为隐藏
    }, {
        label: '单位类型',
        column: 'companyCategory',
        type: 'select',             // 下拉列表
        enutype: 'company_category',// 枚举类型
        defaultValue: "2",          // 默认值
        search: true                // 启用layui可搜索下拉列表
    }, {
        label: '请选择开展时间',
        column: 'startDate',
        type: 'date',           // 时间控件
        width: 150
    }, {
        label: '查询更多',
        column: 'chooseMore',
        type: 'link',           // 链接类型
        width: 60,
        onClick: function () {
            console.log("a")
        }
    }],
    // 工具栏按钮
    tools: [{
        name: "导入",
        icon: '&#xe681;',
        type: 'upload',
        url: 'crm/customer/excel',
        exts: 'xls',
        callback: function (res) {
            // 上传类型，上传完成后的回调，推荐后台使用多线程处理
        }
    }，{
        name: "新增",
        icon: '&#xe654;',
        callback: function () {
            // 工具栏按钮
        }
    }],
    // table列
    cols: [[ 
        {type: 'checkbox'} // 多选框
        , {field: 'companyName', title: '单位名称', sort:true, width: 200}
        , {field: 'companyCategory', title: '单位类别', width: 100, type: 'select', enu: 'company_category'} // 此列进行枚举类型转换，值转为名称显示
    ]],
    // 行内操作按钮
    active: [{
        type: "edit",
        url: "",
        dataId: "",
        area: ['600px', '500px'],
        template: 'pages/merchant/add-product.html',
        onOpen: function () {
        }
    }, {
        type: "delete",
        dataId: "",
        url: ""
    }, {
        type: "add",
        url: "",
        area: ['450px', '350px'],
        template: 'pages/merchant/add-product.html',
        onOpen: function () {
        }
    }，{ 
        type: "btn",
        id: "checkBooth",
        value: "审核", // 按钮名称
        onClick: function (row) {
            console.log(row)
        }
    }],
    // 行内事件回调
    onRowEvent: function (obj) {
        var data = obj.data;
        if (obj.event === 'changeStatus') {
            layer.open({
                title: "修改状态"
                , content: '<div class="layui-form">' +
                '<select name="approvalStatus" by-filter="select" by-label="审核状态" by-enutype="approvalStatus" by-default="'+data.approvalStatus+'"></select>' +
                '</div>'
                , success: function () {
                    byselect.render();
                    form.render();

                    // ajax发送请求

                }
            });
        } else if (obj.event === 'changeResult') {

        }
    }
});

```
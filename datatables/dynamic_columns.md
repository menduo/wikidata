---
title: DataTables Dynamic Columns 动态列数据
categories: datatables, jquery
...

### Dynamic Columns 动态列

* 场景：HTML Table 的 thead 处要写各个列头，在 datatables 的 columns 里又要再写一遍。比较麻烦。尤其是，列数据并不需要在 datatables 中的初始化阶段做特殊处理时，只写一遍代码就很好。
* 方案：thead 处用 html 的标签写好每行数据中的 column name，然后再用 javascript 动态生成一个 columns_list 列表，丢给 datatables 初始化函数。
* 注意：thead 里 `data-attr` 的值必须是你的 行数据（rowData）中有的字段，否则会报错，如果只是为了点位，可以为空，并在 `data_key` 处给空数据赋值 `null`。

### HTML
```html
<thead>
    <tr id="tr-in-thead">
        <th data-attr="description">描述</th>
        <th data-attr="credits_value">积分</th>
        <th data-attr="created_at" style="width:15%">创建时间</th>
        <th data-attr="_id">其他</th>
    </tr>
</thead>
```

### JavaSripts
```javascript
var columns_list = []; // 创建一个 列表
var columons_wrapper = $('#tr-in-thead').find("th");
$.each(columons_wrapper, function (idx, item) {
    var data_key = $(item).attr('data-attr') || "_id"; // 也可以把 "_id" 改为 null；
    var obj = {
        "data": data_key
    };
    columns_list.push(obj);
});

function get_column_index_by_data_attr(name) {
    // 通过 column name 来获得它的 index。
    var idx = -1;
    for (var i = 0; i < columns_list.length; i++) {
        if (columns_list[i].data == name) {
            idx = i;
        }
    }
    return idx;
}

var sort_index = get_column_index_by_data_attr("created_at"); // 创建默认排序键
```

### DataTables
```javascript
$('.data-table').dataTable({
    "bJQueryUI": true,
    "bRetrieve": true,
    "bDestory": true,
    "bLengthChange": true,
    "bPaginate": true,
    "responsive": true,
    "sPaginationType": "full_numbers",
    "sDom": '<""l>t<"F"fp>',
    "order": [[sort_index, "desc"]], // 使用由上面自己写的方法读到的 index id 来排序
    "ajax": {
      "url": data_url,
      "dataSrc": "res_list"
    },
    "language": {
      "url": "/static/venders/datatables/i18n/Chinese.json"
    },
    "columns": columns_list, // 直接拿来使用
});
```
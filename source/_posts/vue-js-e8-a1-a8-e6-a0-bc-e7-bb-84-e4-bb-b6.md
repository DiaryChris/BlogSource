---
title: Vue.js 表格组件
url: 178.html
id: 178
categories:
  - 前端
date: 2018-03-24 00:14:12
tags:
---

HTML：

```html
<div id="target"></div>
```

  JS：

```js
//Vue组件开始
var Grid = Vue.extend({
    template: '<table class="table" :class="{layoutFixed: config.layoutFixed}">\
                <thead>\
                    <tr>\
                        <th v-for="col in config.colModel" \
                        v-if="col.sort" \
                        :width="col.width" \
                        :col="col.name" \
                        :title="removeTag(col.display)" >\

                        <div class="sort ellipsis" :sortType="col.sortType">{ {col.display}}\
                        <i class="icon-chevron-up" @click="sortUp"></i>\
                        <i class="icon-chevron-down" @click="sortDown"></i>\
                        </div></th>\

​                        <th v-else \
​                        class="ellipsis"\
​                        :width="col.width" \
​                        :col="col.name" \
​                        :title="removeTag(col.display)" >{ {col.display}}</th>\
​                    </tr>\
​                </thead>\
​                <tbody>\
​                    <tr v-for="row in resData.rows">\
​                        <td v-for="col in config.colModel"\
​                        :title="removeTag(row\[col.name\])" v-handle="{handler:col.handler, v:row\[col.name\], data:row}">{ {row\[col.name\]}}</td>\
​                    </tr>\
​                </tbody>\
​            </table>',
​    data: function () {
​        return {
​            url: '',
​            config: {},
​            param: {},
​            resData: {}
​        };
​    },
​    computed: {

},
methods: {
    getData: function () {
        var self = this;
        $.ajax({
            url: this.url,
            type: 'GET',
            data: this.param,
            success: function (json) {
                json = JSON.parse(json);
                self.resData = json.data;
            }
        });
    },
    sortUp: function (event) {
        $.extend(this.param, {
            sortType: $(event.target).parent().attr('sorttype'),
            orderType: "1"
        });
        this.getData();
    },
    sortDown: function (event) {
        $.extend(this.param, {
            sortType: $(event.target).parent().attr('sorttype'),
            orderType: "2"
        });
        this.getData();
    },
    removeTag: function (str) {
        if (str) {
            str = str.toString();
            return str.replace(/<("\[^"\]*"|'\[^'\]*'|\[^"'>\])>/g, '');
        } else {
            return '';
        }
    },
    log: function (str) {
        console.log(str);
    }
},
directives: {
    handle: {
        inserted: function (el, binding) {
            var handler = binding.value.handler;
            if (handler && $.isFunction(handler)) {
                handler(binding.value.v, binding.value.data, $(el), $(el).parent('tr'));
            }
        }
    }
},
mounted: function () {
    this.getData();
}

});

Vue.component('grid', Grid);
//Vue组件结束

//创建Vue实例开始
var vm = new Grid({
    el: config.renderTo,
    data: {
        url: config.url,
        config: config
    },
    computed: {

}

});
//创建Vue实例结束
```


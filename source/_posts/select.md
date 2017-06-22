---
title: 自建应用字段编辑+JqueryWeUI
date: 2017-06-15 16:53:15
categories:
- JQuery
tags:
- JQuery
- WeUI
- 工作总结
description: 使用JQueryWeUI时遇到select组件没有提供点击蒙版关闭组件，以及不点击确定关闭控件后再次打开控件默认值的问题
---
> 在使用JqueryWeUI做自建应用各种字段编辑交互的时候遇到不少问题

# select控件
+ 点击蒙版关闭控件
　　weui只有日期和时间控件提供了点击蒙版关闭控件的功能,但是在实际应用中select同样十分需要这种方便体验的交互，不过select控件提供了全局打开/关闭的API：
`$("input").select("close")`
　　实现方法是就是获取蒙版元素，给蒙版元素添加点击事件，手动调用`select("close")`
　　代码：
　　```javascript
	$(element).find('.weui_mask').on('click',function (e){
	    e.stopPropagation();
	    scope.value = oldValue
	    scope.cancel = true;
	    $(element).find('input').select("close");
    })
    ```
    这里需要注意的是，`$(element).find('input').select("close");`实际是去执行了初始化时候的onClose方法,同时onClose回调也会执行。
+ 点击选项但不提交，再次打卡控件默认选项问题
　　需求：一个下拉选框，选项ABC，默认选中A选项。
现在选中B选项。
点击蒙版关闭控件，不改变值，并且下次打开控件仍然默认选中A选项。
点击确定关闭控件，并改变值，下次打开控件默认选中B选项。
　　实现方法：select控件在初始化的时候提供了设置初始值的参数`input`，默认`undefiend`。并且提供了更改这些参数的方法`$("input").select("update", { items: ["法官", "猎人", ...] })`。
这两个配合起来使用就能实现这个需求。

　　代码：
　　```javascript
	scope.value = scope.value||'';		//值
    var oldValue = angular.copy(scope.value);
    $(element).find('input').select({
        input:scope.value,
        title: new_title,
        items: scope.list,
        autoClose: false,
        multi: false,
        onChange: function(d) {
            var chooseone= $(element).find('input').attr('data-values');
            scope.value = chooseone;
            scope.current_select = chooseone;
        },
        onOpen: function () {
            scope.cancel = false;
            $(element).find('.weui_mask').addClass('weui_mask_visible')
        },
        onClose: function () {
            scope.$apply(function () {
                oldValue = angular.copy(scope.value);
                scope.valuename = angular.copy(scope.value);		//显示需要
            });
            if(scope.isdetail&&scope.cancel==false){
                scope.submitEdit();			//提交数据
            }
            $(element).find('.weui_mask').removeClass('weui_mask_visible');
        }
    });
    $(element).find('.weui_mask').on('click',function (e) {
        e.stopPropagation();
        scope.value = oldValue
        scope.cancel = true;
        $(element).find('input').select("close");
    })；
    scope.showSelect = function () {
        $(element).find('input').select('update',{
            input:scope.value
        })
		$(element).find('input').select("open");
    }
　　```
---
title: JavaScript 打印网页
key: 20151112
tags: javascript print
---
几乎所有的浏览器都可以实现 `window.print()` 来调用页面打印。但是打印的是全页面的内容，但我们可以用投机取巧的方式来实现局部的打印。

利用注释的标签把要打印的内容包围进去，注意要先择正文不会含有的字符串来用；比如惯用的 `<!--startprint-->` 与 `<!--endprint-->`

```
<!--startprint-->
这里是需要打印的内容，可以带 css 格式
<!--endprint-->
```

要需要打印的地方把页面上的其他文字用 JS 截取掉

```
var bdhtml = window.document.body.innerHTML;
var sprnstr = "<!--startprint-->";
var eprnstr = "<!--endprint-->";
var prnhtml = bdhtml.substring(bdhtml.indexOf(sprnstr) + 17); 
//注意跟使用的标签长度有关
prnhtml = prnhtml.substring(0, prnhtml.indexOf(eprnstr));
window.document.body.innerHTML = prnhtml;
window.print();
```

对于现代化的浏览器，打印的内容的页眉页脚都可以在打印预览的页面中进行勾选取消；但是对于古老的 ie 来说，还需要对注册表进行修改来实现不打印页眉页脚。
修改的注册表的 key 值的位置位于 `HKEY_CURRENT_USER/Software/Microsoft/Internet Explorer/PageSetup/` 里的 `header` 与 `footer` 值，置空则不打印。
也可以使用 activex 来修改这一值，封闭好两个 js 方法，一个是打印前调用不打印页眉页脚，一个是打印后还原。分别在 `window.print()` 前与后调用

```
function PageSetup_Null() {
    try {
       var RegWsh = new ActiveXObject("WScript.Shell");
            RegWsh.RegWrite("HKEY_CURRENT_USER/Software/Microsoft/Internet Explorer/PageSetup/header", "");
            RegWsh.RegWrite("HKEY_CURRENT_USER/Software/Microsoft/Internet Explorer/PageSetup/footer", "");
    } catch (e) {
    }
}
function PageSetup_Default() {
    try {
        var RegWsh = new ActiveXObject("WScript.Shell");
            RegWsh.RegWrite("HKEY_CURRENT_USER/Software/Microsoft/Internet Explorer/PageSetup/header", "&w&b页码，&p/&P");
            RegWsh.RegWrite("HKEY_CURRENT_USER/Software/Microsoft/Internet Explorer/PageSetup/footer", "&u&b&d");
     } catch (e) {
    }
}
```


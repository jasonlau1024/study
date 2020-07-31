
**使用"+"符号模糊匹配：**
如：`mysql + zabbix`等价于`mysql zabbix `模糊匹配mysql或zabbix。

**使用"-"符号排除匹配：**
如：`mysql - zabbix` 匹配mysql排除zabbix的内容。

**使用双引号""精确匹配：**
如：`"mysql install"` 精确匹配`mysql install`的内容

**使用"\*"符号正则匹配：**
如：`my email address is * error` 这里的"\*"可替换任意长度内容。

**使用 inurl 搜索URL：**
如：`inurl:zabbix` 匹配URL包含zabbix的站点。

**intitle 搜索标题：**
如：`intitle:zabbix`  匹配标题包含zabbix的内容。

**allinurl 和 allintitle 匹配多个关键字：**
如：`alltitle:zabbix mysql`

**filetype 搜索指定文件格式文件：**
如：`filetype:pdf mysql

**site 指定站点搜索内容：**
如：`zabbix site:www.178linux.com`

**混合使用搜索：**
如：inurl:www.edu.cn 性能报告 filetype:doc
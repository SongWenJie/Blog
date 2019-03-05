使用系统管理-代码生成，根据数据库表定义生成相关模板代码，生成目录如下：

![mark](http://songwenjie.vip/blog/20190305/PWYJp4ii2K2y.png?imageslim)

### 数据库

执行 book_menu Sql文件，应注意层级对应关系，例如：1000-1600-1601

```sql
-- 该脚本不要执行，请完善 ID 对应关系,注意层级关系 !!!!

-- 菜单SQL
insert into `sys_menu` ( `parent_id`, `component`, `permission`, `type`, `path`, `icon`, `menu_id`, `del_flag`, `create_time`, `sort`, `update_time`, `name`)
    values ( '1000', 'views/book/book/index', '', '0', 'book', 'icon-bangzhushouji', '1600', '0', '2018-01-20 13:17:19', '8', '2018-07-29 13:38:19', '书籍表管理');

-- 菜单对应按钮SQL
insert into `sys_menu` ( `parent_id`, `component`, `permission`, `type`, `path`, `icon`, `menu_id`, `del_flag`, `create_time`, `sort`, `update_time`, `name`)
    values ( '1600', null, 'book_book_add', '1', null, '1', '1601', '0', '2018-05-15 21:35:18', '0', '2018-07-29 13:38:59', '书籍表新增');
insert into `sys_menu` ( `parent_id`, `component`, `permission`, `type`, `path`, `icon`, `menu_id`, `del_flag`, `create_time`, `sort`, `update_time`, `name`)
    values ( '1600', null, 'book_book_edit', '1', null, '1', '1602', '0', '2018-05-15 21:35:18', '1', '2018-07-29 13:38:59', '书籍表修改');
insert into `sys_menu` ( `parent_id`, `component`, `permission`, `type`, `path`, `icon`, `menu_id`, `del_flag`, `create_time`, `sort`, `update_time`, `name`)
    values ( '1600', null, 'book_book_del', '1', null, '1', '1603', '0', '2018-05-15 21:35:18', '2', '2018-07-29 13:38:59', '书籍表删除');
```

### 前台项目

将 galaxy-ui 目录中的 src 目录  copy 到 UI 项目的主目录下

为了便于项目的管理和后期维护，建立相关文件夹，将生成的代码文件置于相关文件夹中

![mark](http://songwenjie.vip/blog/20190305/djVqnzuln2Hq.png?imageslim)=>![mark](http://songwenjie.vip/blog/20190305/WYgvsklpC3f3.png?imageslim)



![mark](http://songwenjie.vip/blog/20190305/S0Hg1BEHVFJL.png?imageslim)=>![mark](http://songwenjie.vip/blog/20190305/fSkE9d5Sbty4.png?imageslim)

因为改变了项目代码文件位置，需要在 vue 文件中修改依赖文件的路径

![mark](http://songwenjie.vip/blog/20190305/QGTcijuVQzbX.png?imageslim)=>

![mark](http://songwenjie.vip/blog/20190305/LTWmQmvtAUDM.png?imageslim)

因为项目采用前后端分离，涉及到跨域问题，所以需要配置转发代理

具体操作为在项目主目录下的 vue.config.js 文件中`devServer:proxy:`节点下添加如下配置

```javascript
'/book': {
        target: url,
        ws: true,
        pathRewrite: {
          '^/book': '/book'
        }
      }
```

![mark](http://songwenjie.vip/blog/20190305/y8AyWT7cLns9.png?imageslim)

配置完成后需要重启项目 `npm run dev`

### 后台项目

将 galaxy 目录中的 src 目录  copy 到后台项目的主目录下

确认 controller 中的路由是否和前台项目匹配，采用 Restful 风格架构

前台：![mark](http://songwenjie.vip/blog/20190305/MrTnVHpeazOK.png?imageslim)

后台：![mark](http://songwenjie.vip/blog/20190305/DjSm4WejA8HP.png?imageslim)



### 测试

启动前后台项目，权限管理-角色管理 配置角色权限，配置完成后使用拥有此菜单权限的角色账号登录系统，便可以在相应模块下使用书籍表管理。

![mark](http://songwenjie.vip/blog/20190305/gzmAwFYF0JET.png?imageslim)

如果出现404页面，按照如下顺序排查：

1. 存在缓存，浏览器刷新缓存
2. 没有配置转发代理（配置完成后需要重启项目）
3. 前后端路由不对应
4. 其他原因
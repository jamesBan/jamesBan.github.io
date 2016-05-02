title: mysql多值查询
date: 2015-03-23 19:28:06
tags:
- mysql
- sphinx
---

现有如下标签和文章对象表``fy_content_tag`` 如下:

        CREATE TABLE `fy_content_tag` (
            `id` INT(11) NOT NULL,
            `tagid` INT(11) NOT NULL
        )
        COLLATE='utf8_unicode_ci'
        ENGINE=InnoDB;

插入数据

    INSERT INTO `fy_content_tag` (`id`, `tagid`) VALUES
        (1, 1),(1, 2),(1, 3),(1, 4),
        (2, 2),(2, 3),(2, 4),(2, 5),(2, 6),
        (3, 3),(3, 4), (3, 5),(3, 6),(3, 7),
        (4, 4),(4, 5),(4, 6),(4, 7),(4, 8),(5, 5),(5, 6),(5, 7),(5, 8),(5, 9);


需要查询tagid同时包含2、3、4的id

##使用sql行变列



    SELECT id, SUM(CASE tagid WHEN 2 THEN 1 WHEN 3 THEN 1 WHEN 4 THEN 1 ELSE 0 END) AS tag_id
    FROM fy_content_tag
    WHERE 1
    GROUP BY id
    HAVING tag_id = 3;


## 使用sphinx的多值

### 配置
参考[sphinx设置多属性过滤的方法](http://blog.sina.com.cn/s/blog_7eef675d0101fimb.html)
在sphinx 配置文件source 内添加多值属性定义:

    sql_attr_multi = uint tagid from query;\
        SELECT id,tagid FROM fy_content_tag

### 使用phpapi查询
    
    //查出拥有标签2的文档
    $sphinx->setFilter('tagid', array(2));

    查出同时拥有标签2,3,4的文档
    $sphinx->setFilter('tagid', array(2));
    $sphinx->setFilter('tagid', array(3));
    $sphinx->setFilter('tagid', array(4));


如果标签值(tagid)为1,2,3也可以设置为

    sql_attr_multi = uint tagid from filed


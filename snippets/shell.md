### find 命令使用

```shell
find / -name persistence.xml |xargs grep "create-or-extend-tables"

# 查询Java文件，包含365字段的
find . -name "*.java" |xargs grep "365"	

find . -name pom.xml |xargs grep "guava"
```


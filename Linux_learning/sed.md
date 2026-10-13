# sed '地址 命令' 文件
## sed 默认所有行都输出，只是匹配的行被改过
## 地址省略 → 每行都执行
## 地址可以是：行号 3、范围 2,5、正则 /gene/
```
### s/旧/新/	每行第一个"旧"替换为"新"
### s/旧/新/g	每行所有"旧"都替换
### d	删除该行
### -n '...p'	只打印指定的行
sed 's/chr/Chr/g' file      # 所有 chr 替换为 Chr
sed '1d' file               # 删除第 1 行（去表头）
sed '/^#/d' file            # 删除以 # 开头的行（去注释）
sed -n '10,20p' file        # 只打印第 10~20 行
```
```
# 只输出不做任何修改，就是''必须存在，也就是未必有地址和命令传入，但是得让Sed知道不传入不修改只输出
sed '' quote.txt
# -e指令它的作用是"后面跟着一条 sed 命令"，比如删除指定的行
cat books.txt
# 1) A Storm of Swords, George R. R. Martin, 1216
# 2) The Two Towers, J. R. R. Tolkien, 352
# 3) The Alchemist, Paulo Coelho, 197
# 4) The Fellowship of the Ring, J. R. R. Tolkien, 432
# 5) The Pilgrimage, Paulo Coelho, 288
# 6) A Game of Thrones, George R. R. Martin, 864
sed -e '1d' -e '2d' books.txt
# 3) The Alchemist, Paulo Coelho, 197
# 4) The Fellowship of the Ring, J. R. R. Tolkien, 432
# 5) The Pilgrimage, Paulo Coelho, 288
# 6) A Game of Thrones, George R. R. Martin, 864
```

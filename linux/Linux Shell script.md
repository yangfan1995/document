# Shell script

## 概括
帮我们把一大串的指令汇整在一个文件里面， 而直接执行该文件就可以执行那一串又臭又长的指令段，Shell scripts 的速度较慢，且使用的 CPU 资源较多，造成主机资源的分配不良。

## 执行差异
script 主要有三种source, sh script, ./script执行方式
sh script, ./script两种方式是在子bash中执行，在父bash中取不到子bash中的变量。

## 默认变量
Shell script 默认的${0},${1},${2},${3},使用$@取得所有参数，可以使用shift进行参数偏移，shift number 代表跳过前number个参数

## 常用结构

### 选择
1. if结构
```bash
if [[ condition ]]; then
	#statements
	echo -e "test"
fi
```
2. case结构
```bash
case word in
	pattern )
		;;
esac
```
`*`在条件中相当于default

### function,函数
```bash
function functionName(){
    #statements
    echo -e "test"
}
```
函数中的${1}指的是调用参数的第一个，与Shell script 的第一个参数没有关系

### 循环
1. while do done
```bash
while [[ condition ]]; do
	#statements
	echo -e "test"
done
```
2. until do done
```bash
until [[ condition ]]; do
	#statements
	echo -e "test"
done
```
3. for in do done
```bash
for i in words; do
	#statements
	echo -e "test"
done
```
4. for ( () )
```bash
for ((i=0; i<100; i=i+1))
do
    #statements
    echo -e "test"
done
```

## 调试
```
sh [-nvx] scripts.sh
    -n :不要执行 script，仅查询语法的问题；
    -v :再执行 sccript 前，先将 scripts 的内容输出到屏幕上；
    -x :将使用到的 script 内容显示到屏幕上，这是很有用的参数！
```
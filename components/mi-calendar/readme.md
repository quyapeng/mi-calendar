
## mi-calendar 使用帮助

### 快速开始
使用 uni_modules 安装 (目前仅支持 uni_modules)

下载该文件后，放到uni_modules文件下，然后直接页面使用。

<mi-calendar :leaveDateList="[`2022-1-2`,`2022-1-4`]" :suspensionDateList="[`2022-01-05`]" :normalDateList="[`2022-1-3`]"></mi-calendar>

## leaveDateList
请假日期
## suspensionDateList
停课日期
## normalDateList
正常考勤日期

## change 切换日期
`@change="change"`
```
change(date){
    console.log(date); // 日期 eg：'2022-01-01'
}
```

## changeMonth 切换月份
`@changeMonth="changeMonth"`
```
changeMonth(year, month){
    console.log(year, month); // 日期 eg：2022, 6
}
```



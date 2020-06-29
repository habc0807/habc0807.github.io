---
title: '如何开发一个请假时长计算器'
date: 2020-06-29 11:26:12
tags:
---

这是一个简单的请假时长计算器，最终封装好的 [npm包：leave-days-calculator](https://www.npmjs.com/package/leave-days-calculator)。

## leave-days-calculator的安装和使用

### 安装

```javascript
$ npm install leave-days-calculator
```

### 引入

```javascript
import leaveDaysCalculator form 'leave-days-calculator'
```

### 使用

```javascript
const start = '2020-06-22 上午'
const end = '2020-06-24 下午'
let leaveDays = leaveDaysCalculator(start, end) // 3
```
> 时间的格式只支持: YYYY-MM-DD 上午/下午


如果觉得喜欢，能帮助到你就start一下，
[github](https://github.com/habc0807/leave-days-calculator)。


## 需求背景

自从做了小程序外勤打卡需求后，紧接着继续做请假审批流程的开发，在处理请假表单的时候，有个请假时长的展示。当用户选择了请假的开始时间和结束时间后，可以自动推算出请假时长，听上去，需求很常规。但是，哈哈哈，产品要求所有请假都按照半天请，比如说：

请假的开始时间是6月1号上午，结束是6月2号上午，并不是结束时间减去开始时间那么简单。上午到上午的这种情况，多了半天呐。😂😂😂

那么问题来了，如何准确的计算请假时长？

## 解决方案

> 最终采用了方案一

### 方案一：
按照当前请假，非当前请假两类计算，

如果是当天请假：
- 当天开始时间上午 - 当天结束时间上午，请假时长 0.5d;
- 当天开始时间上午 - 当天结束时间下午，请假时长 1.0d;
- 当天开始时间下午 - 当天结束时间上午，报错警告处理;
- 当天开始时间下午 - 当天结束时间下午，请假时长 0.5d;

如果是隔天计算：
结束时间 - 开始时间 = gap
- 结束时间上午，开始时间上午，gap + 0.5
- 结束时间上午，开始时间下午，gap + 1
- 结束时间下午，开始时间上午，gap + 0
- 结束时间下午，开始时间下午，gap + 0.5

此方案的核心是需要准确的计算出gap，可以使用第三方工具来处理。从计算量上来看，工作量比较小。

> 我师傅瞅了一眼我写的代码，可能实在看不下去，我写了一推冗余代码，就给我写了一版，微调后的方案一代码如下：

```javascript
const getLeaveDays = (start, end) => {
    const [times, tags] = start.split(' ')
    const [timet, tagt] = end.split(' ')
    const startStr = +new Date(...times.split('-'))
    const endStr = +new Date(...timet.split('-'))
    const timeDiff1 = tagt === tags ? 0.5 : {'上午': 'error', '下午': 1 }[tagt] // 当天
    const timeDiff2 = tagt === tags ? 0.5 : {'上午': 0, '下午': 1 }[tagt] // 隔天
    
    if(startStr > endStr) {
        return 'error'
    }
    else if(timet === times) {
        return timeDiff1
    } 
    else {
        return (endStr - startStr) / (60 * 60 * 24 * 1000) + timeDiff2
    }
}
```

### 方案二：
纯手动计算，按照当天上下午，隔天上下午，隔月上下午，隔年上下午手动计算出请假时长。这个方案的开发时长比较长，在开发周期比较宽裕的情况下我选择了方案，因为我还很想知道具体是这么计算的。给自己找点事干，给枯燥的coding，增添点趣味。

## 手动实现一个请假时长计算器
基于开始时间和结束时间的格式都是 `YYYY-MM-DD 上午/下午`。

图一：流程图


算起来有四种计算规则：
- 同年同月同日请假时长计算
- 同年同月非同日请假时长计算
- 同年非同月请假时长计算
- 非同年请假时长计算

![1.jpg](https://user-gold-cdn.xitu.io/2020/6/1/1726f3c6e4f18700?w=1000&h=557&f=jpeg&s=44123)

需要一些基础工具函数，比如说：

- 是否是闰年
- 某年的各个月份的天数
- 去0函数
- 加0函数

业务公共方法：
- 开始时间，年月日
- 结束时间，年月是
- 同一天请假天数计算方法
- 隔天请假天数计算方法
- 开始月份总天数，开始月份请假天数计算方法 （跨月，年才会用到的函数）
- 结束月份请假天数计算方法（跨月，年才会用到的函数）
- 中间月份天数累计器
- 开始年的请假天数计算: 开始月剩下的天数 + 剩下月份的天数
- 结束年的请假天数计算: 结束月的天数 + 结束月之前月份的天数
- 中间年份的天数累计器

以下是请假时长计算器的源码，仅供参考。

```javascript
/**
* @Authors habc0807 (gao0807@foxmail.com)
* @Date: 2020-06-1

* @Last Modified by: habc0807
* 请假时长计算器：（通过 （开始时间 + 结束时间）计算请假时长）
*/

/**
 * 是否是闰年
 * @param {*} year 
 */
function isLeapYear (year) {
    return year % 100 !== 0 && year % 4 === 0 || year % 400 === 0
}


/**
 * 获取每年每月的天数
 * @param {*} year 
 * @param {*} month 
 */
function getMaxDay (year, month) {
    year = parseFloat(year)
    month = parseFloat(month)
    if (month === 2) {
        return isLeapYear(year) ? 29 : 28
    }
    return [4, 6, 9, 11].indexOf(month) >= 0 ? 30 : 31
}


// 去0
function trimZero (val) {
    val = String(val)
    val = val ? parseFloat(val.replace(/^0+/g, '')) : ''
    val = val || 0
    val = val + ''
    return val
}

/**
 * 同一天请假
 */
function sameDayLeave(startNoon, endNoon) {
    let leaveDays = 0
    if (startNoon === endNoon) {
        leaveDays = 0.5
    } 
    else if (startNoon === '上午' && endNoon === '下午') {
        leaveDays = 1
    } 
    else if (startNoon === '下午' && endNoon === '上午') {
        console.log('您选择的时间有误')
    }
    return leaveDays
}


/**
 * 隔天请假
 */
function nextDayLeave(startDay, endDay, startNoon, endNoon) {
    let leaveDays = 0
    if (startNoon === '上午' && endNoon === '上午') {
        leaveDays = endDay - startDay + 0.5 
    } 
    else if (startNoon === '上午' && endNoon === '下午') {
        leaveDays = endDay - startDay + 1 
    } 
    else if (startNoon === '下午' && endNoon === '上午') {
        leaveDays = endDay - startDay
    } 
    else if (startNoon === '下午' && endNoon === '下午') {
        leaveDays = endDay - startDay + 0.5
    }
    return leaveDays
}


/**
 * 开始月份请假天数 跨月，年才会用到的函数
 */
function startMonthLeaveDays (startMonthAllDays, startDay, startNoon) {
    let startMonthLeave = 0
    if (startNoon === '上午') {
        startMonthLeave = Number(startMonthAllDays) -  Number(startDay) + 1
    } else if (startNoon === '下午') {
        startMonthLeave = Number(startMonthAllDays) -  Number(startDay) + 0.5
    }
    return Number(startMonthLeave)
}


/**
 * 结束月的请假天数 跨月，年才会用到的函数
 */
function endMonthLeaveDays(endDay, endNoon) {
    let endLeaveDays = 0
    if (endNoon === '上午') {
        endLeaveDays = endDay - 0.5
    } else if (endNoon === '下午') {
        endLeaveDays = endDay
    }
    return Number(endLeaveDays)
}


/**
 * 通过开始时间和结束时间，计算请假时长
 * @param {*} start 【支持：格式 yyyy-mm-dd n】
 * @param {*} end 【支持：格式 yyyy-mm-dd n】
 */
export default (start, end) => {
    if(!start || !end) return 0

    const startArr = start.split(' ') // 开始时间
    const endArr = end.split(' ') // 结束时间
    const startDate = startArr[0] // 开始日期
    const endDate = endArr[0] // 结束日期


    const startDateArr = startDate.split('-') // 开始日期 数组化
    const endDateArr = endDate.split('-') // 结束日期 数组化


    const startYear = startDateArr[0] // 开始日期：年
    const startMonth = startDateArr[1] // 开始日期：月
    const startDay = startDateArr[2] // 开始日期：日
    const startNoon = startArr[1] // 开始时间的上下午


    const endYear = endDateArr[0] // 结束日期：年
    const endMonth = endDateArr[1] // 结束日期：月
    const endDay = endDateArr[2] // 结束日期：日
    const endNoon = endArr[1] // 结束时间的上下午


    // 开始月份的总天数
    const startMonthAllDays = getMaxDay(startYear, startMonth)


    /**
     * 中间月份的请假天数计算
     */
    function centerMonthsLeave() {
        // 中间月份天数累加
        let leaveDays = 0
        let monthArr = [] // 月份数组
        for(let i = Number(trimZero(startMonth)); i <= Number(trimZero(endMonth)); i++) {
            monthArr.push(i)
        }

        let everyMonthDays = 0
        if (monthArr.length > 2) {
            monthArr.pop()
            monthArr.shift()
            
            monthArr.forEach(month => {
                everyMonthDays = Number(everyMonthDays) + Number(getMaxDay(startYear, month))
            })
        }
        return (Number(leaveDays) + Number(everyMonthDays)).toFixed(1)
    }


    /**
     * 开始月份和结束月份请假天数  相邻月请假，或则跨月请假
     */
    function startEndMonthNext() {
        // 开始月份请假天数计算
        let startLeaveDays = startMonthLeaveDays(startMonthAllDays, startDay, startNoon) 
        
        // 结束月份请假天数计算
        let endLeaveDays = endMonthLeaveDays(endDay, endNoon)

        // 天数累加
        return Number(startLeaveDays) + Number(endLeaveDays) || 0
    }


    /**
     * 开始年的请假天数计算
     */
    function startYearLeaveDays () {
        // 开始月份天数计算
        let startMonthDays = startMonthLeaveDays(startMonthAllDays, startDay, startNoon) 

        // 剩下月份的天数
        let otherMonthDays = 0
        for(let i = Number(trimZero(startMonth)) + 1;  i <= 12; i++) {
            otherMonthDays = Number(otherMonthDays) + Number(getMaxDay(startYear, i))
        }

        return Number(startMonthDays) + Number(otherMonthDays) || 0
    }


    /**
     * 结束年的请假天数计算
     */
    function endYearLeaveDays() {
        let endYearDays = 0
        // 结束月份计算
        let endLeaveDays = endMonthLeaveDays(endDay, endNoon)

        // 前几个月份天数累加
        for(let i = 1; i < trimZero(endMonth); i++) {
            endYearDays = Number(endYearDays) + Number(getMaxDay(endYear, i))
        }

        return Number(endYearDays) + Number(endLeaveDays)
    }


    /**
     * 跨年，中间年假期计算
     */
    function centerYearsLeaveDays () {
        let centerYear = 0
        if(endYear - startYear > 1) {
            let centerYears = []
            for(let startYear; startYear < endYear; startYear++) {
                centerYears.push(startYear)
            }
            centerYears.pop()
            centerYears.shift()
            centerYears.forEach(year => {
                for(let month = 1; month <=12; month++) {
                    centerYear = centerYear + getMaxDay(year, month)
                }
            })
        }
        return centerYear
    }

    // 请假天数
    let leaveAllDays = 0
    // 是否同年
    if (startYear === endYear) {
        // 是否同月
        if (startMonth === endMonth) {
            // 是否同天
            if (startDay === endDay) {
                leaveAllDays = sameDayLeave(startNoon, endNoon)

            // 同年 同月 隔天请假计算
            } else if (startDay < endDay) {
                leaveAllDays = nextDayLeave(startDay, endDay, startNoon, endNoon)

            } else {
                console.log('结束时间的天不对')
            }

        // 同年 不是同月
        } else if (startMonth < endMonth) {
            // 两个月相邻情况, 开始月份剩余天数，结束月份请假天数 之和
            let startEndMonthLeaveDays = startEndMonthNext()

            // 中间月份请假的天数
            let centerMonthsDays = centerMonthsLeave()

            // 请假天数汇总
            leaveAllDays = Number(startEndMonthLeaveDays) + Number(centerMonthsDays)
        } else {
            console.log('结束时间的月份不对')
        }

    // 不是同年
    } else if (startYear < endYear) {
        // 开始年的请假天数计算: 开始月剩下的天数 + 剩下月份的天数
        let startYearDays = startYearLeaveDays()

        // 结束年的请假天数计算: 结束月的天数 + 结束月之前月份的天数
        let endYearDays = endYearLeaveDays()

        // 中间年份的天数
        let centerYear = centerYearsLeaveDays()
        
        // 请假天数汇总累加
        leaveAllDays = Number(startYearDays) + Number(endYearDays) + Number(centerYear)
    } else {
        console.log('结束时间的年份不对')
    }

    // console.log(leaveAllDays)s
    return leaveAllDays
}
```

一个请假时长计算器就开发完了，其实方案一更省事一点，方案二的实现，还有很多可以优化的点。最终封装好的请假时长计算器 [npm包：leave-days-calculator](https://www.npmjs.com/package/leave-days-calculator)。

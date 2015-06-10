+++
date = "2015-06-10T17:39:04+08:00"
draft = false
title = "Snippets"

+++

## 背景  
记录常用的比较有技巧的` code snippets`  
### 如何保留float 制定的小数位数  
#### 背景 
默认情况下线 float64 会有很长的小数, 在监控系统的展示中不是非常友好. 因此希望把**0.3333333** 结果截取为 **0.33** , 然后提交到 server.    
#### 实现 

```
    func SetPrecision(from float64, precision int) float64 {
        base := math.Pow10(precision)
        return float64(int64(from*base)) / base
    }
```


## iOS面试败北感悟

- 自实现pow(double, double)

- findMedianSortedArrays (找到两个排序数组的中位数)

- UIContorl -> UIButton

#### 自实现pow(double, double)


```
func _pow_3(_ base: Double, _ exponent: Int) -> Double {

    var isNegative = false
    var exp = exponent
    if exp < 0 {
        isNegative = true
        exp = -exp
    }
    let result = _pow_2(base, exp)
    return isNegative ? 1 / result : result
}

```

#### findMedianSortedArrays (找到两个排序数组的中位数)

```

func findMedianSortedArrays_3(_ array1: [Int], _ array2: [Int]) -> Double {
    
    let total = array1.count + array2.count
    let index = total / 2
    let count = array1.count < array2.count ? array2.count : array1.count
    var array = [Int]()
    
    var i = 0, j = 0;
    for _ in 0...count {
        if array.count >= index + 1 {
            break
        }
        if array1[i] < array2[j] {
            array.append(array1[i])
            i += 1
        } else {
            array.append(array2[j])
            j += 1
        }
    }
    return total % 2 == 1 ? Double(array[index]) : Double(array[index] + array[index - 1]) * 0.5
}

```

#### UIContorl -> UIButton

```
UIButton setTitle的时候才会创建UILabel, setImage的时候才会创建UIImageView, 你为什么吧frame给写死... 不知道UIView有sizeToFit么, 你怎么不实现sizeThatFits, 你是完全不会用吧... 你知道UIButton用AutoLayout布局的时候只要设置origin坐标, 宽高就可以自适应了, 你自定义的时候怎么不实现呢? setBackgroundImage和setImageEdgeInsets
```

- 网络架构实现
	- AFN 

## 链接

- [面试题系列目录](../README.md)
- **上一份**: [一个渣硕iOS春招总结](interview-iOS-9-一个渣硕iOS春招总结.md)
- **下一份**: [如何在天猫、蚂蚁金服、百度等大厂面试中被拒](interview-iOS-11-如何在天猫、蚂蚁金服、百度等大厂面试中被拒.md)

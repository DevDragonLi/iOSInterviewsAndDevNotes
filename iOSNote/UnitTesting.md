# Unit Testing


### 前言

- `维基百科` 模块测试
- 通常来说,程序修改一次就至少需要一次单元测试
- 一般开发人员编写 (敏捷开发注重)
- 测试是技术,但更是文化,测试你的软件,不然你的用户就要测试.


### 单元测试流程
 - 需求分析 -> 流程设计-> 技术选型-> 实现架构(方案)-> 编码-> 高保真实现-> `测试`-> 重构
 - 在编码前 ,我们会罗列功能点,我们自己加以验证,对这些功能细化,就属于测试用例
 
### 出发点

- 正常使用覆盖足够多的场景,(错误场景程序依然足够健壮)
- 异步API调用在多线程下没有异常
- 良好的性能

- `Mock` 

### 测试用例: `条件`.`动作`.`结果`

- 编写条件和动作,设置预期结果然后系统决定最终的结果.

### 好处

- 发现代码缺陷 ,职称不断重构和演进
- 会让我们的代码更加模块化,低耦合


### 开发性质

- UI  
- 业务逻辑  `部分`
- SDK/公共组件/能力层开发  `一定要有对应单元测试`


### iOS单元测试代码部分
- 一些常见的宏


```
//      XCTAssertNil(<#expression, ...#>) //  expression为空测试通过不然失败,接受id 类型数据
//      XCTAssertNotNil(<#expression, ...#>) // expression不为空测试通过不然失败,接受id 类型数据
    
//    XCTAssert(<#expression, ...#>)// expression为true测试通过不然失败,接受 bool 类型数据
//    XCTAssertFalse(<#expression, ...#>) expression为false测试通过不然失败,接受 bool 类型数据

    //    XCTAssertEqualObjects(<#expression1#>, <#expression2, ...#>) expression1 和expression2 地址相同就是通过
//    XCTAssertNotEqual(<#expression1#>, <#expression2, ...#>)expression1 和expression2 地址不相同就是通过

//    XCTAssertEqual(<#expression1#>, <#expression2, ...#>)expression1 和expression2 相等就是通过,接收基本数据类型
//    XCTAssertNotEqual(<#expression1#>, <#expression2, ...#>)expression1 和expression2 相等就是通过

//XCTAssertEqualWithAccuracy(<#expression1#>, <#expression2#>, <#accuracy, ...#>) 如果expression1 和expression2都打与accuracy 测试失败   接收基本类型

//XCTAssertGreaterThan(<#expression1#>, <#expression2, ...#>) expression1 <=expression2 失败
//XCTAssertGreaterThanOrEqual(<#expression1#>, <#expression2, ...#>) expression1 < expression2 失败
//  XCTAssertLessThan(<#expression1#>, <#expression2, ...#>)  expression1 >expression2 失败
    
// 抛异常
//    XCTAssertThrows(<#expression, ...#>) 没抛异常,测试失败  expression 是一个表达式
  

```


```
  @weakify_self;
    [self.testItem addKVOForPath:@"desc" withBlock:^(id newValue) {
        @strongify_self;
        XCTAssertEqualObjects(newValue, @"A cup of wine");
        [KVOExpectation fulfill]; // 被调用才是成功
    }];
    
    self.testItem.desc = @"A cup of wine";
    
    [self waitForExpectationsWithTimeout:0.001f handler:^(NSError *error) {
        [weakSelf.testItem removeAllKVOs];
    }];


```


```
 NSInteger targetCount = 1000;
    @weakify_self;
    [self measureBlock:^
    {
        @strongify_self;
        for (NSUInteger i = 0; i < targetCount; i++)
        {
            @autoreleasepool
            {
                [self.testItem addKVOForPath:@"weight" withBlock:^(id newValue)
                {
                    //NSNumber *weight = (NSNumber *)newValue;
                }];
                [self.testItem removeAllKVOs];
            }
        }
    }];

```








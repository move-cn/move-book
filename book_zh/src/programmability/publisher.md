# 发布者权限

在应用程序设计和开发中，证明发布者的权限往往是必要的。这在数字资产的上下文中特别重要，因为发布者可能会为其资产启用或禁用某些功能。发布者对象是一个对象，在[Sui Framework](./sui-framework.md)中定义，允许发布者证明其对类型的_权威_。

## 定义

发布者对象在Sui框架的`sui::package`模块中定义。它是一个非常简单的、非泛型对象，可以每个模块初始化一次（每个包多次），用于证明发布者对类型的权威。为了声明一个发布者对象，发布者必须向`package::claim`函数提供一个[一次性见证](./one-time-witness.md)。

```move
// File: sui-framework/sources/package.move
public struct Publisher has key, store {
    id: UID,
    package: String,
    module_name: String,
}
```

> 如果您不熟悉一次性见证，可以在[这里](./one-time-witness.md)阅读更多信息。

下面是一个在模块中声明`Publisher`对象的简单示例：

```move
{{#include ../../../packages/samples/sources/programmability/publisher.move:publisher}}
```

## 使用

发布者对象有两个关联的函数，用于证明发布者对类型的权威：

```move
{{#include ../../../packages/samples/sources/programmability/publisher.move:use_publisher}}
```

## 发布者作为管理员角色

对于小型应用程序或简单的用例，发布者对象可以用作管理员[能力](./capability.md)。虽然在更广泛的上下文中，发布者对象对系统配置具有控制权，但它也可以用于管理应用程序的状态。

```move
{{#include ../../../packages/samples/sources/programmability/publisher.move:publisher_as_admin}}
```

然而，发布者对象缺少一些[能力](./capability.md)的本地属性，如类型安全和表达性。`admin_action`的签名不是很明确，可以被其他人调用。由于`Publisher`对象是标准的，如果不执行`from_module`检查，现在存在未经授权访问的风险。因此，在将`Publisher`对象用作管理员角色时需要谨慎。

## 在Sui中的角色

在Sui上，发布者对于某些功能是必需的。[对象展示](./display.md)只能由发布者创建，TransferPolicy——Kiosk系统的重要组成部分——也需要发布者对象来证明类型的所有权。

## 下一步

在下一章中，我们将介绍需要发布者对象的第一个功能——对象展示——一种描述客户端对象并标准化元数据的方法。这是用户友好应用程序的必备功能。
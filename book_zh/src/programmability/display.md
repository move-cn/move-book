# 对象显示

在Sui上，对象的结构和行为是明确的，可以以易于理解的方式显示。然而，为了支持客户机的丰富元数据，定义了一种标准且高效的方式来“描述”对象，即[Sui框架](./sui-framework.md)中的`Display`对象。

## 背景

历史上曾有不同的尝试来达成对象结构的标准，以便可以在用户界面中显示对象。其中一种方法是定义对象结构中的某些字段，当这些字段存在时，会在UI中使用。然而，这种方法不够灵活，需要开发人员在每个对象中定义相同的字段，有时这些字段对对象来说没有意义。

```move
{{#include ../../../packages/samples/sources/programmability/display.move:background}}
```

如果字段包含静态数据，每个对象中都会重复这些数据。而且，由于Move没有接口，无法知道一个对象是否有特定字段，这使得客户端的获取过程更复杂。

## 对象显示

为了解决这些问题，Sui引入了一种标准方式来描述对象的显示。与其在对象结构中定义字段，不如将显示元数据存储在一个单独的对象中，并与类型关联。这样，显示元数据不会重复，且易于扩展和维护。

Sui Display的另一个重要特性是能够定义模板并在这些模板中使用对象字段。这不仅允许更灵活的显示，还解放了开发人员不必在每个对象中定义相同的字段和类型。

> Sui Fullnode本机支持对象显示，如果对象类型关联了Display，客户端可以获取显示元数据。

```move
{{#include ../../../packages/samples/sources/programmability/display.move:hero}}
```

## 创建者特权

虽然对象可以由账户拥有并可能属于[真正的所有权](./../object/ownership.md#account-owner-or-single-owner)范畴，但Display可以由对象的创建者拥有。这样，创建者可以更新显示元数据，并全局应用更改而不需要更新每个对象。创建者还可以将Display转移给其他账户，甚至围绕对象构建一个应用程序来管理元数据。

## 标准字段

最广泛支持的字段包括：

- `name` - 对象的名称。当用户查看对象时显示。
- `description` - 对象的描述。当用户查看对象时显示。
- `link` - 在应用程序中使用的对象链接。
- `image_url` - 对象的图像的URL或Blob。
- `thumbnail_url` - 在钱包、浏览器和其他产品中用作预览的小图像的URL。
- `project_url` - 与对象或创建者相关的网站链接。
- `creator` - 表示对象创建者的字符串。

> 请参考[Sui文档](https://docs.sui.io/standards/display)获取最新支持的字段列表。

尽管有一套标准字段，Display对象并不强制执行这些字段。开发人员可以定义他们需要的任何字段，客户端可以根据需要使用它们。一些应用程序可能需要额外的字段，而忽略其他字段，Display足够灵活以支持这些需求。

## 使用Display

`Display`对象在`sui::display`模块中定义。它是一个泛型结构，接受一个虚拟类型作为参数。虚拟类型用于将`Display`对象与其描述的类型关联。`Display`对象的`fields`是键值对的`VecMap`，其中键是字段名，值是字段值。`version`字段用于显示元数据的版本，并在`update_display`调用时更新。

文件：sui-framework/sources/display.move

```move
struct Display<phantom T: key> has key, store {
    id: UID,
    /// 包含显示字段。目前支持的字段有：name、link、image和description。
    fields: VecMap<String, String>,
    /// 版本只能由发布者手动更新。
    version: u16
}
```

[Publisher](./publisher.md)对象需要一个新的Display，因为它作为类型的所有权证明。

## 模板语法

目前，Display支持简单的字符串插值，并可以在其模板中使用结构字段（和路径）。语法很简单 - `{path}`替换为该路径字段的值。路径是一个以点分隔的字段名列表，针对嵌套字段从根对象开始。

```move
{{#include ../../../packages/samples/sources/programmability/display.move:nested}}
```

上述`LittlePony`类型的Display可以定义如下：

```json
{
  "name": "Just a pony",
  "image_url": "{image_url}",
  "description": "{metadata.description}"
}
```

## 多个Display对象

对于特定的`T`类型，可以创建任意数量的`Display<T>`对象。然而，fullnode将使用最近更新的`Display<T>`。

## 进一步阅读

- [Sui对象显示](https://docs.sui.io/standards/display) 是Sui文档中的一部分
- [Publisher](./publisher.md) - 创建者的表示
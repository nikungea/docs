# JSON
JSON对象包含两个方法: 用于解析 JavaScript Object Notation  (JSON) 的 parse() 方法，以及将对象/值转换为 JSON字符串的 stringify() 方法。除了这两个方法, JSON这个对象本身并没有其他作用，也不能被调用或者作为构造函数调用。

## 描述

JSON 是一种语法，用来序列化对象、数组、数值、字符串、布尔值和 null 。它基于 JavaScript 语法，但与之不同：JavaScript不是JSON，JSON也不是JavaScript。

|  JS类型   | JSON  |
|  ----  | ----  |
| 对象和数组  | 属性名称必须是双引号括起来的字符串；最后一个属性后不能有逗号。 |
| 数值  | 禁止出现前导零（ `JSON.stringify` 方法自动忽略前导零，而在 `JSON.parse` 方法中将会抛出 SyntaxError）；如果有小数点, 则后面至少跟着一位数字。 |
| 字符串  | 字符串必须用双引号括起来。 |

## 方法

### `JSON.stringify()` 

将一个 JavaScript 值（对象或者数组）转换为一个 JSON 字符串，如果指定了 replacer 是一个函数，则可以选择性地替换值，或者如果指定了 replacer 是一个数组，则可选择性地仅包含数组指定的属性。

#### 语法

``` js
JSON.stringify(value[, replacer[, space]])
```

`参数` :

* value
  +  要序列化成 一个 JSON 字符串的值。
* replacer(可选)
  + 如果该参数是一个函数，则在序列化过程中，被序列化的值的每个属性都会经过该函数的转换和处理，如若返回 undefined 或者一个函数，将会被 null 取代。
  + 如果该参数是一个数组，则只有包含在这个数组中的属性名才会被序列化到最终的 JSON 字符串中；
  + 如果该参数为 null 或者未提供，则对象所有的属性都会被序列化；
* space 
  + 指定缩进用的空白字符串，用于美化输出(pretty-print); 
  + 如果参数是个数字，它代表有多少的空格；上限为10。该值若小于1，则意味着没有空格；
  + 如果该参数为字符串（当字符串长度超过10个字母，取其前10个字母），该字符串将被作为空格；
  + 如果该参数没有提供（或者为 null），将没有空格。

`返回值` ：一个表示给定值的JSON字符串。

#### 描述

* 转换值如果有 toJSON() 方法，该方法定义什么值将被序列化。
* 非数组对象的属性不能保证以特定的顺序出现在序列化后的字符串中。
* 布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。
* undefined、任意的函数以及 symbol 值，在序列化过程中会被忽略（出现在非数组对象的属性值中时）或者被转换成 null（出现在数组中时）。函数、undefined 被单独转换时，会返回 undefined，如JSON. stringify(function(){}) or JSON. stringify(undefined). 
* 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。
* 所有以 symbol 为属性键的属性都会被完全忽略掉，即便 replacer 参数中强制指定包含了它们。
* Date 日期调用了 toJSON() 将其转换为了 string 字符串（同Date. toISOString()），因此会被当做字符串处理。
* NaN 和 Infinity 格式的数值及 null 都会被当做 null。
* 其他类型的对象，包括 Map/Set/WeakMap/WeakSet，仅会序列化可枚举的属性。

`replacer参数` 

replacer 参数可以是一个函数或者一个数组。
如果 replacer 是一个数组，数组的值代表将被序列化成 JSON 字符串的属性名。

``` js
JSON.stringify(foo, ['week', 'month']);
// '{"week":45,"month":7}', 只保留 “week” 和 “month” 属性值。
```

作为函数，它有两个参数，键（key）和值（value），它们都会被序列化。

``` js
function replacer(key, value) {
    if (typeof value === "string") {
        return undefined;
    }
    return value;
}
var foo = {
    foundation: "Mozilla",
    model: "box",
    week: 45,
    transport: "car",
    month: 7
};
var jsonString = JSON.stringify(foo, replacer);
// {"week":45,"month":7}
```

`toJSON方法` 

如果一个被序列化的对象拥有 toJSON 方法，那么该 toJSON 方法就会覆盖该对象默认的序列化行为：不是该对象被序列化，而是调用 toJSON 方法后的返回值会被序列化。

``` js
let obj = {
    title: 'lhq',
    author: [
        'arthur Y'
    ],
    edi: 3,
    year: 2019,
    child: {
        a: 'Joke',
        b: 'Luther'
    },
    toJSON: function() {
        return 'bar';
    }
};
let jsonObj = JSON.stringify(obj, (key, value) => {
    switch (key) {
        case 'author':
            return value.join(',');
        case 'year':
            return 5000;
        case 'edi':
            return undefined;
        default:
            return value;
    }
});
console.log(jsonObj); // 写了一堆，最后只会输出"bar"
```

### `JSON.parse()` 

用来解析JSON字符串，构造由字符串描述的JavaScript值或对象。提供可选的 reviver 函数用以在返回之前对所得到的对象执行变换(操作)。

#### 语法

``` js
JSON.parse(text[, reviver])
```

#### 参数

text: 要被解析成 JavaScript 值的字符串。

reviver(可选): 转换器, 如果传入该参数(函数)，可以用来修改解析生成的原始值，调用时机在 parse 函数返回之前。

#### 返回值

Object 类型, 对应给定 JSON 文本的对象/值。

#### 异常

若传入的字符串不符合 JSON 规范，则会抛出 SyntaxError 异常。

#### 使用 `reviver` 函数

如果指定了 reviver 函数，则解析出的 JavaScript 值（解析值）会经过一次转换后才将被最终返回（返回值）。



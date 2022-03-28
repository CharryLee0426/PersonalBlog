# React with TypeScript

## 1. 使用 TypeScript 的优势
TypeScript 在我的认识中并不能称得上是一门语言，它更像是一个语法检查器，通过规范 type 来减少 runtime error。typescript 也许会增大代码量，但是可以使代码更加健壮。
## 2. 在 React 中使用 TypeScript
在 React 中使用 TypeScript 可以规范代码，但是 TS 和 JS 在定义 React 组件的方式略有不同。

在 JS 中定义函数式组件：
```javascript
const Star = ({selected}) => (
    <FaStar color={selected ? 'red' : 'black'} />
);

export Star;

// usage:
// <Star selected={...} />
```

但在 TS 中，要如此定义一个函数式组件：
```typescript
export interface StarProp {
    selected: boolean,
    onSelect: any
};

const Star: React.FunctionComponent<StarProp> = (props: StarProp) => {
    return (
        <FaStar color={props.selected ? 'red' : 'grey'} onClick={props.onSelect} />
    );
}

export Star;

// usage:
// <Star selected={...} onSelect={...} />
```

相比 JS，TS 需要做出以下调整：
* 首先要定义一个该组件 props 的 type 或 interface；
* 定义该组件类型为 `React.FunctionComponent<Prop>`；
* 用类似 `html` 的语法时，其标签属性为 `Prop` type 或 interface 的属性。

## 3. TypeScript 在 React 中使用 useState 钩子进行状态管理
 
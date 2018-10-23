# angular-content-projection

## Angular 内容投影

### `Transclusion`

> `Transclusion`(嵌入包含)原本是AngularJs(1.x)中的概念，随着Angular2+的推出，`Transclusion`这一单词可能已经消失了，但是其根本概念并没有丢失

> 从本质上来说，`Transclusion`在Angular中就是获取node节点或者html的内容并将其插入模板或者一个特定的切入点中

> 该概念现在通过Web API(如Shadow DOM)在Angular中完成，称为`Content-Projection`

### AngularJS `Transclusion`

> `transclusion`在AngularJS中的实现类似于`.directive`API

> 在AngularJS中可以指定一个插槽进行内容转换


#### 单槽`transclusion`

```
function myComponent() {
  scope: {},
  transclude: true,
  template: `
    <div class="my-component">
      <div ng-transclude></div>
    </div>
  `
};
angular
  .module('app')
  .directive('myComponent', myComponent);
```

> 使用`myComponent`指令

```html
<my-component>
  This is my transcluded content!
</my-component>
```

> 编译后的HTML文件为

```html
<my-component>
  <div class="my-component">
    <div ng-transclude>
      This is my transcluded content!
    </div>
  </div>
</my-component>
```

#### 多槽`transclusion`

> 在AngularJS1.5+后可以使用`Object`定义多个`entry point`

```angularjs
function myComponent() {
  scope: {},
  transclude: {
    slotOne: 'p',
    slotTwo: 'div'
  },
  template: `
    <div class="my-component">
      <div ng-transclude="slotOne"></div>
      <div ng-transclude="slotTwo"></div>
    </div>
  `
};
angular
  .module('app')
  .directive('myComponent', myComponent);
```

> `myComponent`指令将上例中的`p`和`div`标记与相关的插槽相匹配

```html
<my-component>
  <p>
    This is my transcluded content!
  </p>
  <div>
    Further content
  </div>
</my-component>
```

> 导出后的Dom节点树为

```html
<my-component>
  <div class="my-component">
    <div ng-transclude="slotOne">
      <p>
        This is my transcluded content!
      </p>
    </div>
    <div ng-transclude="slotTwo">
      <div>
        Further content
      </div>
    </div>
  </div>
</my-component>
```

### Angular content Projection

> 将`transclusion`的概念移植到Angular2+中就成为了`content projection`

#### 单槽内容投影

> 移植之后的投影功能实现更加简洁和明了

```typescript
// my-component.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-my-component',
  template: `
    <div class="my-component">
      <ng-content></ng-content>
    </div>
  `
})
export class MyComponent {}
```

```typescript
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
   <div class="app">
     <app-my-component>
       投影内容
     </app-my-component>
   </div>
  `
})
export class AppComponent {}
```

> 生成的DOM树为

```html
<div _ngcontent-c0="" class="app">
  <app-my-component _ngcontent-c0="" _nghost-c1="">
    <div _ngcontent-c1="" class="my-component">
      投影内容
     </div>
  </app-my-component>
</div>
```

### 多槽内容投影

> 与AngularJS中使用`transclude: {}`属性识别DOM引用不同，Angular2+使用更加直接的方法与DOM节点沟通

```typescript
// 
import { Component } from '@angular/core';
@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      <app-my-component>
        <my-component-title>
          组件标题
        </my-component-title>
        <my-component-content>
          组件内容
        </my-component-content>
      </app-my-component>
    </div>
  `
})
```

> 这里假设存在my-component-title和my-component-content作为自定义组件
> 用以获取对组件的引用，并告诉Angular在适当的位置注入
> 在引用时需要明确`<ng-content>`标签的`select=''`属性

```typescript
// my-component.component.ts
import { Component } from '@angular/core';
@Component({
  selector: 'app-my-component',
  template: `
    <div class="my-component">
      <div>
        Title:
        <ng-content select="my-component-title"></ng-content>
      </div>
      <div>
        Content:
        <ng-content select="my-component-content"></ng-content>
      </div>
    </div>
  `
})
```

> 事实上，在声明要投影的内容时不必像上述那样使用自定义元素
> 可以使用常规元素并以类似于`document.querySelector`的对话方式来定位投影元素

```typescript
// app.component.ts
import { Component } from '@angular/core';
@Component({
  selector: 'app-root',
  template: `
   <div class="app">
     <app-my-component2>
       <div class="my-component-title">
         组件标题
       </div>
       <div class="my-component-content">
         组件内容
       </div>
     </app-my-component2>
   </div>
  `
})
```

```typescript
// my-component.component2.ts
import { Component } from '@angular/core';
@Component({
  selector: 'app-root',
  template: `
   <div class="my-component">
     <div>
       Title:
       <ng-content select=".my-component-title"></ng-content>
     </div>
     <div>
       Content:
       <ng-content select=".my-component-content"></ng-content>
     </div>
   </div>
  `
})
```

> 生成的DOM树为

```html
<div _ngcontent-c0="" class="app">
  <app-my-component2 _ngcontent-c0="" _nghost-c2="">
    <div _ngcontent-c2="" class="my-component">
      <div _ngcontent-c2=""> 
        Title: 
          <div _ngcontent-c0="" class="my-component-title"> 
             组件标题 
          </div>
       </div>
       <div _ngcontent-c2=""> 
         Content: 
           <div _ngcontent-c0="" class="my-component-content">
              组件内容 
           </div>
       </div>
    </div>
  </app-my-component2>
</div>
```

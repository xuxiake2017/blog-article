## TypeScript内置类型一览

#### 前文

当下而言，基本TS开发项目可以说是大势所趋，用过TS开发项目的同学都基本表示再也回不去啦，今天就让我们来了解一下TypeScript官方的内置类型，让你的开发效率再上一层楼

#### Partial（部分的）

```ts
/**
 * Make all properties in T optional
 */
type Partial<T> = {
    [P in keyof T]?: T[P];
};
```

作用是让传入类型中的所有属性变成都是可选的

>  使用举例

```ts
export interface Student {
  name: string;
  age: number;
}

const student1: Student = {}

const student2: Partial<Student> = {}
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/Partial.JPG)

变量student1的类型是Student，Student默认所有的属性都是不能为空的，所有会报错，student2就不会

#### Required（必须的）

```ts
/**
 * Make all properties in T required
 */
type Required<T> = {
    [P in keyof T]-?: T[P];
};
```

跟Partial的作用是相反的，是让传入类型中的所有属性变成都是必填的

>  使用举例

```ts
export interface Student {
  name?: string;
  age?: number;
}

const student1: Student = {}

const student2: Required<Student> = {}
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/Required.JPG)

变量student1的类型是Student，Student默认所有的属性都是可以为空的，所有不会报错，student2会报错

#### Readonly（只读的）

```ts
/**
 * Make all properties in T readonly
 */
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
};
```

作用是让传入类型中的所有属性变成都是只读的（不能修改属性）

>  使用举例

```ts
export interface Student {
  name: string;
  age: number;
}

const student1: Student = {
  name: '张三',
  age: 20
}
student1.age = 21

const student2: Readonly<Student> = {
  name: '李四',
  age: 20
}
student2.age = 21
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/Readonly.JPG)

给student1的属性age重新赋值不会报错，给student2的属性age重新赋值就会报错，因为student2所有的属性都是只读的

#### Pick（选择）

```ts
/**
 * From T, pick a set of properties whose keys are in the union K
 */
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```

作用是选择传入类型中的部分属性组成新类型

>  使用举例

```ts
export interface Student {
  name: string;
  age: number;
}

const student1: Student = {
  name: '张三',
  age: 20
}

const student2: Pick<Student, 'name'> = {
  name: '李四'
}

const student3: Pick<Student, 'name'> = {
  name: '王五',
  age: 20
}
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/Pick.JPG)

变量student1可以有所有属性name和age，变量student2就只能有属性name，变量student3加上属性age就会报错

#### Record（记录）

```ts
/**
 * Construct a type with a set of properties K of type T
 */
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

作用是构建一个类型，这个类型用来描述一个对象，这个对象的属性都具有相同的类型

>  使用举例

```ts
export const student1: Record<string, any> = {
  name: '张三',
  age: 20
}
```

Record应该是日常使用频率较高的内置类型了，主要用来描述对象，一般建议是不用Object来描述对象，而是用Record代替，Record<string, any>几乎可以说是万金油了

#### Exclude（排除）

```ts
/**
 * Exclude from T those types that are assignable to U
 */
type Exclude<T, U> = T extends U ? never : T;
```

针对联合类型（interface这种没用），用人话说，排除相同的，留下不同的

>  使用举例

```ts
export type PersonAttr = 'name' | 'age'

export type StudentAttr = 'name' | 'age' | 'class' | 'school'

const student1: Exclude<StudentAttr, PersonAttr>
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/Exclude.JPG)

student1就只能被赋值为'class' 或者'school'

#### Extract（取出）

```ts
/**
 * Extract from T those types that are assignable to U
 */
type Extract<T, U> = T extends U ? T : never;
```

与Exclude相反，针对联合类型，排除不同的的，取出相同的

>  使用举例

```ts
export type PersonAttr = 'name' | 'age'

export type StudentAttr = 'name' | 'age' | 'class' | 'school'

const student1: Extract<StudentAttr, PersonAttr>
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/Extract.JPG)

student1就只能被赋值为'name'或者'age'

#### Omit（省略）

```ts
/**
 * Construct a type with the properties of T except for those in type K.
 */
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

传入一个类型，和这个类型的几个属性，把传入的属性省略掉，组成一个新类型

>  使用举例

```ts
export interface Student {
  name: string;
  age: number;
  class: string;
  school: string;
}

export type PersonAttr = 'name' | 'age'

export type StudentAttr = 'name' | 'age' | 'class' | 'school'

const student1: Omit<Student, PersonAttr> = {}
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/Omit.JPG)

student1报错，提示没有属性'name'、'age'

#### NonNullable（不能为null）

```ts
/**
 * Exclude null and undefined from T
 */
type NonNullable<T> = T extends null | undefined ? never : T;
```

字面意思，不能为空

>  使用举例

```ts
export interface Student {
  name: string;
  age: number;
}

const student1: NonNullable<Student | undefined | null> = null
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/NonNullable.JPG)

student1赋值为null会报错（在tsconfig.json配置文件中开启类型检查，`"skipLibCheck": false`）

#### Parameters（参数）

```ts
/**
 * Obtain the parameters of a function type in a tuple
 */
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;
```

获取传入函数的参数组成的类型

>  使用举例

```ts
export interface Student {
  name: string;
  age: number;
}

export interface StudentFunc {
  (name: string, age: number): Student
}

const student1: Parameters<StudentFunc>
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/Parameters.JPG)

student1的类型为`[name: string, age: number]`

#### ConstructorParameters（构造参数）

```ts
/**
 * Obtain the parameters of a constructor function type in a tuple
 */
type ConstructorParameters<T extends abstract new (...args: any) => any> = T extends abstract new (...args: infer P) => any ? P : never;
```

获取传入构造函数的参数组成的类型

>  使用举例

```ts
export interface Student {
  name: string;
  age: number;
}

export interface StudentConstructor {
  new (name: string, age: number): Student
}

const student1: ConstructorParameters<StudentConstructor>
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/ConstructorParameters.JPG)

student1的类型为`[name: string, age: number]`

#### ReturnType（返回类型）

```ts
/**
 * Obtain the return type of a function type
 */
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

获取传入函数的返回类型

>  使用举例

```ts
export interface Student {
  name: string;
  age: number;
}

export interface StudentFunc {
  (name: string, age: number): Student
}

const student1: ReturnType<StudentFunc> = {}
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/ReturnType.JPG)

student1的类型为`Student`

#### InstanceType（构造返回类型、实例类型）

```ts
/**
 * Obtain the return type of a constructor function type
 */
type InstanceType<T extends abstract new (...args: any) => any> = T extends abstract new (...args: any) => infer R ? R : any;
```

获取传入构造函数的返回类型

>  使用举例

```ts
const Student = class {
  name: string;
  age: number;
  constructor (name: string, age: number) {
    this.name = name
    this.age = age
  }
  showInfo () {
    console.log('name: ', this.name, 'age: ', this.age);
  }
}

const student1: InstanceType<typeof Student> = new Student('张三', 20)

```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/InstanceType.JPG)

个人认为这是一个非常好用的内置类型，目前在前端项目中，class是用的越来越多了，在TS中，class其实也是可以用作类型声明空间的，用来描述对象类型，但是一般来说好像很少这样用的，一般用interface或者type居多
```ts
export class Student {
  name: string;
  age: number;
}
```
所以一般就是直接把class用作变量声明空间，但是对于 class new 出的实例，怎么描述它的类型呢，就如上文的，直接`const student1: Student`那是铁定会报错的，因为Student用作变量声明空间，没有用作类型声明空间（听起来好绕），这时候就可以用到InstanceType，完美解决问题

#### Uppercase（大写）

```ts
/**
 * Convert string literal type to uppercase
 */
type Uppercase<S extends string> = intrinsic;
```

变大写

>  使用举例

```ts
export type StudentSexType = 'male' | 'female'

const studentSex: Uppercase<StudentSexType> = 'MALE'
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/Uppercase.JPG)


#### Lowercase（小写）

```ts
/**
 * Convert string literal type to lowercase
 */
type Lowercase<S extends string> = intrinsic;
```

变小写

>  使用举例

```ts
export type StudentSexType = 'MALE' | 'FEMALE'

const studentSex: Lowercase<StudentSexType> = ''
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/Lowercase.JPG)

#### Capitalize（首字母大写）

```ts
/**
 * Convert first character of string literal type to uppercase
 */
type Capitalize<S extends string> = intrinsic;
```

首字母变大写

>  使用举例

```ts
export type StudentSexType = 'male' | 'female'

const studentSex: Capitalize<StudentSexType> = ''
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/Capitalize.JPG)

#### Uncapitalize（首字母小写）

```ts
/**
 * Convert first character of string literal type to lowercase
 */
type Uncapitalize<S extends string> = intrinsic;
```

首字母变小写

>  使用举例

```ts
export type StudentSexType = 'MALE' | 'FEMALE'

const studentSex: Uncapitalize<StudentSexType> = ''
```

![](https://read-1252195440.cos.ap-guangzhou.myqcloud.com/TypeScript内置类型一览/Uncapitalize.JPG)
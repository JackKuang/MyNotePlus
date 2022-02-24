# 一、概述

IF判断语句是编程语言的重要组成部分，我们用IF开处理不同的条件下的不同数据处理。
但是一旦我们大量的使用了IF语句，就会让我们的代码更加复杂且难以维护。下面的场景，我们会做几个案例，来代替IF语法。

# 二、案例

我们使用一个计算器案例来处理，计算器可以根据操作符和数字来计算数据，我们可以根据不同的逻辑来判断处理，比如：

```java
public class CaseStudy1 {
  
    public static void main(String[] args) {
        System.out.println(calculate(1, 2, "add"));
    }

    public static Integer calculate(int a, int b, String operator) {
        if ("add".equals(operator)) {
            return a + b;
        } else if ("substract".equals(operator)) {
            return a - b;
        }
        return null;
    }
}

```

当然，我们也swtich来实现

```java
public class CaseStudy2 {

    public static void main(String[] args) {
        System.out.println(calculate(1, 2, "add"));
    }

    public static Integer calculate(int a, int b, String operator) {
        switch (operator) {
            case "add":
                return a + b;
            case "substract":
                return a + b;
            default:
                return null;
        }
    }
}
```

# 三、重构

针对上面这个案例，我们针对重构一下代码。

## 3.1 工厂模式

```java

public class CaseStudyFactory {

    public static void main(String[] args) {
        System.out.println(calculateWithFactory(1, 2, "add"));
    }

    private static int calculateWithFactory(int a, int b, String operator) {
        Operation operation = OperationFactory.getOperation(operator)
                .orElseThrow(() -> new IllegalArgumentException("not support"));
        return operation.apply(a, b);
    }

}

interface Operation {
    int apply(int a, int b);
}

class Add implements Operation {
    @Override
    public int apply(int a, int b) {
        return a + b;
    }
}

class Subtract implements Operation {
    @Override
    public int apply(int a, int b) {
        return a - b;
    }
}

class OperationFactory {

    static Map<String, Operation> operationMap = new HashMap();

    static {
        operationMap.put("add", new Add());
        operationMap.put("subtract", new Subtract());
    }

    public static Optional<Operation> getOperation(String Operator) {
        return Optional.ofNullable(operationMap.get(Operator));
    }
}
```

在这个案例中，我们看到了把判断的逻辑委托给工厂，由工厂来决定实现，有一定程度的解藕。

## 3.2 使用枚举

除了使用Map之外，我们可以使用枚举替换字符串来标记业务逻辑。

```java
enum OperationEnums {
    add ,
    subtract;
}
```

就如我们需要实现的一样，一种枚举值对应来一种计算规则，那么，我们也可以把计算逻辑赋予枚举值中。

```java

public class CaseStudyEnum {

    public static void main(String[] args) {
        System.out.println(calculateWithFactory(1, 2, "add"));
    }

    private static int calculateWithFactory(int a, int b, String operator) {
        return OperationEnums.valueOf(operator).apply(a, b);
    }

}

enum OperationEnums {
    add {
        @Override
        public int apply(int a, int b) {
            return a + b;
        }
    },
    subtract {
        @Override
        public int apply(int a, int b) {
            return a - b;
        }
    };

    abstract int apply(int a, int b);
}

```

在这个案例中，我们看到了把判断的逻辑委托给不同的枚举属性，以此减少了逻辑判断。

## 3.3
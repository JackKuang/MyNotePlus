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

## 3.3 命令行模式

```java

public class CaseStudyCommand {

    public static void main(String[] args) {
        System.out.println(calculateWithCommand(new AddCommand(1, 2)));
    }

    private static int calculateWithCommand(Command command) {
        return command.execute();
    }

}

interface Command {
    int execute();
}

class AddCommand implements Command {

    private int a;
    private int b;

    public AddCommand(int a, int b) {
        this.a = a;
        this.b = b;
    }

    @Override
    public int execute() {
        return a + b;
    }
}

class SubtractCommand implements Command {

    private int a;
    private int b;

    public SubtractCommand(int a, int b) {
        this.a = a;
        this.b = b;
    }

    @Override
    public int execute() {
        return a - b;
    }
}
```

前面的例子里，我们可以看到使用了国内工厂模式根据不同的操作来处理不同的业务逻辑对象，由业务逻辑对象来计算。同样也可以吧计算逻辑封装到一起。这也是一种替换if语句的方式。

## 3.3 规则引擎

当我们每个if条件下都需要写庞大的业务代码时，需要进行特定的判断之后才能执行业务逻辑。
通过规则引擎可以在主核心代码中消除业务中判断的复杂性，主核心代码只要判断规则并根据输入返回结果。

```java
t java.util.*;

public class CaseStudyRule {

    public static void main(String[] args) {
        System.out.println(calculateWithRule(1, 2, RuleOperator.add));
    }

    private static int calculateWithRule(int a, int b, RuleOperator operator) {
        return RuleEngine.process(new Expression(a, b, operator));
    }

}

class Expression {
    Integer x;
    Integer y;
    RuleOperator operator;

    public Expression(Integer x, Integer y, RuleOperator operator) {
        this.x = x;
        this.y = y;
        this.operator = operator;
    }
}

enum RuleOperator {
    add,
    subtract;
}

interface Rule {
    boolean evaluate(Expression expression);

    int getResult();
}

class AddRule implements Rule {

    private int result;

    @Override
    public boolean evaluate(Expression expression) {
        if (expression.operator == RuleOperator.add) {
            this.result = expression.x + expression.y;
            return true;
        }
        return false;
    }

    @Override
    public int getResult() {
        return this.result;
    }
}

class RuleEngine {
    private static List<Rule> rules = new ArrayList<>();

    static {
        rules.add(new AddRule());
    }

    public static int process(Expression expression) {
        Rule rule = rules.stream()
                .filter(item -> item.evaluate(expression))
                .findFirst()
                .orElseThrow(() -> new IllegalArgumentException("not suppport"));
        return rule.getResult();
    }

}
```

# 四、结论

减少if语法可以大大降低软件复杂度、提高可维护性。

建议使用**枚举方法**实现。
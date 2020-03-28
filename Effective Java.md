# 第1条：用静态工厂方法代替构造器

`静态工厂方法与构造器不同的第一大优势在于，它们有名称。`如果构造器的参数本身没有确切地描述正被返回的对象，那么具有适当名称的静态工厂会更容易使用，产生的客户端代码也更易于阅读。例如，构造器BigInteger(int, int, Random)返回BigInteger可能为素数，如果用名为BigInteger.probablePrime的静态工厂方法来表示，显然更为清楚。（Java 4增加了这个方法）

一个类只能有一个带有指定签名的构造器。编程人员通常知道如何避开这一限制：通过提供两个构造器，它们的参数列表只在参数类型的顺序上有所不同。实际上这并不是个好主意。面对这样的API，用户永远也记不住该用哪个构造器，结果常常会调用错误的构造器。并且在读到使用了这些构造器的代码时，如果没有参考类文档，往往不知所云。

由于静态工厂方法有名称，所以它们不受上述限制。当一个类需要多个带有相同签名的构造器时，就用静态工厂方法代替构造器，并且仔细地选择名称以便突出静态工厂方法之间的区别。

`静态工厂方法与构造器不同的第二大优势在于，不必在每次调用它们的时候都创建一个新对象。`这使得不可变类可以使用预先构建好的实例，或者将构建好的实例缓存起来，进行重复利用，从而避免创建不必要的重复对象。如果程序经常请求创建相同的对象，并且创建对象的代价很高，则这项技术可以极大地提升性能。

静态工厂方法能够为重复的调用返回相同对象，这样有助于类总能严格控制在某个时刻哪些实例应该存在。这种类被称作实例受控（instance-controlled）的类。实例受控使得类可以确保它是一个Singleton或者是不可实例化的。它还使得不可变的类可以确保不会存在两个相等的实例，即当且仅当a == b时，a.equals(b)才为true。这是享元模式的基础。枚举类型保证了这一点。

`静态工厂方法与构造器不同的第三大优势在于，它们可以返回原返回类型的任何子类型的对象。`这样我们在选择返回对象的类时就有了更大的灵活性。

这种灵活性的一种应用是，API可以返回对象，同时又不会使对象的类变成公有的。以这种方式隐藏实现类会使API变得非常简洁。这项技术适用于基于接口的框架，因为在这种框架中，接口为静态工厂方法提供了自然返回类型。

在Java 8之前，接口不能有静态方法，因此按照惯例，接口Type的静态工厂方法被放在一个名为Types的不可实例化的伴生类中。例如，Java Collections Framework的集合接口有45个工具实现，分别提供了不可修改的集合、同步集合等。几乎所有这些实现都通过静态工厂方法在一个不可实例化的类（java.util.Collections）中导出。所有返回对象的类都是非公有的。

现在的Collections Framework API比导出45个独立公有类的那种实现方式要小得多，每种便利实现都对应一个类。这不仅仅是指API数量上的减少，也是概念意义上的减少：为了使用这个API，用户必须掌握的概念在数量和难度上都减少了。程序员知道，被返回的对象是由相关的接口精确指定的，所以他们不需要阅读有关的类文档。此外，使用这种静态工厂方法时，甚至要求客户端通过接口来引用被返回的对象，而不是通过它的实现类来引用被返回的对象，这是一种良好的习惯。

在Java 8开始，接口中不能包含静态方法的这一限制成为历史，因此一般没有任何理由给接口提供一个不可实例化的伴生类。已经被放在这种类中的许多公有的静态成员，应该被放到接口中去。但是要注意，仍然有必要将这些静态方法背后的大部分实现代码，单独放进一个包级私有的类中。这是因为在Java 8中仍要求接口的所有静态成员都必须是公有的。在Java 9中允许接口有私有的静态方法，但是静态域和静态成员仍然需要时公有的。

`静态工厂的第四大优势在于，所返回的对象的类可以随着每次调用而发生变化，这取决于静态工厂方法的参数值。`只要是已声明的返回类型的子类型，都是允许的。返回对象的类也可能随着发行版本的不同而不同。

EnumSet没有公有的构造器，只有静态工厂方法。在OpenJDK实现中，它们返回两种子类之一的一个实例，具体取决于底层枚举类型的大小：如果它的元素有64个或者更少，就像大多数枚举类型一样，静态工厂方法就会返回一个RegularEnumSet实例，用单个long进行支持；如果枚举类型有65个或者更多元素，工厂就返回JumboEnumSet实例，用一个long数组进行支持。

这两个实现类的存在对于客户端来说是不可见的。如果RegularEnumSet不能再给小的枚举类型提供性能优势，就可能从未来的发行版本中将它删除，不会造成任何负面的影响。同样地，如果事实证明对性能有好处，也可能在未来的发行版本中添加第三甚至第四个EnumSet实现。客户端永远不知道也不关心它们从工厂方法中得到的对象的类，它们只关心它是EnumSet的某个子类。

`静态工厂的第五大优势在于，方法返回的对象所属的类，在编写包含该静态工厂方法的类时可以不存在。`这种灵活的静态工厂方法构成了服务提供者框架（Service Provider Framework）的基础，例如JDBC API。服务提供者框架是指这样一个系统：多个服务提供者实现一个服务，系统为服务提供者的客户端提供多个实现，并把它们从多个实现中解耦出来。

服务提供者框架中有三个重要的组件：服务接口（Service Interface），这是提供者实现的；提供者注册API（Provider Registration API），这是提供者用来注册实现的；服务访问API（Service Access API），这是客户端用来获取服务的实例。服务访问API是客户端用来指定某种选择实现的条件。如果没有这样的规定，API就会返回默认实现的一个实例，或者允许客户端遍历所有可用的实现。服务访问API是“灵活的静态工厂”，它构成了服务提供者框架的基础。

服务提供者框架的第四个组件是服务提供者接口（Service Provider Interface）是可选的，它表示产生服务接口之实例的工厂对象。如果没有服务提供者接口，实现就通过反射方式进行实例化。对于JDBC来说，Connection就是其服务接口的一部分，DriverManager.registerDriver是提供者注册API，DriverManager.getConnection是服务访问API，Driver是服务提供者接口。

服务提供者框架模式有着无数种变体。例如，服务访问API可以返回比提供者需要的更丰富的服务提供。这就是桥接模式。依赖注入框架可以被看作是一个强大的服务提供者。从Java 6开始，Java平台就提供了一个通用的服务提供者框架java.util.ServiceLoader，因此你不需要（一般来说也不应该）再自己编写了。JDBC不用ServiceLoader，因为前者出现得比后者早。

`静态工厂方法的主要缺点在于，类如果不含公有的或者受保护的构造器，就不能被子类化。`例如，要想将Collections Framework中的任何便利的实现类子类化，这是不可能的。但是这样也许会因祸得福，因为它鼓励程序员使用复合，而不是继承，这正是不可变类型所需要的。

`静态工厂方法的第二个缺点在于，程序员很难发现它们。`在API文档中，它们没有像构造器那样在API文档中明确标识出来，因此，对于提供了静态工厂方法而不是构造器的类来说，要想查明如何实例化一个类是非常困难的。Javadoc工具总有一天会注意到静态工厂方法。同时，通过在类或者接口注释中关注静态工厂，并遵守标准的命名习惯，也可以弥补这一劣势。
# 第2条：遇到多个构造器参数时要考虑使用Builder模式

静态工厂和构造器有个共同的局限性：它们都不能很好地扩展到大量的可选参数。

重叠构造器模式：

```java
public class NutritionFacts {
    private final int servingSize;

    private final int servings;

    private final int calories;

    private final int fat;

    private final int sodium;

    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0)
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

JavaBeans模式：

```java
public class NutritionFacts {
    private int servingSize = -1;

    private int servings = -1;

    private int calories = 0;

    private int fat = 0;

    private int sodium = 0;

    private int carbohydrate = 0;

    public NutritionFacts() {}

    public void setServingSize(int val) {
        servingSize = val;
    }

    public void setServings(int val) {
        servings = val;
    }

    public void setCalories(int val) {
        calories = val;
    }

    public void setFat(int val) {
        fat = val;
    }

    public void setSodium(int val) {
        sodium = val;
    }

    public void setCarbohydrate(int val) {
        carbohydrate = val;
    }
}
```

在JavaBeans模式中，构造过程被分到了几个调用中，`在构造过程中JavaBean可能处于不一致的状态`。类无法仅仅通过检验构造器参数的有效性来保证一致性。试图使用处于不一致状态的对象将会导致失败，这种失败与包含错误的代码大相径庭，因此调试起来十分困难。与此相关的另一点不足在于，`JavaBeans模式使得把类做成不可变的可能性不复存在`。

当对象的构造完成，并且不允许在冻结之前使用时，通过手工“冻结”对象可以弥补这些不足，但是这种方式十分笨拙，在实践中很少使用。此外，它甚至会在运行时导致错误，因为编译器无法确保程序员会在使用之前先调用对象上的freeze方法进行冻结。

Builder模式：

```java
public class NutritionFacts {
    private final int servingSize;

    private final int servings;

    private final int calories;

    private final int fat;

    private final int sodium;

    private final int carbohydrate;

    public static class Builder {
        private int servingSize = -1;

        private int servings = -1;

        private int calories = 0;

        private int fat = 0;

        private int sodium = 0;

        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

NutritionFacts是不可变的，所有的默认参数值都单独放在一个地方。builder的设值方法返回builder本身，以便把调用链接起来，得到一个`流式`的API：

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();
```

这样的客户端代码很容易编写，更为重要的是易于阅读。Builder模式模拟了具名的可选参数。

`Builder模式也适用于类层次结构。`

```java
public abstract class Pizza {
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}

    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
```

Pizza.Builder的类型是泛型，带有一个递归类型参数。它和抽象的self方法一样，允许在子类中适当地进行方法链接，不需要转换类型。

```java
public class NyPizza extends Pizza {
    public enum Size {SMALL, MEDIUM, LARGE}

    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        public NyPizza build() {
            return new NyPizza(this);
        }

        protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}

public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false;

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        public Calzone build() {
            return new Calzone(this);
        }

        protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```

每个子类的构建器中的build方法，都声明返回正确的子类：NyPizza.Builder的build方法返回NyPizza，而Calzone.Builder中的则返回Calzone。在该方法中，子类方法声明返回超类中声明的返回类型的子类型，这被称作`协变返回类型`。它允许客户端无须转换类型就能使用这些构建器。

```java
NyPizza pizza = new NyPizza.Builder(SMALL).addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder().addTopping(HAM).sauceInside().build();
```

与构造器相比，builder的微略优势在于，它可以有多个可变参数。因为builder是利用单独的方法来设置每一个参数。此外，构造器还可以将多次调用某一个方法而传入的参数集中到一个域中，如前面的调用了两次addTopping方法的代码所示。

Builder模式的确也有它自身的不足。为了创建对象，必须先创建它的构建器。虽然创建这个构建器的开销在实践中可能不那么明显，但是在某些十分注重性能的情况下，可能就成问题了。Builder模式还比重叠构造器模式更加冗长，因此它只在有很多参数的时候才使用，比如4个或者更多个参数。但是记住，将来你可能需要添加参数。如果一开始就使用构造器或者静态工厂，等到类需要多个参数时才添加构建器，就会无法控制，那些过时的构造器或者静态工厂显得十分不协调。因此，通常最好一开始就使用构建器。

`如果类的构造器或者静态工厂中具有多个参数，设计这种类时，Builder模式就是一种不错的选择`，特别是当大多数参数都是可选或者类型相同的时候。与使用重叠构造器模式相比，使用Builder模式的客户端代码将更易于阅读和编写，构建器也比JavaBeans更加安全。

# 第3条：用私有构造器或者枚举类型强化Singleton属性

实现Singleton有两种常见的方法。这两种方法都要保持构造器为私有的，并导出公有的静态成员，以便允许客户端能够访问该类的唯一实例。在第一种方法中，公有静态成员是个final域：

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {}

    public void leaveTheBuilding() {}
}
```

私有构造器仅被调用一次，用来实例化公有的静态final域Elvis.INSTANCE。由于缺少公有的或者受保护的构造器，所以保证了Elvis的全局唯一性：一旦Elvis类被实例化，将只会存在一个Elvis实例，不多也不少。客户端的任何行为都不会改变这一点，但要提醒一点：享有特权的客户端可以借助AccessibleObject.setAccessible方法，通过反射机制调用私有构造器。如果需要抵御这种攻击，可以修改构造器，让它在被要求创建第二个实例的时候抛出异常。

在实现Singleton的第二种方法中，公有的成员是个静态工厂方法：

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {}

    public static Elvis getInstance() {
        return INSTANCE;
    }

    public void leaveTheBuilding() {}
}
```

对于静态方法Elvis.getInstance的所有调用，都会返回同一个对象引用，所以，永远不会创建其他的Elvis实例（上述提醒依然适用）。

公有域方法的主要优势在于，API很清楚地表明了这个类是一个Singleton：公有的静态域是final的，所以该域总是包含相同的对象引用。第二个优势在于它更简单。

静态工厂方法的优势之一在于，它提供了灵活性：在不改变其API的前提下，我们可以改变该类是否应该为Singleton的想法。工厂方法返回该类的唯一实例，但是，它很容易被修改，比如改成为每个调用该方法的线程返回一个唯一的实例。第二个优势是，如果应用程序需要，可以编写一个泛型Singleton工厂。使用静态工厂的最后一个优势是，可以通过方法引用作为提供者，比如Elvis::instance就是一个Supplier\<Elvis\>。

为了将利用上述方法实现的Singleton类变成是可序列化的，仅仅在声明中加上implements Serializable是不够的。为了维护并保证Singleton，必须声明所有实例域都是transient，并提供一个readResolve方法。否则，每次反序列化一个序列化的实例时，都会创建一个新的实例。

实现Singleton的第三种方法是声明一个包含单个元素的枚举类型：

```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {}
}
```

这种方法在功能上与公有域方法相似，但更简洁，无偿地提供了序列化机制，绝对防止多次实例化，即使是在面对复杂的序列化或者反射攻击的时候。虽然这种方法还没有广泛使用，但是单元素的枚举类型经常成为实现Singleton的最佳方法。注意，如果Singleton必须扩展一个超类，而不是扩展Enum的时候，则不宜使用这个方法（虽然可以声明枚举去实现接口）。

# 第4条：通过私有构造器强化不可实例化的能力

有时可能需要编写只包含静态方法和静态域的类。这些类的名声很不好，因为有些人在面向对象的语言中滥用这样的类来编写过程化的程序，但它们也确实有特别的用处。我们可以利用这种类，以java.lang.Math或者java.util.Arrays的方式，把基本类型的值或者数组类型上的相关方法组织起来。我们也可以通过java.util.Collections的方式，把实现特定接口的对象上的静态方法，包括工厂方法组织起来。（从Java 8开始，也可以把这些方法放进接口中，假定这是你自己编写的接口可以进行修改。）最后，还可以利用这种类把final类上的方法组织起来，因为不能把它们放在子类中。

这样的工具类不希望被实例化，因为实例化对它没有任何意义。然而，在缺少显式构造器的情况下，编译器会自动提供一个公有的、无参的缺省构造器。对于用户而言，这个构造器与其他的构造器没有任何区别。在已发行的API中常常可以看到一些被无意识地实例化的类。

`企图通过将类做成抽象类来强制该类不可被实例化是行不通的。`该类可以被子类化，并且该子类也可以被实例化。这样做甚至会误导用户，以外这种类是专门为了继承而设计的。然而，有一些简单的习惯用法就可以确保类不可被实例化。由于只有当类不包含显式的构造器时，编译器才会生成缺省的构造器，因此只要让这个类包含一个私有构造器，它就不能被实例化：

```java
public class UtilityClass {
    private UtilityClass() {
        throw new AssertionError();
    }
}
```

由于显式的构造器是私有的，所以不可以再该类的外部访问它。AssertionError不是必需的，但是它可以避免不小心在类的内部调用构造器。它保证该类在任何情况下都不会被实例化。这种习惯用法也有副作用，它使得一个类不能被子类化。所有的构造器都必须显式或隐式地调用超类的构造器，在这种情形下，子类就没有可访问的超类构造器可调用了。

# 第5条：优先考虑依赖注入来引用资源

有许多类会依赖一个或多个底层的资源。例如，拼写检查器需要依赖词典。因此，像下面这样把类实现为静态工具类的做法并不少见：

```java
public class SpellChecker {
    private static final Lexicon dictionary = ...;

    private SpellChecker() {}

    public static boolean isValid(String word) { ... }

    public static List<String> suggestions(String typo) { ... }
}
```

同样地，将这些类实现为Singleton的做法也并不少见：

```java
public class SpellChecker {
    private final Lexicon dictionary = ...;

    private SpellChecker(...) {}

    public static INSTANCE = new SpellChecker(...);

    public boolean isValid(String word) { ... }

    public List<String> suggestions(String typo) { ... }
}
```

以上两种方法都不理想，因为它们都是假定只有一本词典可用，实际上，每一种语言都有自己的词典，特殊词汇还要使用特殊的词典。此外，可能还需要用特殊的词典进行测试。因此假定用一本词典就能满足所有需求，这简直是痴心妄想。

建议尝试用SpellChecker来支持多词典，即在现有的拼写检查器中，设dictionary域为nonfinal，并添加一个方法用它来修改词典，但是这样的设置会显得很笨拙、容易出错，并且无法并行工作。`静态工具类和Singleton类不适合于需要应用底层资源的类。`

这里需要的是能够支持类的多个实例（在本例中是指SpellChecker），每一个实例都使用客户端指定的资源（在本例中是指词典）。满足该需求的最简单的模式是，`当创建一个新的实例时，就将该资源传到构造器中`。这是`依赖注入`的一种形式：词典是拼写检查器的一个依赖，在创建拼写检查器时就将词典注入其中。

```java
public class SpellChecker {
    private final Lexicon dictionary;

    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) { ... }

    public List<String> suggestions(String typo) { ... }
}
```

虽然这个拼写检查器的范例中只有一个资源（词典），但是依赖注入却适用于任意数量的资源，以及任意的依赖形式。依赖注入的对象资源具有不可变性，因此多个客户端可以共享依赖对象（假设客户端们想要的是同一个底层资源）。依赖注入也同样适用于构造器、静态工厂和构建器。

这个程序模式的另一种有用的变体是，将资源工厂传给构造器。工厂是可用被重复调用来创建类型实例的一个对象。这类工厂具体表现为工厂方法模式。在Java 8中增加的接口Supplier\<T\>，最适合用于表示工厂。带有Supplier\<T\>的方法，通常应该限制输入工厂的类型参数使用有限制的通配符类型，以便客户端能够传入一个工厂，来创建指定类型的任意子类型。例如，下面是一个生产马赛克的方法，它利用客户端提供的工厂来生产每一片马赛克：

```java
Mosaic create(Supplier<? entendx Tile> tileFactory) { ... }
```

虽然依赖注入极大地提升了灵活性和可测试性，但它会导致大型项目凌乱不堪，因为它通常包含上千个依赖。不过这种凌乱用一个依赖注入框架便可以终结，如Spring。

总而言之，不要用Singleton和静态工具类来实现依赖一个或多个底层资源的类，且该资源的行为会影响到该类的行为；也不要直接用这个类来创建这些资源。而应该将这些资源或者工厂传给构造器（或者静态工厂，或者构建器），通过它们来创建类。这个世间就被称作依赖注入，它极大地提升了类的灵活性、可重用性和可测试性。

# 第6条：避免创建不必要的对象

一般来说，最好能重用单个对象，而不是在每次需要的时候就创建一个相同功能的新对象。如果对象是不可变的，它就始终可以被重用。

作为一个极端的反面例子，看看下面的语句：

```java
String s = new String("bikini");  //DON'T DO THIS!
```

该语句每次被执行的时候都创建一个新的String实例，但是这些创建对象的动作全都是不必要的。传递给String构造器的参数“bikini”本身就是一个String实例，功能方面等同于构造器创建的对象。如果这种用法是在一个循环中，或者是在一个呗频繁调用的方法中，就会创建出成千上万不必要的String实例。

改进后的版本如下所示：

```java
String s = "bikini";
```

这个版本只用了一个String实例，而不是每次执行的时候都创建一个新的实例。而且，它可以保证，对于所有在同一台虚拟机中运行的代码，只要它们包含相同的字符串字面常量，该对象就会被重用。

对于同时提供了静态工厂方法和构造器的不可变类，通常优先使用静态工厂方法而不是构造器，以避免创建不必要的对象。例如，静态工厂方法Boolean.valueOf(String)几乎总是优先于构造器Boolean(String)，构造器Boolean(String)在Java 9中已经被废弃了。构造器在每次被调用的时候都会创建一个新的对象，而静态工厂方法则从来不要求这样做，实际上也不会这样做。除了重用不可变的对象之外，也可以重用那些已知不会被修改的可变对象。

有些对象创建的成本比其他对象要高得多。如果重复地需要这类“昂贵的对象”，建议将它缓存下来重用。遗憾的是，在创建这种对象的时候，并非总是那么显而易见。假设想要编写一个方法，用它确定一个字符串是否为一个有效的罗马数字。下面介绍一种最容易的方法，使用一个正则表达式：

```java
static boolean isRomanNumeral(String s) {
    return s.matcher("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

这个实现的问题在于它依赖String.matches方法。`虽然String.matches方法最易于查看一个字符串是否与正则表达式相匹配，但并不适合在注重性能的情形中重复使用。`问题在于，它在内部为正则表达式创建了一个Pattern实例，却只用了一次，之后就可以进行垃圾回收了。创建Pattern实例的成本很高，因为需要将正则表达式编译成一个有限状态机。

为了提升性能，应该显式地将正则表达式编译成一个Pattern实例（不可变），让它成为类初始化的一部分，并将它缓存起来，每当调用isRomanNumeral方法的时候就重用同一个实例：

```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeral(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

改进后的isRomanNumeral方法如果被频繁地调用，会显示出明显的性能优势。除了提高性能之外，可以说代码也更清晰了。将不可见的Pattern实例做成final静态域时，可以给它起个名字，这样会比正则表达式本身更有可读性。

如果包含改进后的isRomanNumeral方法的类被初始化了，但是该方法没有被调用，那就没必要初始化ROMAN域。通过在isRomanNumeral方法第一次被调用的时候延迟初始化这个域，有可能消除这个不必要的初始化工作，但是不建议这样做。正如延迟初始化中常见的情况一样，这样做会使方法的实现更加复杂，从而无法将性能显著提高到超过已经达到的水平。

如果一个对象是不可变的，那么它显然能够被安全地重用，但其他有些情形则并不总是这么明显。考虑适配器的情形，有时也叫作视图。适配器是指这样一个对象：它把功能委托给一个后备对象，从而为后备对象提供一个可以替代的接口，由于适配器除了后备对象之外，没有其他的状态信息，所以针对某个给定对象的特定适配器而言，它不需要创建多个适配器实例。

例如，Map接口的keySet方法返回该Map对象的Set视图，其中包含该Map中所有的键（key）。乍看之下，好像每次调用keySet都应该创建一个新的Set实例，但是，对于一个给定的Map对象，实际上每次调用keySet都返回同样的Set实例。虽然被返回的Set实例一般是可改变的，但是所有返回的对象在功能上是等同的：当其中一个返回对象发生变化的时候，所有其他的返回对象也要发生变化，因为它们是由同一个Map实例支撑的。虽然创建keySet视图对象的多个实例并无害处，却是没有必要，也没有好处的。

另一种创建多余对象的方法，称作`自动装箱`，它允许程序员将基本类型和装箱基本类型混用，按需要自动装箱和拆箱，`自动装箱使得基本类型和装箱基本类型之间的差别变得模糊起来，但是并没有完全消除。`它们在语义上还有这微妙的差别，在性能上也有着比较明显的差别。请看下面的程序，它计算所有int正整数值的总和：

```java
private static long sum() {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
    return sum;
}
```

这段程序算出的答案是正确的，但是比实际情况要更慢一些，只因为变量sum被声明成Long而不是long，意味着程序构造了大约2^31个多余的Long实例。结论很明显：`要优先使用基本类型而不是装箱基本类型，要当心无意识的自动装箱。`

不要错误地认为本条目所介绍的内容暗示着”创建对象的代价非常昂贵，我们应该要尽可能地避免创建对象”。相反，由于小对象的构造器只做很少量的显式工作，所以小对象的创建和回收动作是非常廉价的，特别是在现代的JVM实现上更是如此。通过创建附加的对象，提升程序的清晰性、简洁性和功能性，这通常是件好事。

反之，通过维护自己的`对象池`来避免创建对象并不是一种好的作法，除非池中的对象是非常重量级的。正确使用对象池的典型对象示例就是数据库连接池。建立数据库连接的代价是非常昂贵的，因此重用这些对象非常有意义。而且，数据库的许可可能限制你只能使用一定数量的连接。但是，一般而言，维护自己的对象池必定会把代码弄得很乱，同时增加内存占用，并且还会损害性能。现代的JVM实现具有高度优化的垃圾回收器，其性能很容易就会超过轻量级对象池的性能。

# 第7条：消除过期的对象引用

```java
public class Stack {
    private Object[] elements;

    private int size = 0;

    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```

这段程序中并没有很明显的错误。无论如何测试，它都会成功地通过每一项测试，但是这个程序中存在“内存泄漏”。随着垃圾回收器活动的增加，或者由于内存占用的不断增加，程序性能的降低会逐渐表现出来。

如果一个栈先是增长，然后再收缩，那么，从栈中弹出来的对象将不会被当作垃圾回收，即使使用栈的程序不再引用这些对象，它们也不会被回收。这是因为栈内部维护着对这些对象的`过期引用`。所谓的过期引用，是指永远也不会再被解除的引用。在本例中，凡是在elements数组的“活动部分”之外的任何引用都是过期的。活动部分是指elements中下标小于size的那些元素。

在支持垃圾回收的语言中，内存泄漏是很隐蔽的。如果一个对象引用被无意识地保留起来了，那么垃圾回收机制不仅不会处理这个对象，而且也不会处理被这个对象所引用的所有其他对象。即使只有少量的几个对象引用被无意识地保留下来，也会有许许多多的对象被排除在垃圾回收机制之外，从而对性能造成潜在的重大影响。

这类问题的修复方法很简单：一旦对象引用已经过期，只需清空这些引用即可。

```java
public Object pop() {
    if (size == 0) {
        throw new EmptyStackException();
    }
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
```

清空过期引用的另一个好处是，如果它们以后又被错误地解除引用，程序就会立即抛出NullPointerException异常，而不是悄悄地错误运行下去。

但是`清空对象引用应该是一种例外，而不是一种规范行为`。消除过期引用最好的方法是让包含该引用的变量结束其生命周期（在最紧凑的作用域范围内定义每一个变量）。

那么，何时应该清空引用呢？Stack类的哪方面特性使它易于遭受内存泄漏的影响呢？简而言之，问题在于，Stack类自己管理内存。存储池包含了elements数组（对象引用单元，而不是对象本身）的元素。数组活动区域中的元素是已分配的，而数组其余部分的元素则是自由的。但是垃圾回收器并不知道这一点；对于垃圾回收器而言，elements数组中的所有对象引用都同等有效。只有程序员知道数组的非活动部分是不重要的。程序员可以把这个情况告知垃圾回收器：一旦数组元素变成了非活动部分的一部分，程序员就手工清空这些数组元素。

`只要类是自己管理内存，程序员就应该警惕内存泄漏问题。`一旦元素被释放掉，则该元素中包含的任何对象引用都应该被清空。

`内存泄漏的另一个常见来源是缓存。`一旦你把对象引用放到缓存里，它就很容易被遗忘掉，从而使得它不再有用之后很长一段时间内仍然留在缓存中。如果你正好要实现这样的缓存：只要在缓存之外存在对某个项的键的引用，该项就有意义，那么就可以使用WeakHashMap代表缓存；当缓存中的项过期之后，它们就会自动被删除。只有当所要的缓存项的生命周期是由该键的外部引用而不是由值决定时，WeakHashMap才有用处。

更为常见的情形则是，“缓存项的生命周期是否有意义”并不是很容易确定，随着时间的推移，其中的项会变得越来越没有价值。在这种情况下，缓存应该时不时地清除掉没用的项。这项清除工作可以由一个后台线程来完成，或者也可以在给缓存添加新条目的时候顺便进行清理。LinkedHashMap类利用它的removeEldestEntry方法可以很容易地实现后一种方案。

`内存泄漏的第三个常见来源是监听器和其他回调。`如果你实现了一个API，客户端在这个API中注册回调，却没有显式地取消注册，那么除非你采取某些动作，否则它们就会不断地堆积起来。确保回调立即被当作垃圾回收的最佳方法是只保存它们的弱引用，例如，只将它们保存成WeakHashMap中的键。

由于内存泄漏通常不会表现成明显的失败，所以它们可以在一个系统中存在很多年。往往只有通过仔细检查代码，或者借助于Heap剖析工具才能发现内存泄漏问题。因此，如果能够在内存泄漏发生之前就知道如何预测此类问题，并阻止它们发生，那是最好不过的了。

# 第8条：避免使用终结方法和清除方法

`终结方法（finalizer）通常是不可预测的，也是很危险的，一般情况下是不必要的`。使用终结方法会导致行为不稳定、性能降低，以及可移植性问题。在Java 9中用清除方法（cleaner）代替了终结方法。`清除方法没有终结方法那么危险，但仍然是不可预测、运行缓慢，一般情况下也是不必要的`。

不要把终结方法当作是C++中析构器的对应物。在C++中，析构器是回收一个对象所占用资源的常规方法，是构造器所必需的对应物。在Java中，当一个对象变得不可达的时候，垃圾回收器会回收与该对象相关联的存储空间，并不需要程序员做专门的工作。C++的析构器也可以被用来回收其他的非内存资源。而在Java中，一般用try-finally块来完成类似的工作。

终结方法和清除方法的缺点在于不能保证会被及时执行。从一个对象变得不可到达开始，到它的终结方法被执行，所花费的这段时间是任意长的。这意味着，注重时间的任务不应该由终结方法或者清除方法来完成。例如，用终结方法或者清除方法来关闭已经打开的文件，就是一个严重的错误，因为打开文件的描述符是一种很有限的资源。如果系统无法及时运行终结方法或者清除方法就会导致大量的文件仍然保留在打开状态，于是当一个程序再也不能打开文件的时候，它可能会运行失败。

及时地执行终结方法和清除方法正是垃圾回收算法的一个主要功能，这种算法在不同的JVM实现中会大相径庭。如果程序依赖于终结方法或者清除方法被执行的时间掉，那么这个程序的行为在不同的JVM中运行的表现就会截然不同。一个程序在你测试用的JVM平台上运行得非常好，而在你最重要顾客的JVM平台上却根本无法运行，这是完全有可能的。

延迟终结过程并不只是一个理论问题。在很少见的情况下，为类提供终结方法，可能会随意延迟其实力的回收过程。一位同事最近在调试一个长期运行的GUI应用程序的时候，该应用程序莫名其妙地出现OutOfMemoryError错误而死掉。分析表明，该应用程序死掉的时候，其终结方法队列中有数千个图形对象正在等待被终结和回收。遗憾的是，终结方法线程的优先级比该应用程序的其他线程的优先级要低得多，所以，图形对象的终结速度达不到它们进入队列的速度。Java语言规范并不保证哪个线程将会执行终结方法，所以，除了不使用终结方法之外，并没有很轻便的办法能够避免这样的问题。在这方面，清除方法比终结方法稍好一些，因为类的设计者可以控制自己的清除线程，但清除方法仍然在后台运行，处于垃圾回收器的控制之下，因此不能确保及时清除。

Java语言规范不仅不保证终结方法或者清除方法会被及时地执行，而且根本就不保证它们会被执行。当一个程序终止的时候，某些已经无法访问的对象上的终结方法却根本没有被执行，这是完全有可能的。结论是：`永远不应该依赖终结方法或者清除方法来更新重要的持久状态`。例如，依赖终结方法或者清除方法来释放共享资源（比如数据库）上的永久锁，这很容易让整个分布式系统垮掉。

不要被System.gc和System.runFinalization这两个方法所诱惑，它们确实增加了终结方法或者清除方法被执行的机会，但是它们并不保证终结方法或者清除方法一定会被执行。唯一声称保证它们会被执行的两个方法是System.runFinalizersOnExit和Runtime.runFinalizersOnExit。这两个方法都有致命的缺陷，并且已经被废弃很久了。

使用终结方法的另一个问题是：如果忽略在终结过程中被抛出来的未被捕获的异常，该对象的终结过程也会终止。未被捕获的异常会使对象处于破坏的状态，如果一个线程企图使用这种被破坏的对象，则可能发生任何不确定的行为。正常情况下，未被捕获的异常将会使线程终止，并打印出栈轨迹，但是，如果异常发生在终结方法之中，则不会如此，甚至连警告都不会打印出来。清除方法没有这个问题，因为使用清除方法的一个类库在控制它的线程。

`使用终结方法和清除方法有一个非常严重的性能损失`。

终结方法有一个严重的安全问题：它们为终结方法攻击打开了类的大门。终结方法攻击背后的思想很简单：如果从构造器或者它的序列化对等体（readObject和readResolve方法）抛出异常，恶意子类的终结方法就可以在构造了一部分的应该已经半途夭折的对象上运行。这个终结方法会将对该对象的引用记录在一个静态域中，阻止它被垃圾回收。一旦记录到异常的对象，就可以轻松地在这个对象上调用任何原本永远不允许在这里出现的方法。`从构造器抛出的异常，应该足以防止对象继续存在；有了终结方法的存在，这一点就做不到了`。这种攻击可能造成致命的后果。final类不会受到终结方法攻击，因为没有人能够编写出final类的恶意子类。`为了防止非final类受到终结方法攻击，要编写一个空的final的finalize方法`。

那么，如果类的对象中封装的资源（例如文件或者线程）确实需要终止，应该怎么做才能不用编写终结方法或者清除方法呢？`只需让类实现AutoCloseable，并要求其客户端在每个实例不再需要的时候调用close方法，一般是利用try-with-resources确保终止，即时遇到异常也是如此`。值得提及的一个细节是，该实例必须记录下自己是否已经被关闭了：close方法必须在一个私有域中记录下“该对象已经不再有效”。如果这些方法是在对象已经终止之后被调用，其他的方法就必须检查这个域，并抛出IllegalStateException异常。

那么终结方法和清除方法有什么好处呢？它们有两种合法用途。第一种用途是，当资源的所有者忘记调用它的close方法时，终结方法或者清除方法可以充当“安全网”。虽然这样做并不能保证终结方法或者清除方法会被及时地运行，但是在客户端无法正常结束操作的情况下，迟一点释放资源总比永远不释放要好。如果考虑编写这样的安全网终结方法，就要认真考虑清除，这种保护释放值得付出这样的代价。有些Java类（如FileInputStream、FileOutputStream、ThreadPoolExecutor和java.sql.Connection）都具有能充当安全网的终结方法。

清除方法的第二种合理用途与对象的本地对等体有关。本地对等体是一个本地（非Java的）对象，普通对象通过本地方法委托给一个本地对象。因为本地对等体不是一个普通对象，所以垃圾回收器不会知道它，当它的Java对等体被回收的时候，它不会被回收。如果本地对等体没有关键资源，并且性能也可以接受的话，那么清除方法或者终结方法正是执行这项任务最合适的工具。如果本地对等体拥有必须被及时终止的资源，或者性能无法接受，那么该类就应该具有一个close方法，如前所述。

```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {
        int numJunkPiles;

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        @Override
        public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    private final State state;

    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() {
        cleanable.clean();
    }
}
```

内嵌的静态类State保存清除方法清除房间所需的资源。在这个例子中，就是numJunkPiles域，表示房间的杂乱度。更现实地说，它可以是final的long，包含一个指向本地对等体的指针。State实现了Runnable接口，它的run方法最多被Cleanable调用一次，后者是我们在Room构造器中用清除器注册State实例时获得的。以下两种情况之一会触发run方法的调用：通常是通过调用Room的close方法触发的，后者又调用了Cleanable的清除方法。如果到了Room实例应该被垃圾回收时，客户端还没有调用close方法，清除方法就会（希望如此）调用State的run方法。

关键是State实例没有引用它的Room实例。如果它引用了，会造成循环，阻止Room实例被垃圾回收（以及防止被自动清除）。因此State必须是一个静态的嵌套类，因为非静态的嵌套类包含了对其外围实例的引用。同样地，也不建议使用lamda，因为它们很容易捕捉到外围对象的引用。

如前所述，Room的清除方法只用作安全网。如果客户端将所有的Room实例都包在try-with-resource块中，将永远不会请求到自动清除。

```java
public class Adult {
    public static void main(String[] args) {
        try (Room myRoom = new Room()) {
            System.out.println("Goodbye");
        }
    }
}
```

运行Adult程序会打印出Goodbye，接着是Cleaning room。

```java
public class Teenager {
    public static void main(String[] args) {
        new Room(99);
        System.out.println("Peace out");
    }
}
```

你可能期望打印出Peace out，然后是Cleaning room，但是在我的机器上，它没有打印出Cleaning room，就退出程序了。这就是不可预见性。Cleaner规范指出：”清除方法在System.exit期间的行为是与实现相关的，不确保清除动作是否会被调用。”虽然规范没有指明，其实对于正常的程序退出也是如此。在我的机器上，只要在Teenager的main方法上添加代码行System.gc()，就足以让它在退出之前打印Cleaning room，但是不能保证在你的机器上也能看到相同的行为。

总而言之，除非是作为安全网，或者是为了终止非关键的本地资源，否则请不要使用清除方法，对于在Java 9之前的发行版本，请尽量不要使用终结方法。若使用了终结方法或者清除方法，则要注意它的不确定性和性能后果。

# 第10条：覆盖equals时请遵守通用约定

覆盖equals方法看起来似乎很简单，但是有许多覆盖方式会导致错误，并且后果非常严重。最容易避免这类问题的办法就是不覆盖equals方法，在这种情况下，类的每个实例都只与它自身相等。如果满足了以下任何一个条件，这就正是所期望的结果：

（1）`类的每个实例本质上都是唯一的。`对于代表活动实体而不是值的类来说确实如此，例如Thread。Object提供的equals实现对于这些类来说正是正确的行为。

（2）`类没有必要提供“逻辑相等”的测试功能。`例如，java.util.regex.Pattern可以覆盖equals方法，以检查两个Pattern实例是否代表同一个正则表达式，但是设计者并不认为客户端需要或者期望这样的功能。在这类情况之下，从Object继承得到的equals实现已经足够了。

（3）`超类已经覆盖了equals，超类的行为对于这个类也是合适的。`例如，大多数的Set实现都从AbstractSet继承equals实现，List实现从AbstractList继承equals实现，Map实现从AbstractMap继承equals实现。

（4）`类是私有的，或者是包级私有的，可以确定它的equals方法永远不会被调用。`如果你非常想要规避风险，可以覆盖equals方法，以确保它不会被以外调用：

```java
@Override
public boolean equals(Object o) {
    throw new AssertionError();
}
```

那么，什么时候应该覆盖equals方法呢？如果类具有自己特有的“逻辑相等”概念（不同于对象等同的概念），而且超类还没有覆盖equals。这通常属于“值类”的情形。值类仅仅是一个表示值的类，例如Integer或者String。程序员在利用equals方法来比较值对象的引用时，希望知道它们在逻辑上是否相等，而不是想了解它们是否指向同一个对象。为了满足程序员的要求，不仅必须覆盖equals方法，而且这样做也使得这个类的实例可以被用作映射表（map）的键（key），或者集合（set）的元素，使映射或者集合表现出预期的行为。

有一种“值类”不需要覆盖equals方法，即用实例受控确保”每个值至多只存在一个对象”的类。枚举类型就属于这种类。对于这样的类而言，逻辑相同与对象等同是一回事，因此Object的equals方法等同于逻辑意义上的equals方法。

在覆盖equals方法的时候，必须要遵守它的通用约定。

equals方法实现了等价关系，其属性如下：

（1）`自反性`：对于任何非null的引用值x，x.equals(x)必须返回true。

对象必须等于其自身。很难想象会无意识地违反这一条。假如违背了这一条，然后把该类的实例添加到集合中，该集合的contains方法将果断告诉你，该集合不包含你刚刚添加的实例。

（2）`对称性`：对于任何非null的引用值x和y，当且仅当y.equals(x)返回true时，x.equals(y)必须返回true。

任何两个对象对于“它们是否相等”的问题都必须保持一致。与第一个要求不同，若无意中违反这一条，这种情形倒是不难想象。

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) {
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        }
        if (o instanceof String) {
            return s.equalsIgnoreCase((String) o);
        }
        return false;
    }
}
```

在这个类中，equals方法的意图非常好，它企图与普通的字符串对象进行互操作。假设我们有一个不区分大小写的字符串和一个普通的字符串：

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

不出所料，cis.equals(s)返回true。问题在于，虽然CaseInsensitiveString类中的equals方法知道普通的字符串对象，但是，String类的equals方法却并不知道不区分大小写的字符串。因此，s.equals(cis)返回false，显然违反了对称性。假设你把不区分大小写的字符串对象放到一个集合中：

```java
List<CaseInsensitiveString> list = new ArrayList<>();
list.add(cis);
```

此时list.contains(s)会返回什么结果呢？没人知道。在当前的OpenJDK实现中，它碰巧返回false，但这只是这个特定实现得出的结果而已。在其他的实现中，它有可能返回true，或者抛出一个运行时异常。一旦违反了equals约定，当其他对象面对你的对象时，你完全不知道这些对象的行为会怎么样。

为了解决这个问题，只需要把企图与String互操作的这段代码从equals方法中去掉就可以了。这样做之后，就可以重构该方法，使它变成一条单独的返回语句：

```java
@Override
public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equals(s);
}
```

（3）`传递性`：对于任何非null的引用值x、y和z，如果x.equals(y)返回true，并且y.equals(z)也返回true，那么x.equals(z)也必须返回true。

如果一个对象等于第二个对象，而第二个对象又等于第三个对象，则第一个对象一定等于第三个对象。同样地，无意识地违反这条规则的情形也不难想象。用子类举个例子。假设它将一个新的值组件添加到了超类中。换句话说，子类增加的信息会影响equals的比较结果。

```java
public class Point {
    private final int x;

    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

假设你想扩展这个类，为一个点添加颜色信息：

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
}
```

equals方法会是什么样的呢？如果完全不提供equals方法，而是直接从Point继承过来，在equals做比较的时候颜色信息就被忽略掉了。虽然这样做不会违反equals约定，但很明显这是无法接受的。假设编写了一个equals方法，只有当它的参数是另一个有色点，并且具有同样的位置和颜色时，它才会返回true：

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof ColorPoint)) {
        return false;
    }
    return super.equals(o) && ((ColorPoint) o).color = color;
}
```

这个方法的问题在于，在比较普通点和有色点，以及相反的情形时，可能会得到不同的结果。前一种比较忽略了颜色信息，而后一种比较则总是返回false，因为参数的类型不正确。为了直观地说明问题所在，我们创建一个普通点和一个有色点：

```java
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);
```

然后，p.equals(cp)返回true，cp.equals(p)则返回false。你可以做这样的尝试来修正这个问题，让ColorPoint.equals在进行混合比较时忽略颜色信息：

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point)) {
        return false;
    }
    if (!(o instanceof ColorPoint)) {
        return o.equals(this);
    }
    return super.equals(o) && ((ColorPoint) o).color = color;
}
```

这种方法确实提供了对称性，但是却牺牲了传递性：

```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

此时，p1.equals(p2)和p2.equals(p3)都返回true，但是p1.equals(p3)则返回false，很显然这违反了传递性。前两种比较不考虑颜色信息，而第三者比较则考虑了颜色信息。

此外，这种方法还可能导致无限递归问题：假设Point有两个子类，如ColorPoint和SmellPoint，它们各自都带有这种equals方法。那么对myColorPoint.equals(mySmellPoint)的调用将会抛出StackOverflowError异常。

那该怎么解决呢？事实上，这是面向对象语言中关于等价关系的一个基本问题。`我们无法再扩展可实例化的类的同时，既增加新的值组件，同时又保留equals约定`，除非愿意放弃面向对象的抽象所带来的优势。

你可能听说过，在equals方法中用getClass测试代替instanceof测试，可以扩展可实例化的类和增加新的值组件，同时保留equals约定：

```java
@Override
public boolean equals(Object o) {
    if (null == o || o.getClass() != getClass()) {
        return false;
    }
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

这段程序只有当对象具有相同的实现类时，才能使对象等同。虽然这样也不算太糟糕，但结果却是无法接受的：Point子类的实例仍然是一个Point，它仍然需要发挥作用，但是如果采用了这种方法，它就无法完成任务！假设我们要编写一个方法，以检验某个点是否处在单位圆中：

```java
private static final Set<Point> unitCircle = Set.of(new Point(1, 0), new Point(0, 1), new Point(-1, 0), new Point(0, -1));

public static boolean onUnitCircle(Point p) {
    return unitCircle.contains(p);
}
```

虽然这可能不是实现这种功能的最快方式，不过它的效果很好。但是假设你通过某种不增加值组件的方式扩展了Point，例如让它的构造器记录创建了多少个实例：

```java
public class CounterPoint extends Point {
    private static final AtomicInteger counter = new AtomicInteger();

    public CounterPoint(int x, int y) {
        super(x, y);
        counter.incrementAndGet();
    }

    public static int numberCreated() {
        return counter.get();
    }
}
```

里氏替换原则认为，一个类型的任何重要属性也将适用于它的子类型，因此为该类型编写的任何方法，在它的子类型上也应该同样运行得很好。针对上述Point的子类（如CounterPoint）仍然是Point，并且必须发挥作用的例子，这个就是它的正式语句。但是假设我们将CounterPoint实例传递给了onUnitCircle方法。如果Point类使用了基于getClass的equals方法，无论CounterPoint实例的x和y值是什么，onUnitCircle方法都会返回false。这是因为像onUnitCircle方法所用的HashSet这样的集合，利用equals方法检验包含条件，没有任何CounterPoint实例域任何Point对应。但是，如果在Point上使用基于instanceof的equals方法，当遇到CounterPoint时，onUnitCircle方法就会工作得很好。

虽然没有一种令人满意的方法可以既扩展不可实例化的类，又增加值组件，但还是有一种不错的权宜之计：我们不再让ColorPoint扩展Point，而是在ColorPoint中加入一个私有的Point域，以及一个公有的视图（view）方法，此方法返回一个与该有色点处在相同位置的普通Point对象：

```java
public class ColorPoint {
    private final Point point;

    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
```

在Java平台类库中，有一些类扩展了可实例化的类，并添加了新的值组件。例如，java
.sql.Timestamp对java.util.Date进行了扩展，并增加了nanoseconds域。Timestamp的equals实现确实违反了对称性，如果Timestamp和Date对象用于同一个集合中，或者以其他方式被混合在一起，则会引起不正确的行为。Timestamp类有一个免责声明，告诫程序员不要混合使用Date和Timestamp对象。只要你不把它们混合在一起，就不会有麻烦，除此之外没有其他的措施可以防止你这么做，而且结果导致的错误将很难调试。Timestamp类的这种行为是个错误，不值得仿效。

注意，你可以在一个抽象类的子类中增加新的值组件且不违反equals约定。例如，你可能有一个抽象的Shape类，它没有任何值组件，Circle子类添加了一个radius域，Rectangle子类添加了length和width域。只要不可能直接创建超类的实例，前面所述的种种问题就都不会发生。

（4）`一致性`：对于任何非null的引用值，只要equals的比较操作在对象中所用的信息没有被修改，多次调用x.equals(y)就会一致地返回true，或者一致地返回false。

如果两个对象相等，它们就必须始终保持相等，除非它们中有一个对象（或者两个都）被修改了。换句话说，可变对象在不同的时候可以与不同的对象相等，而不可变对象则不会这样。当你在写一个类的时候，应该仔细考虑它是否应该是不可变的。如果认为它应该是不可变的，就必须保证equals方法满足这样的限制条件：相等的对象永远相等，不相等的对象永远不相等。

无论类是否是不可变的，都`不要使equals方法依赖于不可靠的资源`。如果违反了这条禁令，要想满足一致性的要求就十分困难了。例如，java.net.URL的equals方法依赖于对URL中主机IP地址的比较。将一个主机名转变成IP地址可能需要访问网络，随着时间的推移，就不能确保会产生相同的结果，即有可能IP地址发生了改变。这样会导致URL equals方法违反equals约定，在实践中有可能引发一些问题。URL equals方法的行为是一个大错误并且不应该被模仿。遗憾的是，因为兼容性的要求，这一行为无法被改变。为了避免发生这种问题，equals方法应该对驻留在内存中的对象执行确定性的计算。

（5）对于任何非null的引用值x，x.equals(null)必须返回false。

尽管很难想象在扫描情况下o.equals(null)调用会意外地返回true，但是意外抛出NullPointerException异常的情形却不难想象。通用约定不允许抛出NullPointerException异常。许多类的equals方法都通过一个显式的null测试来防止这种情况：

```java
@Override
public boolean equals(Object o) {
    if (null == o) {
        return false;
    }
}
```

这些测试时不必要的。为了测试其参数的等同性，equals方法必须先把参数转换成适当的类型，以便可以调用它的访问方法，或者访问它的域。在进行转换之前，equals方法必须使用instanceof操作符，检查其参数的类型是否正确：

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof MyType)) {
        return false;
    }
    MyType mt = (MyType) o;
}
```

如果漏掉了这一步的类型检查，并且传递给equals方法的参数又是错误的类型，那么equals方法将会抛出ClassCastException异常，这就违反了equals约定。但是，如果instanceof的第一个参数为null，那么，不管第二个操作数是哪种类型，instanceof操作符都指定应该返回false。因此，如果把null传给equals方法，类型检查就会返回false，所以不需要显式的null检查。

结合所有这些要求，得出了以下实现高质量equals方法的诀窍：

（1）`使用==操作符检查“参数是否为这个对象的引用”。`如果是，则返回true。这只不过是一种性能优化，如果比较操作有可能很昂贵，就值得这么做。

（2）`使用instanceof操作符检查“参数是否为正确的类型”。`如果不是，则返回false。一般来说，所谓”正确的类型”是指equals方法所在的那个类。某些情况下，是指该类所实现的某个接口。如果类实现的接口改进了equals约定，允许在实现了该接口的类之间进行比较，那么就使用接口。集合接口改进了equals约定，允许在实现了该接口的类之间进行比较，那么就使用接口。集合接口如Set、List、Map和Map.Enrty具有这样的特性。

（3）`把参数转换成正确的类型。`因为转换之前进行过instanceof测试，所以确保会成功。

（4）`对于该类中的每个“关键域”，检查参数中的域是否与该对象中对应的域相匹配。`如果这些测试全部成功，则返回true；否则返回false。如果第2步中的类型是个接口，就必须通过接口方法访问参数中的域；如果该类型是个类，也许就能够直接访问参数中的域，这要取决于它们的可访问性。

对于既不是float也不是double类型的基本类型域，可以使用==操作符进行比较；对于对象引用域，可以递归地调用equals方法；对于float域，可以使用静态Float.compare(float, float)方法；对于double域，则使用Double.compare(double, double)。对float和double域进行特殊的处理是有必要的，因为存在着Float.NaN、-0.0f以及类似的double常量。虽然可以用静态方法Float.equals和Double.equals对float和double域进行比较，但是每次比较都要进行自动装箱，这会导致性能下降。对于数组域，则要把以上这些指导原则应用到每一个元素上。如果数组域中的每个元素都很重要，就可以使用一个Arrays.equals方法。

有些对象引用域包含null可能是合法的，所以，为了避免可能导致NullPointerException异常，则使用静态方法Objects.equals(Object, Object)来检查这类域的等同性。

对于有些类，比如前面提到的CaseInsensitiveString类，域的比较要比简单的等同性测试复杂得多。如果是这种情况，可能希望保存该域的一个“范式”，这样equals方法就可以根据这些范式进行低开销的精确比较，而不是高开销的非精确比较。这种方法对于不可变类是最为合适的；如果对象可能发生变化，就必须使其范式保持最新。

域的比较顺序可能会影响equals方法的性能。为了获得最佳的性能，应该最先比较最有可能不一致的域，或者是开销最低的域，最理想的情况是两个条件同时满足的域，不应该比较那些不属于对象逻辑状态的域，例如用于同步操作的Lock域。也不需要比较衍生域，因为这些域可以由”关键域”计算获得，但是这样做有可能提高equals方法的性能。如果衍生域代表了整个对象的综合描述，比较这个域可以节省在比较失败时区比较实际数据所需要的开销。例如，假设有一个Polygon类，并缓存了该面积。如果两个多边形有着不同的面积，就没有必要去比较它们的边和顶点。

`在编写完equals方法之后，应该问自己三个问题：它是否是对称的、传递的、一致的？`并且不要只是自问，还要编写单元测试来检验这些特性。

```java
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "area code");
        this.prefix = rangeCheck(prefix, 999, "prefix");
        this.lineNum = rangeCheck(lineNum, 9999, "line num");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max) {
            throw new IllegalArgumentException(arg + ": " + val);
        }
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }
        if (!(o instanceof PhoneNumber)) {
            return false;
        }
        PhoneNumber pn = (PhoneNumber) o;
        return pn.lineNum = lineNum && pn.prefix = prefix && pn.areaCode == areaCode;
    }
}
```

下面是最后的一些告诫：

（1）`覆盖equals时总要覆盖hashCode。`

（2）`不要企图让equals方法过于智能。`如果只是简单地测试域中的值是否相等，则不难做到遵守equals约定。如果想过度地去寻求各种等价关系，则很容易陷入麻烦之中。把任何一种别名形式考虑到等价的范围内，往往不会是个好主意。例如，File类不应该试图把指向同一个文件的符号链接（symbolic link）当作相等的对象来看待。所幸File类没有这样做。

（3）`不要将equals声明中的Object对象转换为其他的类型。`程序员编写出下面的equals方法并不鲜见，这会使程序员花上数个小时都搞不清为什么它不能正常工作：

```java
public boolean equals(MyClass o) {

}
```

问题在于，这个方法并没有覆盖Object.equals，因为它的参数应该是Object类型，相反，它重载了Object.equals。

# 第11条：覆盖equals时总要覆盖hashCode

在每个覆盖了equals方法的类中，都必须覆盖hashCode方法。如果不这样做的话，就会违反hashCode的通用约定，从而导致该类无法结合所有基于散列的集合一起正常运作，这类集合包含HashMap和HashSet。下面是约定的内容：

（1）在应用程序的执行期间，只要对象的equals方法的比较操作所用到的信息没有被修改，那么对同一个对象的多次调用，hashCode方法都必须始终返回同一个值。在一个应用程序与另一个程序的执行过程中，执行hashCode方法所返回的值可以不一致。

（2）如果两个对象根据equals(Object)方法比较是相等的，那么调用这两个对象中的hashCode方法都必须产生同样的整数效果。

（3）如果两个对象根据equals(Object)方法比较是不相等的，那么调用这两个对象中的hashCode方法，则不一定要求hashCode方法必须产生不同的结果。但是程序员应该知道，给不相等的对象产生截然不同的整数结果，有可能提高散列表（hashtable）的性能。

`因没有覆盖hashCode而违反的关键约定是第二条：相等的对象必须具有相等的散列码。`根据类的equals方法，两个截然不同的实例在逻辑上有可能是相等的，但是根据Object类的hashCode方法，它们仅仅是两个没有任何共同之处的对象。因此，对象的hashCode方法返回两个看起来是随机的整数，而不是根据第二个约定所要求的那样，返回两个相等的整数。

假设在HashMap中用第10条中出现过的PhoneNumber类的实例作为键：

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "Jenny");
```

此时，你可能期望m.get(new PhoneNumber(707, 867, 5309))会返回”Jenny"，但它实际上返回的是null。注意，这里涉及两个PhoneNumber的实例：第一个被插入HashMap中，第二个实例与第一个相等，用于从Map中根据PhoneNumber去获取用户名字。由于PhoneNumber类没有覆盖hashCode方法，从而导致两个相等的实例具有不相等的散列码，违反了hashCode的约定。因此，put方法把电话号码对象存放在一个散列桶中，get方法却在另一个散列桶中查找这个电话号码。即使这两个实例正好被放到同一个散列桶中，get方法也必定会返回null，因为HashMap有一项优化，可以将每个项相关联的散列码缓存起来，如果散列码不匹配，也就不再去检验对象的等同性。

修正这个问题非常简单，只需为PhoneNumber类提供一个适当的hashCode方法即可。那么，hashCode方法应该是什么样的呢？编写一个合法但并不好用的hashCode方法没有任何价值。例如，下面这个方法总是合法的，但是它永远都不应该被正式使用：

```java
@Override
public int hashCode() {
    return 42;
}
```

上面这个hashCode方法是合法的，因为它确保了相等的对象总是具有同样的散列码。但它也极为恶劣，因为它使得每个对象都具有同样的散列码。因此，每个对象都被映射到同一个散列桶中，使散列表退化为链表。它使得本该线性时间运行的程序变成了以平方级时间在运行。对于规模很大的散列表而言，这会关系到散列表能否正常工作。

一个好的散列函数通常倾向于“为不相等的对象产生不相等的散列码”。这正是hashCode约定中第三条的定义。理想情况下，散列函数应该把集合中不相等的实例均匀地分布到所有可能的int值上。要想完全达到这种理想的情形是非常困难的。幸运的是，相对接近这种理想情形则并不太困难。

Objects类有一个静态方法，它带有任意数量的对象，并为它们返回一个散列码：

```java
@Override
public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

如果一个类是不可变的，并且计算散列码的开销也比较大，就应该考虑把散列码缓存在对象内部，而不是每次请求的时候都重新计算散列码。如果你觉得这种类型的大多数对象会被用作散列键，就应该在创建实例的时候计算散列码。否则，可以选择“延迟初始化”散列码，即一直到hashCode被第一次调用的时候才初始化：

```java
private int hashCode;

@Override
public int hashCode() {
    int result = hashCode;
    if (result == 0) {
        result = Short.hashCode(areaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```

`不要试图从散列码计算中排除掉一个对象的关键域来提高性能。`虽然这样得到的散列函数运行起来可能更快，但是它的效果不见得会好，可能会导致散列表慢到根本无法使用。特别是在实践中，散列函数可能面临大量的实例，在你选择忽略的区域之中，这些实例仍然区别非常大。如果是这样，散列函数就会把所有这些实例映射到极少数的散列码上，原本应该以线性级时间运行的程序，将会以平方级的时间运行。

这在Java 2发行版本之前，一个String散列函数最多只能使用16个字符，若长度少于16个字符就计算所有的字符，否则就从第一个字符开始，在整个字符串中间隔均匀地选取样本进行计算。对于像URL这种层次名称的大型集合，该散列函数正好表现出了这里所提高的病态行为。

`不要对hashCode方法的返回值做出具体的规定，因此客户端无法理所当然地依赖它；这样可以为修改提供灵活性。`Java类库中的许多类，比如String和Integr，都可以把它们的hashCode方法返回的确切值规定为该实例值的一个函数。一般来说，这并不是个好主意，因为这样做严格地限制了再未来的版本中改进散列函数的能力。如果没有规定散列函数的细节，那么当你发现了它的内部缺陷时，或者发现了更好的散列函数时，就可以在后面的发行版本中修正它。

总而言之，每当覆盖equals方法时都必须覆盖hashCode，否则程序将无法正确运行。hashCode方法必须遵守Object规定的通用约定，并且必须完成一定的工作，将不相等的散列码分配给不相等的实例。

# 第12条：始终要覆盖toString

虽然Object提供了toString方法的一个实现，但它返回的字符串通常并不是类的用户所期望看到的。它包含类的名称，以及一个“@”符号，接着是散列码的无符号十六进制表示法，例如PhoneNumber@163b91。toString的通用约定指出，被返回的字符串应该是一个“简洁的但信息丰富，并且易于阅读的表达形式”。尽管有人认为PhoneNumber@163b91算得上是简洁和易于阅读了，但是与707-867-5309比较起来，它还算不上是信息丰富的。toString约定进一步指出，“建议所有的子类都覆盖这个方法。”

遵守toString约定并不像遵守equals和hashCode的约定那么重要，但是，提供好的toString实现可以使类用起来更加舒适，使用了这个类的系统也更易于调试。当对象被传递给println、printf、字符串联操作符（+）以及assert，或者被调试器打印出来时，toString方法会被自动调用。即使你永远不调用对象的toString方法，但是其他人也许可能需要。例如，带有对象引用的一个组件，在它记录的错误消息中，可能包含该对象的字符串表示法。如果你没有覆盖toString，这条消息可能就毫无用处。

如果为PhoneNumber提供了好的toString方法，那么要产生有用的诊断消息会非常容易：

```java
System.out.println("Failed to connect to " + phoneNumber);
```

不管是否覆盖了toString方法，程序员都将以这种方式来产生诊断消息，但是如果没有覆盖toString方法，产生的消息将难以理解。提供好的toString方法，不仅有益于这个类的实例，同样也有益于那些包含这些实例的引用的对象，特别是集合对象。打印Map时会看到消息{Jenny = PhoneNumber@163b91}或{Jenny = 707-867-5309}，你更愿意看到哪一个？

`在实际应用中，toString方法应该返回对象中包含的所有值得关注的信息，`例如上述电话号码例子那样。如果对象太大，或者对象中包含的状态信息难以用字符串来表达，这样做就有点不切实际。

在实现toString的时候，必须要做出一个很重要的决定：是否在文档中指定返回值的格式。对于值类，比如电话号码类、矩阵类，建议这么做。指定格式的好处是，它可以被用作一种标准的、明确的、适合人阅读的对象表示法。如果你指定了格式，通常最好再提供一个相匹配的静态工厂或者构造器，以便程序员可以很容易地在对象及其字符串表示法之间来回切换。Java平台类库中的许多值类都采用了这种做法，包括BigInteger、BigDecimal和绝大多数的基本类型包装类。

指定toString返回值的格式也有不足之处：如果这个类已经被广泛使用，一旦指定格式，就必须始终如一地坚持这种格式。程序员会编写出相应的代码来解析这种字符串表示法、产生字符串表示法，以及把字符串表示法嵌入持久的数据中。如果将来的发行版本中改变了这种表示法，就会破坏他们的代码和数据。如果不指定格式，就可以保留灵活性，便于在将来的发行版本中增加信息，或者改进格式。

`无论是否指定格式，都应该在文档中明确地表明你的意图。`

无论是否指定格式，`都为toString返回值中包含的所有信息提供一种可以通过编程访问之的途径。`例如，PhoneNumber类应该包含针对area code、prefix和line number的访问方法。如果不这么做，就会迫使需要这些信息的程序员不得不自己去解析这些字符串。除了降低了程序的性能，使得程序员们去做这些不必要的工作之外，这个解析过程也很容易出错，会导致系统不稳定，如果格式发生变化，还会导致系统崩溃。如果没有提供这些访问方法，即使你已经指明了字符串的格式是毁变化的，这个字符串格式也成了事实上的API。

在静态工具类中编写toString方法是没有意义的。也不要在大多数枚举类型中编写toString方法，因为Java已经为你提供了非常完美的方法。但是，在所有其子类共享通用字符串表示法的抽象类中，一定要编写一个toString方法。例如，大多数集合实现中的toString方法都是继承自抽象的集合类。

# 第13条：谨慎地覆盖clone

Cloneable接口的目的是作为对象的一个mixin接口，表明这样的对象允许克隆。遗憾的是，它并没有成功地达到这个目的。它的主要缺陷在于缺少一个clone方法，而Object的clone方法是受保护的。如果不借助于反射，就不能仅仅因为一个对象实现了Cloneable，就调用clone方法。即使是反射调用也可能会失败，因为不能保证该对象一定具有可访问的clone方法。

既然Cloneable接口并没有包含任何方法，那么它到底有什么作用呢？它决定了Object中受保护的clone方法实现的行为：如果一个类实现了Cloneable，Object的clone方法就返回该对象的逐域拷贝，否则就会抛出CloneNotSupportedException异常。这是接口的一种极端非典型的用法，也不值得仿效。通常情况下，实现接口是为了表明类可以为它的客户端做些什么。然而，对于Cloneable接口，它改变了超类中受保护的方法的行为。

虽然规范中没有明确指出，`事实上，实现Cloneable接口的类是为了提供一个功能适当的公有的clone方法`。为了达到这个目的，类及其所有超类都必须遵守一个相当复杂的、不可实施的，并且基本上没有文档说明的协议。由此得到一种语言之外的机制：它无须调用构造器就可以创建对象。

clone方法的通用约定是非常弱的：

创建和返回该对象的一个拷贝。这个“拷贝”的精确含义取决于该对象的类。一般的含义是，对于任何对象x，表达式x.clone() != x将会返回结果true，并且表达式x.clone().getClass() == x.getClass()将会返回结果true，但这些都不是绝对的要求。虽然通常情况下，表达式x.clone().equals(x)将会返回结果true，但是，这也不是一个绝对的要求。

按照约定，这个方法返回的对象应该通过调用super.clone获得。如果类及其超类（Object除外）遵守这一约定，那么x.clone().getClass == x.getClass()。

按照约定，返回的对象应该不依赖于被克隆的对象。为了成功地实现这种独立性，可能需要在super.clone返回对象之前，修改对象的一个或更多个域。

这种机制大体上类似于自动的构造器调用链，只不过它不是强制要求的：如果类的clone方法返回的实例不是通过调用super.clone方法获得，而是通过调用构造器获得，编译器就不会发出警告，但是该类的子类调用了super(clone)方法，得到的对象就会拥有错误的类，并组织了clone方法的子类正常工作。如果final类覆盖了clone方法，那么这个约定可以被安全地忽略，因为没有子类需要担心它。如果有final类的clone方法没有调用super.clone方法，这个类就没有理由去实现Cloneable接口了，因为它不依赖于Object克隆实现的行为。

假设你希望在一个类中实现Cloneable接口，并且它的超类都提供了行为良好clone方法。首先，调用super.clone方法。由此得到的对象将是原始对象功能完整的克隆。在这个类中声明的域将等同于被克隆对象中相应的域。如果美国域包含一个基本类型的值，或者包含一个指向不可变对象的引用，那么被返回的对象则可能正是你所需要的对象，在这种情况下不需要再做进一步处理。但要注意，`不可变的类永远都不应该提供clone方法，`因为它只会激发不必要的克隆。

如果对象中包含的域引用了可变的对象，必须要拷贝该可变的对象。`Cloneable架构与引用可变对象的final域的政策用法是不相兼容的，除非是在原始对象和克隆对象之间可以安全地共享此可变对象`。为了使类成为可克隆的，可能有必要从某些域中去掉final修饰符。

递归地调用clone有时还不够。例如，假设你正在为一个散列表编写clone方法，它的内部数据包含一个散列桶数组，每个散列桶都指向“键-值”对链表的第一项。出于性能方面的考虑，该类实现了它自己的轻量级单向链表，而没有使用Java内部的java.util.LinkedList：

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
}
```

假设你仅仅递归地克隆这个散列桶数组：

```java
@Override
public HashTable clone() {
    try {
        HashTable result = (HashTable) super.clone();
        result.buckets = buckets.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

虽然被克隆对象有它自己的散列桶数组，但是，这个数组引用的链表与原始对象是一样的，从而很容易引起克隆对象和原始对象中不确定的行为。为了修正这个问题，必须单独地拷贝并组成每个桶的链表。下面是一种常见的做法：

```java
public class HashTable implements Cloneable {
    private Entry[] buckets = ...;

    private static class Entry {
        final Object key;
        Object value;
        Entry next;

        Entry(Object key, Object value, Entry next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }

        Entry deepCopy() {
            return new Entry(key, value, next == null ? null : next.deepCopy());
        }
    }

    @Override
    public HashTable clone() {
        try {
            HashTable result = (HashTable) super.clone();
            result.buckets = new Entry[buckets.length];
            for (int i = 0; i < buckets.length; i++) {
                if (buckets[i] != null) {
                    result.buckets[i] = buckets[i].deepCopy();
                }
            }
            return result;
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}
```

私有了HashTable.Entry被加强了，它支持一个“深度拷贝”方法。HashTable上的clone方法分配了一个大小适中、新的buckets数组，并且遍历原始的buckets数组，对每一个非空散列桶进行深度拷贝。Entry类中的深度拷贝方法递归地调用它自身，以便拷贝整个链表（它是链表的头节点）。虽然这种方法很灵活，如果散列桶不是很长，也会工作得很好，但是，这样克隆一个链表并不是一种好办法，因为针对列表中的每个元素，它都要消耗一段栈空间。如果链表比较长，这很容易导致栈溢出。为了避免发生这种情况，可以用迭代代替递归：

```java
Entry deepCopy() {
    Entry result = new Entry(key, value, next);
    for (Entry p = result; p.next != null; p = p.next) {
        p.next = new Entry(p.next.key, p.next.value, p.next.next);
    }
    return result;
}
```

克隆复杂对象的最后一种办法是，先调用super.clone()方法，然后把结果对象中的所有域都设置成它们的初始状态，然后调用高层的方法来重新产生对象的状态。在HashTable例子中，buckets域将被初始化为一个新的散列桶数组，然后，对于正在被克隆的散列表中的每一个键-值映射，都调用put(key, value)方法。这种做法往往会产生一个简单、合理且相当又没的clone方法，但是它运行起来通常没有“直接操作对象及其克隆对象的内部状态的clone方法”快。虽然这种方法干脆利落，但它与整个Cloneable架构是对立的，因为它完全抛弃了Cloneable架构基础的逐域对象复制的机制。

像构造器一样，clone方法也不应该在构造的过程中，调用可以覆盖的方法。如果clone调用了一个在子类中被覆盖的方法，那么在该方法所在的子类有机会修正它在克隆对象中的状态之前，该方法就会先被执行，这样很有可能会导致克隆对象和原始对象之间的不一致。因此，上一段中讨论的put(key, value)方法要么应是final的，要么应是私有的。（如果是私有的，它应该算是非final公有方法的“辅助方法”。）

Object的clone方法被声明为可抛出CloneNotSupportedException异常，但是，覆盖版本的clone方法可以忽略这个声明。`公有的clone方法应该省略throws声明，`因为不会抛出受检异常的方法使用起来更加轻松。

为继承设计类有两种选择，但是无论选择其中的哪一种方法，这个类都不应该实现Cloneable接口。你可以选择模拟Object的行为：实现一个功能适当的受保护的clone方法，它应该被声明抛出CloneNotSupportedException异常。这样可以使子类具有实现或不实现Cloneable接口的自由，就仿佛它们直接扩展了Object一样。或者，也可以选择不去实现一个有效的clone方法，并防止子类去实现它。只需要提供下列退化了的clone实现即可。

```java
@Override
protected final Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException();
}
```

还有一点值得注意。如果你编写线程安全的类准备实现Cloneable接口，要记住它的clone方法必须得到严格的同步。Object的clone方法没有同步。

所有实现了Cloneable接口的类都应该覆盖clone方法，并且是公有的方法，它的返回类型为类本身。该方法应该先调用super.clone方法，然后修正任何需要修正的域。一般情况下，这意味着要拷贝任何包含内部“深层结构”的可变对象，并用指向新对象的引用代替原来指向这些对象的引用。虽然，这些内部拷贝操作往往可以通过递归地调用clone来完成，但这通常并不是最佳方法。如果该类只包含基本类型的域，或者指向不可变对象的引用，那么多半的情况是没有域需要修正。这条规则也有例外，例如，代表序列化或其他唯一ID值的域，不管这些域是基本类型还是不可变的，它们也都需要被修正。

如果你扩展一个实现了Cloneable接口的类，那么你除了实现一个行为良好的clone方法外，没有别的选择。否则，最好提供某些其他的途径来代替对象拷贝。`对象拷贝的更好的办法是提供一个拷贝构造器或拷贝工厂。`拷贝构造器只是一个构造器，它唯一的参数类型是包含该构造器的类，例如：

```java
public Yum(Yum yum) {...};
```

拷贝工厂是类似于拷贝构造器的静态工厂：

```java
public static Yum newInstance(Yum yum) {...};
```

拷贝构造器的做法，及其静态工厂方法的变形，都比Cloneable/clone方法具有更多的优势：它们不依赖于某一种很有风险的、语言之外的对象创建机制；它们不要求遵守尚未制定好的文档规范；它们不会与final域的正常使用发生冲突；它们不会抛出不必要的受检异常；它们不需要进行类型转换。

甚至，拷贝构造器或者拷贝工厂可以带一个参数，参数类型是该类所实现的接口。例如，按照惯例所有通用集合实现都提供了一个拷贝构造器，其参数类型为Collection或者Map接口。基于接口的拷贝构造器和拷贝工厂（更准确的叫法应该是转换构造器和转换工厂），允许客户选择拷贝的实现类型，而不是强迫客户接受原始的实现类型。例如，假设你有一个HashSet:s，并且希望把它拷贝成一个TreeSet。clone方法无法提供这样的功能，但是用转换构造器很容易实现：new TreeSet<>(s)。

既然所有的问题都与Cloneable接口有关，新的接口就不应该扩展这个接口，新的可扩展的类也不应该实现这个接口。虽然final类实现Cloneable接口没有太大的危害，这个应该被视同性能优化，留到少数必要的情况下才使用。总之，复制功能最好由构造器或者工厂提供。这条规则最绝对的例外是数组，最好利用clone方法复制数组。

# 第14条：考虑实现Comparable接口

compareTo方法并没有在Object类中声明，它是Comparable接口中唯一的办法。compareTo方法不但允许进行简单的等同性比较，而且允许执行顺序比较。类实现了Comparable接口，就表明它的实例具有内在的排序关系。为实现Comparable接口的对象数组进行排序就这么简单：

```java
Arrays.sort(a);
```

对存储在集合中的Comparable对象进行搜索、计算极限值以及自动维护也同样简单。例如，下面的程序依赖于实现了Comparable接口的String类，它去掉了命令行参数列表中的重复参数，并按字母顺序打印出来：

```java
public class WordList {
    public static void main(String[] args) {
        Set<String> s = new TreeSet<>();
        Collections.addAll(s, args);
        System.out.println(s);
    }
}
```

一旦类实现了Comparable接口，它就可以跟许多泛型算法以及依赖于该接口的集合实现进行协作。你付出很小的努力就可以获得非常强大的功能。事实上，Java平台类库中的所有值类，以及所有的枚举类型都实现了Comparable接口。如果你正在编写一个值类，它具有非常明显的内在排序关系，比如按字母顺序、按数值顺序或者按年代顺序，那你就应该坚决考虑实现Comparable接口：

```java
public interface Comparable<T> {
    int compareTo(T t);
}
```

compareTo方法的通用约定与equals方法的约定相似：

将这个对象与指定的对象进行比较。当该对象小于、等于或大于指定对象的时候，分别返回一个负整数、零和正整数。如果由于指定对象的类型而无法与该对象进行比较，则抛出ClassCastException异常。

在下面的说明中，符号sgn表示signum函数，它根据表达式的值为负值、零和正值，分别返回-1、0或1。

（1）实现者必须确保所有的x和y都满足sgn(x.compareTo(y)) == -sgn(y.compareTo(x))。（这也暗示着，当且仅当y.compareTo(x)抛出异常时，x.compareTo(y)才必须抛出异常。）

（2）实现者还必须确保这个比较关系是可传递的：(x.compareTo(y) > 0 && y.compareTo(z) > 0)暗示着x.compareTo(z) > 0。

（3）最后，实现者必须确保x.compareTo(y) == 0暗示着所有的z都满足sgn(x.compareTo(z)) == sgn(y.compareTo(z))。

（4）强烈建议(x.compareTo(y) == 0) == (x.equals(y))，但这并非绝对必要。一般来说，任何实现了Comparable接口的类，若违反了这个条件，都应该明确予以说明：该类具有内在的排序功能，但是与equals不一致。

与equals方法不同的是，它对所有的对象强行施加了一种通用的等同关系，compareTo不能跨越不同类型的对象进行比较：在比较不同类型的对象时，compareTo可以抛出ClassCastException异常。通常，这正是compareTo在这种情况下应该做的事情。合约确实允许进行跨类型之间的比较，这一般是在被比较对象实现的接口中进行定义。

就好像违反了hashCode约定的类会破坏其它依赖于散列的类一样，违反compareTo约定的类也会破坏其它依赖于比较关系的类。依赖于比较关系的类包括有序集合TreeSet和TreeMap，以及工具类Collections和Arrays，它们内部包含有搜索和排序算法。

`无法在用新的值组件扩展可实例化的类时，同时保持compareTo约定，除非愿意放弃面向对象的抽象优势。`如果你想为一个实现了Comparable接口的类增加值组件，请不要扩展这个类；而是要编写一个不相关的类，其中包含第一个类的一个实例。然后提供一个“视图”方法返回这个实例。这样既可以让你自由地在第二个类上实现compareTo方法，同时也允许它的客户端在必要的时候，把第二个类的实例视同第一个类的实例。

compareTo方法施加的等同性测试，在通常情况下应该返回与equals方法同样的结果。如果遵守了这一条，那么由compareTo方法所施加的顺序关系就被认为与equals一致。如果违反了这一条，顺序关系就被认为与equals不一致。如果一个类的compareTo方法施加了一个与equals方法不一致的顺序关系，它仍然能够正常工作，但是如果一个有序集合包含了该类的元素，这个集合就可能无法遵守相应集合接口的通用约定。因为这些接口的通用约定是按照equals方法来定义的，但是`有序集合使用了由compareTo方法而不是equals方法所施加的等同性测试`。

例如，以BigDecimal类为例，它的compareTo方法与equals方法不一致。如果你创建了一个空的HashSet实例，并且添加new BigDecimal("1.0")和new BigDecimal("1.00")，这个集合就将包含两个元素，因为新增到集合中的两个BigDecimal实例，通过equals方法来比较时是不相等的。然而，如果你使用TreeSet而不是HashSet来执行同样的过程，集合中将只包含一个元素，因为这两个BigDecimal实例在通过compareTo方法进行比较时是相等的。

编写compareTo方法与编写equals方法非常相似，但也存在几处重大的差别。因为Comparable接口是参数化的，而且compareTo方法是静态的类型，因此不必进行类型检查，也不必对它的参数进行类型转换。如果参数的类型不合适，这个调用甚至无法编译。如果参数为null，这个调用应该抛出NullPointerException异常，并且一旦该方法试图访问它的成员时就应该抛出异常。

compareTo方法中域的比较是顺序的比较，而不是等同性的比较。比较对象引用域可以通过递归地调用compareTo方法来实现。如果一个域并没有实现Comparable接口，或者你需要使用一个非标准的排序关系，就可以使用一个显式的Comparator来代替。

`在Java 7版本中，已经在Java的所有装箱基本类型的类中增加了静态的compare方法`。

如果一个类有多个关键域，那么，按什么样的顺序来比较这些域是非常关键的。你必须从最关键的域开始，逐步进行到所有的重要域。如果某个域的比较产生了非零的结果，则整个比较操作结束，并返回该结果。如果最关键的域是相等的，则进一步比较次关键的域，以此类推。如果所有的域都是相等的，则对象就是相等的，并返回零。

在Java 8中，Comparator接口配置了一组比较器构造方法，使得比较器的构造工作变得非常流畅。之后，按照Comparable接口的要求，这些比较器可以用来实现一个compareTo方法。

```java
private static final Comparator<PhoneNumber> COMPARATOR = Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode).thenComparingInt(pn -> pn.prefix).thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
    return COMPARATOR.compare(this, pn);
}
```

Comparator类具备全套的构造方法。对于基本类型long和double都有对应的comparingInt和thenComparingInt。int版本也可以用于更狭义的整数型类型，如short。double版本也可以用于float。这样便涵盖了所有的Java数字型基本类型。

对象引用类型也有比较器构造方法。静态方法comparing有两个重载。一个带有键提取器，使用键的内在排序关系。第二个既带有键提取器，还带有要用在被提取的键上的比较器。这个名为thenComparing的实例方法有三个重载。一个重载只带一个比较器，并用它提供次级顺序。第二个重载只带一个键提取器，并利用键的内在排序关系作为次级顺序。最后一个重载既带有键提取器，又带有要在被提取的键上使用的比较器。

每当实现一个对排序敏感的类时，都应该让这个类实现Comparable接口，以便其实例可以轻松地被分类、搜索，以及用在基于比较的集合中。每当在compareTo方法的实现中比较阈值时，都要避免使用>和<操作符，而应该在装箱基本类型的类中使用静态的compare方法，或者在Comparator接口中使用比较器构造方法。

# 第15条：使类和成员的可访问性最小化

区分一个组件设计得好不好，唯一重要的因素在于，它对于外部的其他组件而言，是否隐藏了其内部数据和其他实现细节。设计良好的组件会隐藏所有的实现细节，把API与实现清晰地隔离开来。然后，组件之间只通过API进行通信，一个模块不需要知道其他模块的内部工作情况。这个概念被称为`信息隐藏`或`封装`，是软件设计的基本原则之一。

信息隐藏可以有效地解除组成系统的各组件之间的耦合关系，即`解耦`，使得这些组件可以独立地开发、测试、优化、使用、理解和修改。因为这些组件可以并行开发，所以加快了系统开发的速度。同时减轻了维护的负担，程序员可以更快地理解这些组件，并且在调试它们的时候不影响其他的组件。虽然信息隐藏本身无论是对内还是对外都不会带来更好的性能，但是可以有效地调节性能：一旦完成一个系统，并通过剖析确定了哪些组件影响了系统的性能，那些组件就可以被进一步优化，而不会影响到其他组件的正确性。信息隐藏提高了软件的可重用性，因为组件之间并不紧密相连，除了开发这些模块所使用的环境之外，它们在其他的环境中往往也很有用。最后，信息隐藏也降低了构建大型系统的风险，因为即使整个系统不可用，这些独立的组件仍有可能是可用的。

Java提供了许多机制来协助信息隐藏。访问控制机制决定了类、接口和成员的可访问性。实体的可访问性是由该实体声明所在的位置，以及该实体声明中所出现的访问修饰符（private、protected和public）共同决定的。正确地使用这些修饰符对于实现信息隐藏是非常关键的。

`尽可能地使每个类或者成员不被外界访问。`

对于顶层的（非嵌套的）类和接口，只有两种可能的访问级别：包级私有的（package-private）和公有的（public）。如果你用public修饰符声明了顶层类或接口，那它就是公有的；否则，它将是包级私有的。如果类或者接口能够被做成包级私有的，它就应该被做成包级私有。通过把类或者接口做成包级私有，它实际上成了这个包的实现的一部分，而不是该包导出的API的一部分，在以后的发行版本中，可以对它进行修改、替换或者删除，而无须担心会影响到现有的客户端程序。如果把它做成公有的，你就有责任永远支持它，以保持它们的兼容性。

如果一个包级私有的顶层类（或者接口）只是在某一个类的内部被用到，就应该考虑使它成为唯一使用它的那个类的私有嵌套类。这样可以使它的可访问范围从包中的所有类缩小到使用它的那个类。然而，降低不必要的公有类的可访问性，比降低包级私有的顶层类的可访问性重要得多：因为公有类是包的API的一部分，而包级私有的顶层类则已经是这个包的实现的一部分。

对于成员（域、方法、嵌套类和嵌套接口）有四种可能的访问级别，下面按照可访问性的递增顺序罗列出来：

（1）私有的（private）——只有在声明该成员的顶层类内部才可以访问这个成员。

（2）包级私有的（package-private）——声明该成员的包内部的任何类都可以访问这个成员。从技术上讲，它被称为“缺省”访问级别，如果没有为成员指定访问修饰符，就采用这个访问级别（当然，接口成员除外，它们默认的访问级别是公有的）。

（3）受保护的（protected）——声明该成员的类的子类可以访问这个成员，并且声明该成员的包内部的任何类也可以访问这个成员。

（4）公有的（public）——在任何地方都可以访问该成员。

当你仔细地设计了类的公有API之后，可能觉得应该把所有其他的成员都变成私有的。其实，只有当同一个包内的另一个类真正需要访问一个成员的时候，你才应该删除private修饰符，使该成员变成包级私有的。如果你发现自己经常要做这样的事，就应该重新检查系统设计，看看是否另一种分解方案得到的类，与其他类之间的耦合度会更小。私有成员和包级私有成员都是一个类的实现中的一部分，一般不会影响导出的API。然而，如果这个类实现了Serializable接口，这些域就有可能会被“泄漏”到导出的API中。

对于公有类的成员，当访问级别从包级私有变成保护级别时，会大大增强可访问性。受保护的成员是类的导出的API的一部分，必须永远得到支持。导出的类的受保护成员也代表了该类对于某个实现细节的公开承诺。应该尽量少用受保护的类。

有一条规则限制了降低了方法的可访问性的能力。`如果方法覆盖了一个超类中的一个方法，子类中的访问级别就不允许低于超类中的访问级别`。这样可以确保任何可使用超类的实例的地方也都可以使用子类的实例（里式替换原则）。如果违反了这条规则，那么当你试图编译该子类的时候，编译器就会产生一条错误信息。这条规则有一个特例：如果一个类实现了一个接口，那么接口中所有的方法在这个类中也都必须被声明为公有的。

为了便于测试代码，你可以试着使类、接口或者成员变得更容易访问。这么做在一定程度上来说是好的。为了测试而将一个公有类的私有成员变量变成包级私有的，这还可以接受，但是要将访问级别提高到超过它，这就无法接受了。换句话说，不能为了测试，而将类、接口或者成员变成包的导出的API的一部分。幸运的是，也没有必要这么做，因为可以让测试作为被测试的包的一部分来运行，从而能够访问它的包级私有的元素。

`公有类的实例域绝不能是公有的。`如果实例域是非final的，或者是一个指向可变对象的final引用，那么一旦这个域成为公有的，就等于放弃了对存储在这个域中的值进行限制的能力；这意味着，你也放弃了强制这个域不可变的能力。同时，当这个域被修改的时候，你也失去了对它采取任何行动的能力。因此，`包含公有可变域的类通常并不是线程安全的。`即使域是final的，并且引用不可变的对象，但当把这个域变成公有的时候，也就放弃了“切换到一种新的内部数据表示法”的灵活性。

这条建议也同样适用于静态域，只有一种情况例外。假设常量构成了类提供的整个抽象中的一部分，可以通过公有的静态final域来暴露这些常量。按惯例，这种域的名称由大写字母组成，单词之间用下划线隔开。很重要的一点是，这些域要么包含基本类型的值，要么包含指向不可变对象的引用。如果final域包含可变对象的引用，它便具有非final域的所有缺点。虽然引用本身不能被修改，但是它所引用的对象却可以被修改，这会导致灾难性的后果。

注意，`长度非零的数组总是可变的`，所以`让类具有公有的静态final数组域，或者返回这种域的访问方法，这是错误的`。如果类具有这样的域或者访问方法，客户端将能够修改数组中的内容。这是安全漏洞的一个常见根源：

```java
public static final Thing[] value = {...};
```

许多IDE产生的访问方法会返回指向私有数组域的引用，正好导致了这个问题。修正这个问题有两种方法。可以使公有数组变成私有的，并增加一个公有的不可变列表：

```java
private static final Thing[] PRIVATE_VALUES = {...};

public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

另一种方法是，也可以使数组变成私有的，并添加一个公有方法，它返回私有数组的一个浅拷贝：

```java
private static final Thing[] PRIVATE_VALUES = {...};

public static final Thing[] values() {
    return PRIVATE_VALUES.clone();
}
```

从Java 9开始，又新增了两种隐式访问级别，作为模块系统的一部分。一个模块就是一组包，就像一个包就是一组类一样。模块可以通过其模块声明中的导出声明显式地导出它的一部分包（按照惯例，这包含在名为module-info.java的源文件中）。模块中未被导出的包在模块之外时不可访问的。在模块内部，可访问性不受导出声明的影响。使用模块系统可以在模块内部的包之间共享类，不用让它们对全世界都可见。未导出的包中公有类的公有成员和受保护的成员都提高了两个隐式访问等级，这是正常的公有和受保护级别在模块内部的对等体。对于这种共享的需求相对罕见，经常通过在包内部重新安排类来解决。

与四个主访问级别不同，这两个基于模块的级别主要提供咨询。如果把模块的JAR文件放在应用程序的类路径下，而不是放在模块路径下，模块中的包就会恢复其非模块的行为：无论包是否通过模块导出，这些包中公有类中的所有公有的和受保护的成员将都有正常的可访问性。严格执行新引入的访问级别的一个示例是JDK本身：Java类库中未导出的包在其模块之外确实是不可访问的。

应该尽可能地降低程序元素的可访问性。在仔细地设计了一个最小的公有API之后，应该防止把任何散乱的类、接口或者成员变成API的一部分。除了公有静态final域的特殊情形之外（此时它们充当常量），公有类都不应该包含公有域，而且要确保公有静态final域所引用的对象都是不可变的。

# 第16条：要在公有类而非公有域中使用访问方法

有时候，可能需要编写一些只是用来集中实例域的类：

```java
class Point {
    public double x;

    public double y;
}
```

由于这种类的数据域是可用被直接访问的，这些类没有提供封装的功能。如果不改变API，就无法改变它的数据表示法，也无法强加任何约束条件：当域被访问的时候，无法采取任何辅助的行动。坚持面向对象编程的程序员对这种类深恶痛绝，认为应该用包含私有域和公有访问方法的类代替。对于可变的类来说，应该用公有设值方法的类代替：

```java
class Point {
    private double x;

    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }

    public void setX(double x) {
        this.x = x;
    }

    public void setY(double y) {
        this.y = y;
    }
}
```

毫无疑问，说到公有类的时候，坚持面向对象编程思想的看法是正确的：如果类可以在它所在的包之外进行访问，就提供访问方法，以保留将来改变该类的内部表示法的灵活性。如果公有类暴露了它的数据域，要想在将来改变去其内部表示法是不可能的，因为公有类的客户端代码已经遍布各处了。

然而，如果类是包级私有的，或者是私有的嵌套类，直接暴露它的数据域并没有本质的错误——假设这些数据域确实描述了该类所提供的抽象。无论是在类定义中，还是在使用该类的客户端代码中，这种方法比访问方法的做法更不容易产生视觉混乱。虽然客户端代码与该类的内部表示法紧密相连，但是这些代码被限定在包含该类的包中。如有必要，也可以不改变包之外的任何代码，而只改变内部数据表示法。在私有嵌套类的情况下，改变的作用范围被进一步限制在外围类中。

Java平台类库中有几个类违反了“公有类不应该直接暴露数据域”的告诫，如java.awt包中的Point类和Dimension类，它们是反面例子。

让公有类直接暴露域虽然从来都不是种好办法，但是如果域是不可变的，这种做法的危害就比较小一些。如果不改变类的API，就无法改变这种类的表示法，当域被读取的时候，你也无法采取辅助的行动，但是可以强加约束条件。例如，这个类确保了每个实例都表示一个有效的时间：

```java
public final class Time {
    private static final int HOURS_PER_DAY = 24;

    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;

    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY) {
            throw new IllegalArgumentException("Hour: " + hour);
        }
        if (minute < 0 || minute >= MINUTES_PER_HOUR) {
            throw new IllegalArgumentException("Min: " + minute);
        }
        this.hour = hour;
        this.minute = minute;
    }
}
```

简而言之，公有类永远都不应该暴露可变的域。虽然还是有问题，但是让公有类暴露不可变的域，其危害相对来说比较小。但有时候会需要用包级私有的或者私有的嵌套类来暴露域，无论这个类是可变的还是不可变的。

# 第17条：使可变性最小化

不可变类是指其实例不能被修改的类。每个实例中包含的所有信息都必须在创建该实例的时候就提供，并在对象的整个生命周期内固定不变。Java平台类库中包含许多不可变的类，其中有String、基本类型的包装类、BigInteger和BigDecimal。存在不可变的类有许多理由：不可变的类比可变类更加易于设计、实现和使用。它们不容易出错，且更加安全。

为了使类成为不可变，要遵循下面五条规则：

（1）`不要提供任何会修改对象状态的方法（设值方法）。`

（2）`保证类不会被扩展。`这样可以防止粗心或者恶意的子类假装对象的状态已经改变，从而破坏该类的不可变行为。为了防止子类化，一般做法是声明这个类成为final的。

（3）`声明所有的域都是final的。`通过系统的强制方式可以清楚地表明你的意图。而且，如果一个指向新创建实例的引用在缺乏同步机制的情况下，从一个线程被传递到另一个线程，就必须确保正确的行为，正如内存模型中所述。

（4）`声明所有的域都为私有的。`这样可以防止客户端获得被域引用的可变对象的权限，并防止客户端直接修改这些对象。虽然从技术上讲，允许不可变的类具有公有的final域，只要这些域包含基本类型的值或者指向不可变对象的引用，但是不建议这样做，因为这样会使得在以后的版本中无法再改变内部的表示法。

（5）`确保对于任何可变组件的互斥访问。`如果类具有指向可变对象的域，则必须确保该类的客户端无法获得指向这些对象的引用。并且，永远不要用客户端提供的对象引用来初始化这样的域，也不要从任何方法中返回该对象引用。在构造器、访问方法和readObject方法中请使用保护性拷贝技术。

```java
public final class Complex {
    private final double re;

    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() {
        return re;
    }

    public double imaginaryPart() {
        return im;
    }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im, re * c.im + im * c.re);
    }

    public Complex divideBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp, (im * c.re - re * c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }
        if (!(o instanceof Complex)) {
            return false;
        }
        Complex c = (Complex) o;
        return Double.compare(c.re, re) == 0 && Double.compare(c.im, im) == 0;
    }

    @Override
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override
    public String toString() {
        return "(" + re + " + " + im + "i)";
    }
}
```

这个类表示一个复数。除了标准的Object方法之外，它还提供了针对实部和虚部的访问方法，以及4种基本的算术运算：加法、减法、乘法和除法。注意这些算术运算如何创建并返回新的Complex实例，而不是修改这个实例。大多数重要的不可变类都使用了这种模式。它被称为函数的方法，因为这些方法返回了一个函数的结果，这些函数对操作数进行运算但并不修改它。与之相对应更常见的是过程的或者命令使的方法，使用这些方法时，将一个过程作用在它们的操作数上，会导致它的状态发生改变。注意，这些方法名称都是介词（如plus），而不是动词（如add）。这是为了强调该方法不会改变对象的值。

`不可变对象比较简单。`不可变对象可以只有一种状态，即被创建时的状态。如果你能够确保所有的构造器都建立了这个类的约束关系，就可以确保这些约束关系在整个生命周期内永远不再发生变化，你和使用这个类的程序员都无须再做额外的工作来维护这些约束关系。另一方面，可变的对象可以有任意复杂的状态空间。如果文档中没有为设值方法所执行的状态转换提供精确的描述，要可靠地使用可变类是非常困难的，甚至是不可能的。

`不可变对象本质上是线程安全的，它们不要求同步。`当多个线程并发访问这样的对象时，它们不会遭到破坏。这无疑是获得线程安全最容易的办法。实际上，没有任何线程会注意到其他线程对于不可变对象的影响。所以，`不可变对象可以被自由地共享。`不可变类应该充分利用这种优势，鼓励客户端尽可能地重用现有的实例。要做到这一点，一个很简便的办法就是：对于频繁用到的值，为它们提供公有的静态final常量。例如，Complex类有可能会提供下面的常量：

```java
public static final Complex ZERO = new Complex(0, 0);

public static final Complex ONE = new Complex(1, 0);

public static final Complex I = new Complex(0, 1);
```

这种方法可以被进一步扩展。不可变的类可以提供一些静态工厂，它们把频繁被请求的实例缓存起来，从而当现有实例可以符合请求的时候，就不必创建新的实例。所有基本类型的包装类和BigInteger都有这些的静态工厂。使用这样的静态工厂也使得客户端之间可以共享现有的实例，而不用创建新的实例，从而降低内存占用和垃圾回收的成本。在设计新的类时，选择用静态工厂代替公有的构造器可以让你以后有添加缓存的灵活性，而不必影响客户端。

“不可变对象可以被自由地共享”导致的结果是，永远也不需要进行保护性拷贝。实际上，你根本无须做任何拷贝，因为这些拷贝始终等于原始的对象。因此，你不需要，也不应该为不可变的类提供clone方法或者拷贝构造器。这一点在Java平台的早期并不好理解，所以String类仍然具有拷贝构造器，但是应该尽量少用它。

`不仅可以共享不可变对象，甚至也可以共享它们的内部信息。`例如，BigInteger类内部使用了符号数值表示法。符号用一个int类型的值来表示，数值则用一个int数组表示。negate方法产生一个新的BigInteger，其中数值是一样的，符号则是相反的。它并不需要拷贝数组，新建的BigInteger也指向原始实例中的同一个内部数组。

`不可变对象为其他对象提供了大量的构件`，无论是可变的还是不可变的对象。如果知道一个复杂对象内部的组件对象不会改变，要维护它的不变性约束是比较容易的。这条原则的一种特例在于，不可变对象构成了大量的映射键和集合元素；一旦不可变对象进入到映射或者集合中，尽管这破坏了映射或者集合的不变性约束，但是也不用担心它们的值会发生变化。

`不可变对象无偿地提供了失败的原子性。`它们的状态永远不变，因此不存在临时不一致的可能性。

`不可变类真正唯一的缺点是，对于每个不同的值都需要一个单独的对象。`创建这些对象的代价可能很高，特别是大型的对象。例如，假设你有一个上百万位的BigInteger，想要改变它的低位：

```java
BigInteger moby = ...;
moby = moby.flipBit(0);
```

flipBit方法创建了一个新的BigInteger实例，也有上百万位长，它与原来的对象只差一位不同。这项操作所消耗的时间和空间与BigInteger的长度成正比。我们拿它与java.util.BitSet进行比较。与BigInteger类似，BitSet代表一个任意长度的位序列，但是与BigInteger不同的是，BitSet是可变的。BitSet提供了一个方法，允许在固定时间内改变此“百万位”实例中单个位的状态：

```java
BitSet moby = ...;
moby.flip(0);
```

如果你执行一个多步骤的操作，并且每个步骤都会产生一个新的对象，除了最后的结果之外，其他的对象最终都会被丢弃，此时性能问题就会显露出来。处理这种问题有两种办法。第一种办法，先猜测一下经常会用到哪些多步骤的操作，然后将它们作为基本类型提供。如果某个多步骤操作已经作为基本类型提供，不可变的类就无须在每个步骤单独创建一个对象。不可变的类在内部可以更加灵活。例如，BigInteger有一个包级私有的可变“配套类”，它的用途是加速诸如“模指数”这样的多步骤操作。

如果能够精确地预测出客户端将要在不可变的类上执行哪些复杂的多阶段操作，这种包级私有的可变配套类的方法就可以工作得很好。如果无法预测，最好的办法是提供一个公有的可变配套类。在Java平台类库中，这种方法的主要例子是String类，它的可变配套类是StringBuilder（及其已经被废弃的祖先StringBuffer）。

为了确保不可变性，类绝对不允许自身被子类化。除了”使类成为final的”这种方法之外，还有一种更加灵活的方法可以做到这一点。不可变的类变成final的另一种办法就是，让类的所有构造器都变成私有的或者包级私有的，并添加公有的静态工厂来代替公有的构造器。为了具体说明这种方法，下面以Complex为例：

```java
public class Complex {
    private final double re;

    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
}
```

这种方法虽然并不常用，但它通常是最好的替代方法。它最灵活，因为它允许使用多个包级私有的实现类。对于处在包外部的客户端而言，不可变的类实际上是final的，因为不可能对来自另一个包的类，缺少公有的或受保护的构造器的类进行扩展。除了允许多个实现类的灵活性之外，这种方法还使得有可能通过改善静态工厂的对象缓存能力，在后续的发行版本中改进该类的性能。

当BigInteger和BigDecimal刚被编写出来的时候，对于“不可变的类必须为final”的说法还没有得到广泛的理解，所以它们的所有方法都有可能被覆盖。遗憾的是，为了保持向后兼容，这个问题一直无法得以修正。如果你在编写一个类，它的安全性依赖于来自不可信客户端的BigInteger或者BigDecimal参数的不可变性，就必须进行检查，以确定这个参数是否为“真正的”BigInteegr或者BigDecimal，而不是不可信任子类的实例。如果是后者，就必须在假设它可能是可变的前提下对它进行保护性拷贝：

```java
public static BigInteger safeInstance(BigInteger val) {
    return val.getClass() == BigInteger.class ? val : new BigInteger(val.toByteArray());
}
```

本条开头关于不可变类的诸多规则指出，没有方法会修改对象，并且它的所有域都必须是final的。实际上，这些规则比真正的要求更强硬了一点，为了提高性能可以有所放松。事实上应该这样：没有一个方法能够对对象的状态产生外部可见的改变。然而，许多不可变的类拥有一个或者多个非final的域，它们在第一次被请求执行这些计算的时候，把一些开销昂贵的计算结果缓存在这些域中。如果将来再次请求同样的计算，就直接返回这些缓存的值，从而节约了重新计算所需要的开销。这种技巧可以很好地工作，因为对象是不可变的，它的不可变性保证了这些计算如果被再次执行，就会产生同样的结果。

例如，PhoneNumber类的hashCode方法在第一次被调用的时候，计算出散列码，然后把它缓存起来，以备将来被再次调用时使用。这种方法是延迟初始化的一个例子，String类也用到了。

如果你选择让自己的不可变类实现Serializable接口，并且它包含一个或者多个指向可变对象的域，就必须提供一个显式的readObject或者readResolve方法，或者使用ObjectOutputStream.writeUnshared和ObjectInputStream.readUnshared方法，即便默认的序列化形式是可以接受的，也是如此。否则，攻击者可能从不可变的类创建可变的实例。

总之，坚决不要为每个get方法编写一个相应的set方法。`除非有很好的理由要让类成为可变的类，否则它就应该是不可变的`。不可变的类有许多优点，唯一的缺点是在特定的情况下存在潜在的性能问题。你应该总是使一些小的值对象，比如PhoneNumber和Complex成为不可变的。（在Java平台类库中，有几个类如java.util.Date和java.awt.Point，它们本应该是不可变的，但实际上却不是。）你也应该认真考虑把一些较大的值对象做成不可变的，例如String和BigInteger。只有当你确认有必要实现令人满意的性能时，才应该为不可变的类提供公有的可变配套类。

对于某些类而言，其不可变性是不切实际的。`如果类不能被做成不可变的，仍然应该尽可能地限制它的可变性。`降低对象可以存在的状态数，可以更容易地分析该对象的行为，同时降低出错的可能性。因此，除非有令人信服的理由使域变成非final的，否则让每个域都是final的。

`构造器应该创建完全初始化的对象，并建立起所有的约束关系。`不要在构造器或者静态工厂之外再提供公有的初始化方法，除非有令人信服的理由必须这么做。同样地，也不应该提供“重新初始化”方法（它使得对象可以被重用，就好像这个对象是由另一不同的初始状态构造出来一样）。与增加的复杂性相比，”重新初始化”方法通常并没有带来太多的性能优势。

通过CountDownLatch类的例子可以说明这些原则。它是可变的，但是它的状态空间被有意地设计得非常小。比如创建一个实例，只使用一次，它的任务就完成了：一旦定时器的计数达到零，就不能重用了。

# 第18条：复合优先于继承

继承时实现代码重用的有力手段，但它并非永远是完成这项工作的最佳工具。使用不当会导致软件变得脆弱。在包的内部使用继承是非常安全的，在那里子类和超类的实现都处在同一个程序员的控制之下。对于专门为了继承而设计并且具有很好的文档说明的类来说，使用继承也是非常安全的。然而，对普通的具体类进行跨越包边界的继承，则是非常危险的。

`与方法调用不同的是，继承打破了封装性。`换句话说，子类依赖于其超类中特定功能的实现细节。超类的实现有可能会随着发行版本的不同而有所变化，如果真的发生了变化，子类可能会遭到破坏，即使它的代码完全没有改变。因而，子类必须要跟着其超类的更新而演变，除非超类是专门为了扩展而设计的，并且具有很好的文档说明。

为了说明得更加具体一点，我们假设有一个程序使用了HashSet。为了调优该程序的性能，需要查询HashSet，看一看自从它被创建以来添加了多少个元素（不要与它当前的元素数目混淆起来，它会随着元素的删除而递减）。为了提供这种功能，我们得编写一个HashSet变体，定义记录试图插入的元素的数量addCount，并针对该计数值导出一个方位方法。HashSet类包含两个可以增加元素的方法：add和addAll，因此这两个方法都要被覆盖：

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    public InstrumentedHashSet() {}

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

这个类看起来非常合理，但是它并不能正常工作。假设我们创建了一个实例，并利用addAll方法添加了三个元素。顺便提一句，注意我们利用静态工厂方法List.of创建了一个列表，该方法是在Java 9中增加的。如果使用较早的版本，则用Arrays.asList代替：

```java
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("Snap", "Crackle", "Pop"));
```

此时我们期望getAddCount方法能返回3，但是实际上它返回的是6。哪里出错了呢？在HashSet的内部，addAll方法是基于它的add方法来实现的，即使HashSet的文档中并没有说明这样的实现细节，这也是合理的。InstrumentedHashSet中的addAll方法首先给addCount增加3，然后利用super.addAll来调用HashSet的addAll实现。然后又依次调用到被InstrumentedHashSet覆盖了的add方法，每个元素调用一次。这三次调用又分别给addCount加了1，所以总共增加了6：通过addAll方法增加的每个元素都被计算了两次。

我们只要去掉被覆盖的addAll方法，就可以“修正”这个子类。虽然这样得到的类可以正常工作，但是它的功能正确性则需要依赖于这样的事实：HashSet的addAll方法是在它的add方法上实现的。这种“自用性”是实现细节，不是承诺，不能保证在Java平台的所有实现中都保持不变，不能保证随着发行版本的不同而不发生变化。因此，这样得到的InstrumentedHashSet类将是非常脆弱的。

稍微好一点的做法是，覆盖addAll方法来遍历指定的集合，为每个元素调用一次add方法。这样做科研保证得到正确的结果，不管HashSet的addAll方法是否在add方法的基础上实现，因为HashSet的addAll实现将不会再被调用到。然而，这项技术并没有解决所有的问题，它相当于重新实现了超类的方法，这些超类的方法可能是自用的，也可能不是，这种方法很困难，也非常耗时，容易出错，并且可能降低性能。此外，这样做并不总是可行的，因为无法访问对于子类来说是私有的域，所以有些方法就无法实现。

导致子类脆弱的一个相关的原因是，它们的超类在后续的发行版本中可以获得新的方法。假设一个程序的安全性依赖于这样的事实：所有被插入某个集合中的元素都满足某个先决条件。下面的做法就可以确保这一点：对集合进行子类化，并覆盖所有能够添加元素的方法，以便确保在加入每个元素之前它是满足这个先决条件的。如果在后续的发行版本中，超类中没有增加能插入元素的新方法，这种做法就可以正常工作。然而，一旦超类增加了这样的方法，则很可能仅仅由于调用了这个未被子类覆盖的新方法，而将“非法的”元素添加到子类的实例中。这不是一个纯粹的理论问题。在把Hashtable和Vector加入到Collections Framework中的时候，就修正了几个这类性质的安全漏洞。

上面这两个问题都来源于覆盖方法。你可能会认为在扩展一个类的时候，仅仅增加新的方法，而不覆盖现有的方法是安全的。虽然这种扩展方式比较安全一些，但是也并非完全没有风险。如果超类在后续的发行版本中获得了一个新的方法，并且不幸的是，你给子类提供了一个签名相同但返回类型不同的方法，那么这样的子类将无法通过编译。如果给子类提供的方法带有与新的超类方法完全相同的签名和返回类型，实际上就覆盖了超类中的方法，因此又回到了上述两个问题。此外，你的方法是否能够遵守新的超类方法的约定，这也是很值得怀疑的，因为当你在编写子类方法的时候，这个约定压根还没有面世。

幸运的是，有一种办法可以避免前面提到的所有问题。即不扩展现有的类，而是在新的类中增加一个私有域，它引用现有类的一个实例。这种设计被称为“复合”，因为现有的类变成了新类的一个组件。新类中的每个实例方法都可以调用被包含的现有类实例中对应的方法，并返回它的结果。这被称为转发，新类中的方法被称为转发方法。这样得到的类将会非常稳固，它不依赖于现有类的实现细节。即使现有的类添加了新的方法，也不会影响新的类。为了进行更具体的说明，情况下面的例子，它用复合/转发的方法来代替InstrumentedHashSet类。注意这个实现分为两部分：类本身和可重用的转发类，其中包含了所有的转发方法，没有任何其他的方法：

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
   }

   public int getAddCount() {
     return addCount;
   }
}

public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;

    public ForwardingSet(Set<E> s) {
        this.s = s;
    }

    public void clear() {
        s.clear();
    }

    public boolean contains(Object o) {
        return s.contains(o);
    }

    public boolean isEmpty() {
        return s.isEmpty();
    }

    public int size() {
        return s.size();
    }

    public Iterator<E> iterator() {
        return s.iterator();
    }

    public boolean add(E e) {
        return s.add(e);
    }

    public boolean remove(Object o) {
        return s.remove(o);
    }

    public boolean containsAll(Collection<?> c) {
        return s.containsAll(c);
    }

    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }

    public boolean removeAll(Collection<?> c) {
        return s.removeAll(c);
    }

    public boolean retainAll(Collection<?> c) {
        return s.retainAll(c);
    }

    public Object[] toArray() {
        return s.toArray();
    }

    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }

    @Override
    public boolean equals(Object o) {
        return s.equals(o);
    }

    @Override
    public int hashCode() {
        return s.hashCode();
    }

    @Override
    public String toString() {
        return s.toString();
    }
}
```

Set接口的存在使得InstrumentedSet类的设计成为可能，因为Set接口保存了HashSet类的功能特性。除了获得健壮性之外，这种设计也带来了更多的灵活性。InstrumentedSet类实现了Set接口，并且拥有单个构造器，它的参数也是Set类型。从本质上讲，这个类把一个Set转变成了另一个Set，同时增加了计数的功能。前面提到的基于继承的方法只适用于单个具体的类，并且对于超类中所支持的每个构造器都要求有一个单独的构造器，与此不同的是，这里的包装类可以被用来包装任何Set实现，并且可以结合任何先前存在的构造器一起工作。

因为每一个InstrumentedSet实例都把另一个Set实例包装起来了，所以InstrumentedSet类被称为包装类。这也正是装饰者模式，因为InstrumentedSet类对一个集合进行了修饰，为它增加了计数特性。有时复合和转发的结合也被宽松地称为“委托”。从技术的角度而言，这不是委托，除非包装对象把自身传递给被包装的对象。

包装类几乎没有什么缺点。需要注意的一点是，包装类不适合用于回调框架；在回调框架中，对象把自身的引用传递给其他的对象，用于后续的调用（”回调”）。因为被包装起来的对象并不知道它外面的包装对象，所以它传递一个指向自身的引用（this），回调时避开了外面的包装对象。这被称为SELF问题。有些人担心转发方法调用所带来的性能影响，或者包装对象导致的内存占用。在实践中，这两者都不会造成很大的影响。编写转发方法倒是有点琐碎，但是只需要给每个接口编写一次构造器，转发类则可以通过包含接口的包提供。例如，Guava就为所有的集合接口提供了转发类。

只有当子类真正是超类的子类型时，才适合用继承。换句话说，对于两个类A和B，只有当两者之间确实存在“is-a”关系的时候，类B才应该扩展类A。如果你打算让类B扩展类A，就应该问问自己：每个B确实也是A吗？如果你不能够确定这个问题的答案是肯定的，那么B就不应该扩展A。如果答案是否定的，通常情况下，B应该包含A的一个私有实例，并且暴露一个较小的、较简单的API：A本质上不是B的一部分，只是它的实现细节而已。

在Java平台类库中，有许多明显违反这条原则的地方。例如，栈并不是向量，所以Stack不应该扩展Vector。同样地，属性列表也不是散列表，所以Properties不应该扩展Hashtable。在这两种情况下，复合模式才是恰当的。

如果在适合使用复合的地方使用了继承，则会不必要地暴露实现细节。这样得到的API会把你限制在原始的实现上，永远限定了类的性能。更为严重的是，由于暴露了内部的细节，客户端就有可能直接访问这些内部细节。这样至少会导致语义上的混淆。例如，如果P指向Properties实例，那么p.getProperty(key)就有可能产生与p.get(key)不同的结果：前一个方法考虑了默认的属性表，而后一个方法则继承自Hashtable，没有考虑默认的属性列表。最严重的是，客户有可能直接修改超类，从而破坏子类的约束条件。在Properties的情形中，设计者的目标是只允许字符串作为键和值，但是直接访问底层的Hashtable就允许违反这种约束条件。一旦违反了约束条件，就不可能再使用Properties API的其他部分（load和store）了。等到发现这个问题时，要改正它已经太晚了，因为客户端依赖于使用非字符串的键和值了。

在决定使用继承而不是复合之前，还应该问自己最后一组问题。对于你正试图扩展的类，它的API中有没有缺陷呢？如果有，你是否愿意把那些缺陷传播到类的API中？继承机制会把超类API中的所有缺陷传播到子类中，而复合则允许设计新的API来隐藏这些缺陷。

简而言之，继承的功能非常强大，但是也存在诸多问题，因为它违背了封装原则。只有当子类和超类之间确实存在子类型关系时，使用继承才是恰当的。即便如此，如果子类和超类处在不同的包中，并且超类并不是为了继承而设计的，那么继承将会导致脆弱性。为了避免这种脆弱性，可以用复合和转发机制来代替继承，尤其是当存在适当的接口可以实现包装类的时候。包装类不仅比子类更加健壮，而且功能也更加强大。

# 第19条：要么设计继承并提供文档说明，要么禁止继承

首先，该类的文档必须精确地描述覆盖每个方法所带来的影响。换句话说，`该类必须有文档说明它可覆盖的方法的自用性。`对于每个公有的或受保护的方法或者构造器，它的文档必须指明该方法或者构造器调用了哪些可覆盖的方法，是以什么顺序调用的，每个调用的结果又是如何影响后续处理过程的（所谓可覆盖的方法，是指非final的、公有的或受保护的）。更广义地说，即类必须在文档中说明，在哪些情况下它会调用可覆盖的方法。

如果方法调用到了可覆盖的方法，在它的文档注释的末尾应该包含关于这些调用的描述信息。

`对于为了继承而设计的类，唯一的测试方法就是编写子类。`

为了允许继承，类还必须遵守其他一些约束。`构造器决不能调用可被覆盖的方法，`无论是直接调用还是间接调用。如果违反了这条规则，很有可能导致程序失败。超类的构造器在子类的构造器之前运行，所以，子类中覆盖版本的方法将会在子类的构造器运行之前先被调用。如果该覆盖版本的方法依赖于子类构造器所执行的任何初始化工作，该方法将不会如预期般执行。为了更加直观地说明这一点，下面举个例子，其中有个类违反了这条规则：

```java
public class Super {
    public Super() {
        overrideMe();
    }

    public void overrideMe() {}
}
```

下面的子类覆盖了方法overrideMe，Super唯一的构造器就错误地调用了这个方法：

```java
public final class Sub extends Super {
    private final Instant instant;

    Sub() {
        instant = Instant.now();
    }

    @Override
    public void overrideMe() {
        System.out.println(instant);
    }

    public static void main(String[] args) {
        Sub sub = new Sub();
        sub.overrideMe();
    }
}
```

你可能会期待这个程序会打印两次日期，但是它第一次打印出的是null，因为overrideMe方法被Super构造器调用的时候，构造器Sub还没有机会初始化instant域。注意，这个程序观察到的final域处于两种不同的状态。还要主义，如果overrideMe已经调用了instant中的任何方法，当Super构造器调用overrideMe的时候，调用就会抛出NullPointerException异常。如果该程序没有抛出NullPointerException异常，唯一的原因就在于println方法可以容忍null参数。

注意，通过构造器调用私有的方法、final方法和静态方法是安全的，这些都不是可以被覆盖的方法。

在为了继承而设计类的时候，Cloneable和Serializable接口出现了特殊的困难。如果类是为了继承而设计的，无论实现这其中的哪个接口通常都不是个好主意，因为它们把一些实质性的负担转嫁到了扩展这个类的程序员身上。然而，你还是可以采取一些特殊的手段，允许子类实现这些接口，无须强迫子类的程序员去承受这些负担。

如果你决定在一个为了继承而设计的类中实现Cloneable或者Serializable接口，就应该意识到，因为clone和readObject方法在行为上非常类似于构造器，所以类似的限制规则也是适用的：无论是clone还是readObject，都不可以调用可覆盖的方法，不管是以直接还是间接的方式。对于readObject方法，覆盖的方法将在子类的状态被反序列化之前先被允许；而对于clone方法，覆盖的方法则是在子类的clone方法有机会修正被克隆对象的状态之前先被运行。无论哪种情形，都不可避免地将导致程序失败。在clone方法的情形中，这种失败可能会同时损害到原始的对象以及被克隆的对象本身。例如，如果覆盖版本的方法假设它正在修改对象深层结构的克隆对象的备份，就会发生这种情况，但是该备份还没有完成。

最后，如果你决定在一个为了继承而设计的类中实现Serializable接口，并且该类有一个readResolve或者writeReplace方法，就必须使readResolve或者writeReplace成为受保护的方法，而不是私有的方法。如果这些方法是私有的，那么子类将会不声不响地忽略掉这两个方法。这正是“为了允许继承，而把实现细节变成一个类的API的一部分”的另一种情形。

`为了继承而设计类，对这个类会有一些实质性的限制。`这并不是很轻松就可以承诺的决定。在某些情况下，这样的决定很明显是正确的，比如抽象类，包括接口的骨架实现。但是，在另外一些情况下，这样的决定却很明显是错误的，比如不可变类。

但是，对于普通的具体类应该怎么办呢？它们既不是final的，也不是为了子类化而设计和编写文档的，所以这种状况很危险。每次对这种类进行修改，从这个类扩展得到的客户类就有可能遭到破坏。这不仅仅是个理论问题。对于一个并非为了继承而设计的非final具体类，在修改了它的内部实现之后，接收到与子类化相关的错误报告也并不少见。

`这个问题的最佳解决方案是，对于那些并非为了安全地进行子类化而设计和编写文档的类，要禁止子类化。`有两种办法可以禁止子类化。比较容易的办法是把这个类声明为final的。另一种办法是把所有的构造器都变成私有的，或者包级私有的，并增加一些公有的静态工厂来替代构造器。

# 第20条：接口优于抽象类

Java提供了两种机制，可以用来定义允许多个实现的类型：接口和抽象类。自从Java 8为继承引入了default方法，这两种机制都允许为某些实例方法提供实现。主要的区别在于，为了实现由抽象类定义的类型，类必须成为抽象类的一个子类。因为Java只允许单继承，所以用抽象类作为类型定义受到了限制。任何定义了所有必要的方法并遵守通用约定的类，都允许实现一个接口，无论这个类是处在类层次结构中的什么位置。

`现有的类可以很容易被更新，以实现新的接口。`如果这些方法尚不存在，你所需要做的就只是增加必要的方法，然后在类的声明中增加一个implements子句。例如，当Comparable、Iterable和Autocloseable接口被引入Java平台时，更新了许多现有的类，以实现这些接口。一般来说，无法更新现有的类来扩展新的抽象类。如果你希望两个类扩展同一个抽象类，就必须把抽象类放到类型层次的高处，这样它就成了那两个类的一个祖先。遗憾的是，这样做会间接地伤害到类层次，迫使这个公共祖先的所有后代类都扩展这个新的抽象类，无论它对于这些后代类是否合适。

`接口是定义mixin（混合类型）的理想选择。`mixin类型是指：类除了实现它的“基本类型”之外，还可以实现这个mixin类型，以表明它提供了某些可供选择的行为。例如，Comparable是一个mixin接口，它允许类表明它的实例可以与其他的可相互比较的对象进行排序。这样的接口之所以被称为mixin，是因为它允许任选的功能可被混合到类型的主要功能中。抽象类不能被用于定义mixin，同样也是因为它们不能被更新到现有的类中：类不可能有一个以上的父类，类层次结构中叶没有适当的地方来插入mixin。

`接口允许构造非层次结构的类型框架。`类型层次对于组织某些事物是非常合适的，但是其他事物并不能被整齐地组织成一个严格的层次结构。例如，假设我们有一个接口代表一个singer（歌唱家），另一个接口代表一个songwriter（作曲家）：

```java
public interface Singer {
    SutoClip sing(Song s);
}

public interface Songwriter {
    Song compose(int chartPosition);
}
```

在现实生活中，有些歌唱家本身也是作曲家。因为我们使用了接口而不是抽象类来定义这些类型，所以对于单个类而言，它同时实现Singer和Songwriter是完全允许的。实际上，我们可以定义第三个接口，它同时扩展Singer和Songwriter，并添加一些适合于这种组合的新方法：

```java
public interface SingerSongwriter extends Singer, Songwriter {
    AudioClip strum();

    void actSensitive();
}
```

也许并非总是需要这种灵活性，但是一旦这样做了，接口可就成了救世主。另外一种做法是编写一个臃肿的类层次，对于每一种要被支持的属性组合，都包含一个单独的类。如果在整个类型系统中有n个属性，那么就必须支持2 ^ n种可能的组合。这种现象被称为“组合爆炸”。类层次臃肿会导致类也臃肿，这些类包含许多方法，并且这些方法只是在参数的类型上有所不同而已，因为类层次中没有任何类型体现了公共的行为特征。

通过第18条中介绍的包装类模式，`接口使得安全地增强类的功能成为可能。`如果使用抽象类来定义类型，那么程序员除了使用继承的手段来增加功能，再没有其他的选择了。这样得到的类与包装类相比，功能更差，也更加脆弱。

当一个接口方法根据其他接口方法有了明显的实现时，可以考虑以default方法的形式为程序员提供实现协助。

通过default方法可以提供的实现协助是有限的。虽然许多接口都定义了Object方法的行为，如equals和hashCode，但是不允许给它们提供default方法。而且接口中不允许包含实例域或者非公有的静态成员（私有的静态方法除外）。最后一点，无法给不受你控制的接口添加default方法。

但是，通过对接口提供一个抽象的骨架实现类，可以把接口和抽象类的优点结合起来。接口负责定义类型，或许还提供一些default方法，而骨架实现类则负责实现除基本类型接口方法之外，剩下的非基本类型接口方法。扩展骨架实现占了实现接口之外的大部分工作。这就是模板方法模式。

按照惯例，骨架实现类被称为AbstractInterface，这里的Interface是指所实现的接口的名字。例如，Collections Framework为每个重要的集合接口都提供了一个骨架实现，包括AbstractCollection、AbstractSet、AbstractList和AbstractMap。将它们称作SkeletalCollection、SkeletalSet、SkeletalList和SkeletalMap也是有道理的，但是现在Abstract的用法已经根深蒂固。如果设计得当，骨架实现（无论是单独一个抽象类，还是接口中唯一包含的缺省方法）可以使程序员非常容易地提供他们自己的接口实现。例如，下面是一个静态工厂方法，除AbstractList之外，它还包含了一个完整的、功能全面的List实现：

```java
static List<Integer> intArrayAsList(int[] a) {
    Objects.requireNonNull(a);
    return new AbstractList<>() {
        @Override
        public Integer get(int i) {
            return a[i];
        }

        @Override
        public Integer set(int i, Integer val) {
            int oldVal = a[i];
            a[i] = val;
            return oldVal;
        }

        @Override
        public int size() {
            return a.length;
        }
    }
}
```

如果想知道一个List实现应该为你完成哪些工作，这个例子就充分演示了骨架实现的强大功能。顺便提一下，这个例子是个Adapter，它允许将int数组看作Integer实例的列表。由于在int值和Integer实例之间来回转换需要开销，它的性能不会很好。注意，这个实现采用了匿名类的形式。

骨架实现类的美妙之处在于，它们为抽象类提供了实现上的帮助，但又不强加“抽象类被用作类型定义时”所持有的严格限制。对于接口的大多数实现来讲，扩展骨架实现类时个很显然的选择，但并不是必须的。如果预置的类无法扩展骨架实现类，这个类始终能手工实现这个接口。同时，这个类本身仍然受益于接口中出现的任何default方法。此外，骨架实现类仍然有助于接口的实现。实现了这个接口的类可以把对于接口方法的调用转发到一个内部私有类的实例上，这个内部私有类扩展了骨架实现类。这种方法被称作模拟多重继承。

编写骨架实现类相对比较简单，只是过程有点乏味。首先，必须认真研究接口，并确定哪些方法是最为基本的，其他的方法则可以根据它们来实现。这些基本方法将成为骨架实现类中的抽象方法。接下来，在接口中为所有可以在基本方法之上直接实现的方法提供default方法，弹药记住，不能为Object方法（如equals和hashCode）提供缺省方法。如果基本方法和缺省方法覆盖了接口，你的任务就完成了，不需要骨架实现类了。否则，就要编写一个类，声明实现接口，并实现所有剩下的接口方法。这个类可以包含任何非公有的域，以及适合该任务的任何方法。

以Map.Entry接口为例，举个简单的例子。明显的基本方法是getKey、getValue和setValue。接口定义了equals和hashCode的行为，并且有一个明显的toString实现。由于不允许给Object方法提供缺省实现，因此所有实现都放在骨架实现类中：

```java
public abstract class AbstractMapEntry<K, V> implements Map.Entry<K, V> {
    @Override
    public V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    @Override
    public boolean equals(Object o) {
        if (o == this) {
            return true;
        }
        if (!(o instanceof Map.Entry)) {
            return false;
        }
        Map.Entry<?, ?> e = (Map.Entry) o;
        return Objects.equals(e.getKey(), getKey()) && Objects.equals(e.getValue(), getValue());
    }

    @Override
    public int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    @Override

    public String toString() {
        return getKey() + "=" + getValue();
    }
}
```

注意，这个骨架实现不能再Map.Entry接口中实现，也不能作为子接口，因为不允许default方法覆盖Object方法，如equals、hashCode和toString。

骨架实现上有个小小的不同，就是简单实现，AbstractMap.SimpleEntry就是就是个例子。简单实现就像骨架实现一样，这是因为它实现了接口，并且是为了继承而设计的，但是区别在于它不是抽象的：它是最简单的可能的有效实现。你可以原封不动地使用，也可以看情况将它子类化。

总而言之，接口通常是定义允许多个实现的类型的最佳途径。如果你导出了一个重要的接口，就应该坚决考虑同时提供骨架实现类。而且，还应该尽可能地通过default方法在接口中提供骨架实现，以便接口的所有实现类都能使用。也就是说，对于接口的限制，通常也限制了骨架实现会采用的抽象类的形式。

# 第21条：为后代设计接口

在Java 8发行之前，如果不破坏现有的实现，是不可能给接口添加方法的。如果给某个接口添加了一个新的方法，一般来说，现有的实现中是没有这个方法的，因此就会导致编译错误。在Java 8中，增加了default方法构造，目的就是允许给现有的接口添加方法。但是给现有接口添加新方法还是充满风险的。

default方法的声明中包括一个default实现，这是给实现了该接口但没有实现默认方法的所有类使用的。虽然Java增加了default方法之后，可以给现有接口添加方法了，但是并不能确保这些方法在之前存在的实现中都能良好运行。因为这些默认的方法是被“注入”到现有实现中的，它们的实现者并不知道，也没有许可。在Java 8之前，编写这些实现时，是默认它们的接口永远不需要任何新方法的。

Java 8在核心集合接口中增加了许多新的default方法，主要是为了便于使用lambda。Java类库的default方法是高品质的通用实现，它们在大多数情况下都能正常使用。但是，并非每一个可能的实现的所有变体，始终都可以编写出一个default方法。

比如，以removeIf方法为例，它在Java 8中被添加到了Collection接口。这个方法用来移除所有元素，并用一个boolean函数（或者断言）返回true。default实现指定用其迭代器来遍历集合，在每个元素上调用断言，并利用迭代器的remove方法移除断言返回值为true的元素。其声明大致如下：

```java
default boolean removeIf(Predicate<? super E> filter) {
    Objects.requireNonNull(filter);
    boolean result = false;
    for (Iterator<E> it = iterator(); it.hasNext(); ) {
        if (filter.test(it.next())) {
            it.remove();
            result = true;
        }
    }
    return result;
}
```

这是适用于removeIf方法的最佳通用实现，但遗憾的是，它在某些现实的Collection实现中会出错。比如，以org.apache.commons.collections4.CollectionUtils.SynchronizedCollection为例，这个类来自Apache Commons类库，类似于java.util中的静态工厂Collections.synchronizedCollection。Apache版本额外提供了利用客户端提供的对象（而不是用集合）进行锁定的功能。换句话说，它是一个包装类，它的所有方法在委托给包装集合之前，都在锁定对象上进行了同步。

Apache版本的SynchronizedCollection类依然有人维护，但是到编写本书之时，它也没有取代removeIf方法。如果这个类与Java 8结合使用，将会继承removeIf的default实现，它不会（实际上也无法）保持这个类的基本承诺：围绕着每一个方法调用执行自动同步。default实现压根不知道同步这回事，也无权访问包含该锁定对象的域。如果客户在SynchronizedCollection实例上调用removeIf方法，同时另一个线程对该集合进行修改，就会导致ConcurrentModificationException或者其他异常行为。

为了避免在类似的Java平台类库实现中发生这种异常，如Collections.synchronizedCollection返回的包私有类，JDK维护人员必须覆盖默认的removeIf实现，以及像它一样的其他方法，以便在调用default实现之前执行必要的同步。不属于Java平台组成部分的预先存在的集合实现，过去无法做出与接口变化相类似的改变，现在有些已经可以做到了。

`有了default方法，接口的现有实现就不会出现编译时没有报错或警告，运行时却失败的情况。`这个问题虽然并非普遍，但也不是孤立的意外事件。Java 8在集合接口中添加许多方法是极易受影响的，有些现有实现已知将会受到影响。

建议尽量避免利用default方法在现有接口上添加新的方法，除非有特殊需要，但就算在那样的情况下也应该慎重考虑：default的方法实现是否会破坏现有的接口实现。然而，在创建接口的时候，用default方法提供标准的方法实现时非常方便的，它简化了实现接口的任务。

还要注意的是，default方法不支持从接口中删除方法，也不支持修改现有方法的签名。对接口进行这些修改肯定会破坏现有的客户端代码。

结论很明显：尽管default方法现在已经是Java平台的组成部分，`但谨慎设计接口仍然是至关重要的。`虽然default方法可以在现有接口上添加方法，但这么做还是存在着很大的风险。就算接口中只有细微的缺陷都可能永远给用户带来不愉快；假如接口有严重的缺陷，则可能摧毁包含它的API。

因此，在发布程序之前，测试每一个新的接口就显得尤其重要。程序员应该以不同的方法实现每一个接口。最起码不应少于三种实现。编写多个客户端程序，利用每个新接口的实例来执行不同的任务，这一点也同样重要。这些步骤对确保每个接口都能满足其既定的所有用途起到了很大的帮助。它还有助于在接口发布之前及时发现其中的缺陷，使你依然能够轻松地把它们纠正过来。`或许接口程序发布之后也能纠正，但是千万别指望它啦！`

# 第22条：接口只用于定义类型

当类实现接口时，接口就充当可以引用这个类的实例的类型。因此，类实现了接口，就表明客户端可以对这个类的实例实施某些动作。为了任何其他目的而定义接口是不恰当的。

有一种接口被称为常量接口，它不满足上面的条件。这种接口不包含任何方法，它只包含静态的final域，每个域都导出一个常量。使用这些常量的类实现这个接口，以避免用类名来修饰常量名。下面举个例子：

```java
public interface PhysicalConstants {
    static final double AVOGADROS_NUMBER = 6.022_140_857e23;

    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

常量接口模式是对接口的不良使用。类在内部使用某些常量，这纯粹是实现细节。实现常量接口会导致把这样的实现细节泄漏到该类的导出API中。类实现常量接口对于该类的用户而言并没有什么价值。实际上，这样做反而会使他们更加糊涂。更糟糕的是，它代表了一种承诺：如果在将来的发行版本中，这个类被修改了，它不再需要使用这些常量了，它依然必须实现这个接口，以确保二进制兼容性。如果非final类实现了常量接口，它的所有子类的命名空间也会被接口中的常量所“污染”。

在Java平台类库中有几个常量接口，例如java.io.ObjectStreamConstants。这些接口应该被认为是反面的典型，不值得仿效。

如果要导出常量，可以有几种合理的选择方案。如果这些常量与某个现有的类或者接口紧密相关，就应该把这些常量添加到这个类或者接口中。例如，在Java平台类库中所有的数值包装类，如Integer和Double，都导出了MIN_VALUE和MAX_VALUE常量。如果这些常量最好被看作枚举类型的成员，就应该用枚举类型来导出这些常量。否则，应该使用不可实例化的工具类来导出这些常量。下面的例子是前面的PhysicalConstants例子的工具类翻版：

```java
public class PhysicalConstants {
    private PhysicalConstants() {}

    static final double AVOGADROS_NUMBER = 6.022_140_857e23;

    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    static final double ELECTRON_MASS = 9.109_383_56e-31;
}
```

注意，有时候会在数字的字面量中使用下划线。从Java 7开始，下划线的使用已经合法了，它对数字字面量的值没有影响，如果使用得当，还可以极大地提升它们的可读性。如果其中包含五个或五个以上连续的数字，无论是浮点还是定点，都要考虑在数字的字面量中添加下划线。对于基数为10的字面量，无论是整数还是浮点，都应该用下划线把数字隔成每三位一组。

工具类通常要求客户端用类名来修饰这些常量名，例如PhysicalConstants.AVOGADROS_NUMBER。如果大量利用工具类导出的常量，可以通过利用静态导入机制，避免用类名来修饰常量。

简而言之，接口应该只被用来定义类型，它们不应该被用来导出常量。

# 第23条：类层次优于标签类

有时可能会遇到带有两种甚至更多风格的实例的类，并包含表示实例风格的标签域。例如，以下面这个类为例，它能够表示圆形或者矩形：

```java
class Figure {
    enum Shape {RECTANGLE, CIRCLE};

    final Shape shape;

    double length;

    double width;

    double radius;

    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

这种标签类有许多缺点。它们中充斥着样板代码，包括枚举声明、标签域以及条件语句。由于多个实现乱七八糟地挤在单个类中，破坏了可读性。由于实例承担着属于其他风格的不相关的域，因此内存占用也增加了。域不能做成final的，除非构造器初始化了不相关的域，产生了更多的样板代码。构造器必须不借助编译器来设置标签域，并初始化正确的数据域：如果初始化了错误的域，程序就会在运行时失败。无法给标签类添加风格，除非可以修改它的源文件。如果一定要添加风格，就必须记得给每个条件语句都添加一个条件，否则类就会在运行时失败。最后，实例的数据类型没有提供任何关于其风格的线索。一句话，`标签类过于冗长、容易出错，并且效率低下。`

幸运的是，面向对象的语言提供了其他更好的方法来定义能表示多种风格对象的单个数据类型：子类型化。`标签类正是对类层次的一种简单的仿效。`

为了将标签类转变成类层次，首先要为标签类的每个方法都定义一个包含抽象方法的抽象类，标签类的行为依赖于标签值。在Figure类中，只有一个这样的方法：area。这个抽象类是类层次的根。如果还有其他的方法其行为不依赖于标签的值，就把这样的方法放在这个类中。同样地，如果所有的方法都用到了某些数据域，就应该把它们放在这个类中。在Figure类中，不存在这种类型独立的方法或者数据域。

接下来，为每种原始标签类都定义根类的具体子类。在前面的例子中，这样的类型有两个：圆形（circle）和矩形（rectangle）。在每个子类中都包含特定于该类型的数据域。在我们的示例中，radius是特定于圆形的，length和width是特定于矩形的。同时在每个子类中还包括针对根类中每个抽象方法的相应实现。以下是与原始的Figure类相对应的类层次：

```java
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {
    final double length;

    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    @Override
    double area() {
        return length * width;
    }
}
```

这个类层次纠正了前面提到过的标签类的所有缺点。这段代码简单且清楚，不包含在原来的版本中见到的所有样板代码。每个类型的实现都配有自己的类，这些类都没有受到不相关数据域的拖累。所有的域都是final的。编译器确保每个类的构造器都初始化它的数据域，对于根类中声明的每个抽象方法都确保有一个实现。这样就杜绝了由于遗漏switch case而导致运行时失败的可能性。多名程序员可以独立地扩展层次结构，并且不用访问根类的源代码就能相互操作。每种类型都有一种相关的独立的数据类型，允许程序员指明变量的类型，限制变量，并将参数输入到特殊的类型。

类层次的另一个好处在于，它们可以用来反映类型之间本质上的层次关系，有助于增强灵活性，并有助于更好地进行编译时类型检查。假设上述例子中的标签类也允许表达正方形。类层次可以反映出正方形是一种特殊的矩形这一事实（假设两者都是不可变的）：

```java
class Square extends Rectangle {
    Square(double side) {
        super(side, side);
    }
}
```

注意，上述层次中的域被直接访问，而不是通过访问方法访问。这是为了简洁起见，如果层次结构是公有的，则不允许这样做。

简而言之，标签类很少有适用的时候。当你想要编写一个包含显式标签域的类时，应该考虑一下，这个标签是否可以取消，这个类是否可以用类层次来代替。当你遇到一个包含标签域的现有类时，就要考虑将它重构到一个层次结构中去。

# 第24条：静态成员类优于非静态成员类

嵌套类是指定义在另一个类的内部的类。嵌套类存在的目的应该只是为它的外围类提供服务。如果嵌套类将来可能会用于其他的某个环境中，它就应该是顶层类。嵌套类有四种：静态成员类、非静态成员类、匿名类和局部类。除了第一种之外，其他三种都称为内部类。本条目将告诉你什么时候应该使用哪种嵌套类，以及这样做的原因。

静态成员类是最简单的一种嵌套类。最好把它看作是普通的类，只是碰巧被声明在另一个类的内部而已，它可以访问外围类的所有成员，包括那些声明为私有的成员。静态成员外围类是外围类的一个静态成员，与其他的静态成员一样，也遵守同样的可访问性规则。如果它被声明为私有的，它就只能在外围类的内部才可以被访问，等等。

静态成员类的一种常见用法是作为公有的辅助类，只有与它的外部类一起使用才有意义。例如，以枚举为例，它描述了计算机支持的各种操作。Operation枚举应该是Calculator类的公有静态成员类，之后Calculator类的客户端就可以用诸如Calculator.Operation.PLUS和Calculator.Operation.MINUS这样的名称来引用这些操作。

从语法上讲，静态成员类和非静态成员类之间唯一的区别是，静态成员类的声明中包含修饰符static。尽管它们的语法非常相似，但是这两种嵌套类有很大的不同。非静态成员类的每个实例都隐含地与外围类的一个外围实例相关联。在非静态成员类的实例方法内部，可以调用外围实例上的方法，或者利用修饰过的this构造获得外围实例的引用。如果嵌套类的实例可以在它外围类的实例之外独立存在，这个嵌套类就必须是静态成员类：在没有外围实例的情况下，要想创建非静态成员类的实例是不可能的。

当非静态成员类的实例被创建的时候，它和外围实例之间的关联关系也随之被建立起来；而且，这种关联关系以后不能被修改。通常情况下，当在外围类的某个实例方法的内部调用非静态成员类的构造器时，这种关联关系被自动建立起来。使用表达式enclosing-Instance.new MemberClass(args)来手工建立这种关联关系也是有可能的，但是很少使用。正如你所预料的那样，这种关联关系需要消耗非静态成员类实例的空间，而且会增加构造的时间开销。

非静态成员类的一种常见用法是定义一个Adapter，它允许外部类的实例被看作是另一个不相关的类的实例。例如，Map接口的实现往往使用非静态成员类来实现它们的集合视图，这些集合视图是由Map的keySet、entrySet和values方法返回的。同样地，诸如Set和List这种集合接口的实现往往也使用非静态成员类来实现它们的迭代器：

```java
public class MySet<E> extends AbstractSet<E> {
    @Override
    public Iterator<E> iterator() {
        return new MyIterator();
    }

    private class MyIterator implements Iterator<E> {
        ...
    }
}
```

如果声明成员类不要求访问外围实例，就要始终把修饰符static放在它的声明中，使它成员静态成员类，而不是非静态成员类。如果省略了static修饰符，则每个实例都将包含一个额外的指向外围对象的引用。如前所述，保存这份引用要消耗时间和空间，并且会导致外围实例在符合垃圾回收时却仍然得以保留。由此造成的内存泄漏可能是灾难性的。但是常常难以发现，因为这个引用是不可见的。

私有静态成员类的一种常见用法是代表外围类所代表的对象的组件。以Map实例为例，它把key和value关联起来。许多Map实现的内部都有一个Entry对象，对应于Map中的每个key-value对。虽然每个entry都与一个Map关联，但是entry上的方法（getKey、getValue和setValue）并不需要访问该Map。因此，使用费静态成员类来标识entry是很浪费的：私有的静态成员类是最佳的选择。如果不小心漏掉了entry声明中的static修饰符，该Map仍然可以工作，但是每个entry中将会包含一个指向该Map的引用，这样就浪费了空间和时间。

如果相关的类是导出类的公有或受保护的成员，毫无疑问，在静态和非静态成员类之间做出正确的选择是非常重要的。在这种情况下，该成员类就是导出的API元素，在后续的发行版本中，如果不违反向后兼容性，就无法从非静态成员类变为静态成员类。

顾名思义，匿名类是没有名字的。它不是外围类的一个成员。它并不与其他的成员一起呗声明，而是在使用的同时被声明和实例化。匿名类可以出现在代码中任何允许存在表达式的地方。当且仅当匿名类出现在非静态的环境中时，它才有外围实例。但是即使它们出现在静态的环境中，也不可能拥有任何静态成员，而是拥有常数变量，常数变量是final基本类型，或者被初始化成常量表达式的字符串域。

匿名类的运用受到诸多的限制。除了在它们被声明的时候之外，是无法将它们实例化的。不能执行instanceof测试，或者做任何需要命名类的其他事情。无法声明一个匿名类来实现多个接口，或者扩展一个类，并同时扩展类和实现接口。除了从超类型中继承得到之外，匿名类的客户端无法调用任何成员。由于匿名类出现在表达式中，它们必须保持简短（大约10行或者更少），否则会影响程序的可读性。

在Java中增加lambda之前，匿名类是动态地创建小型函数对象和过程对象的最佳方式，但是现在会优先选择lambda。匿名类的另一种常见用法是在静态工厂方法的内部。

局部类是四种嵌套类中使用最少的类。在任何“可以声明局部变量”的地方，都可以声明局部类，并且局部类也遵守同样的作用域规则。局部类与其他三种嵌套类中的每一种都有一些共同的属性。与成员类一样，局部类有名字，可以被重复使用。与匿名类一样，只有当局部类是在非静态环境中定义的时候，才有外围实例，它们也不能包含静态成员。与匿名内部类一样，它们必须非常简短，以便不会影响可读性。

总而言之，公有四种不同的嵌套类，每一种都有自己的用途。如果一个嵌套类需要在单个方法之外仍然是可见的，或者它太长了，不适合放在方法内部，就应该使用成员类。如果成员类的每个实例都需要一个指向其外围实例的引用，就要把成员类做成非静态的；否则，就做成静态的。假设这个嵌套类属于一个方法的内部，如果你只需要在一个地方创建实例，并且已经有了一个预置的类型可以说明这个类的特征，就要把它做成你们类；否则，就做成局部类。

# 第25条：限制源文件为单个顶级类

虽然Java编译器允许在一个源文件中定义多个顶级类，但这么做并没有什么好处，只会带来巨大的风险。因为在一个源文件中定义多个顶级类，可能导致给一个类提供多个定义，哪一个定义会被用到，取决于源文件被传给编译器的顺序。

为了更具体地说明，下面举个例子，这个源文件中中包含一个Main类，它将引用另外两个顶级类（Utensil和Dessert）的成员：

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}
```

现在假设你在一个名为Utensil.java的源文件中同时定义了Utensil和Dessert：

```java
class Utensil {
    static final String NAME = "pan";
}

class Desert {
    static final String NAME = "cake";
}
```

当然，主程序会打印出“pancake”。

现在假设你不小心在另一个名为Dessert.java的源文件中也定义了同样的两个类：

```java
class Utensil {
    static final String NAME = "pot";
    static final String NAME = "pie";
}
```

如果你侥幸是用命令javac Main.java Dessert.java来编译程序，那么编译就会失败，此时编译器会提醒你定义了多个Utensil和Dessert类。这是因为编译器会先编译Main.java，当它看到Utensil的引用（在Dessert引用之前），就会在Utensil.java中查看这个类，结果找到Utensil和Dessert这两个类。当编译器在命令行遇到Dessert.java时，也会去查找该文件，结果会遇到Utensil和Dessert这两个定义。

如果用命令javac Main.java或者javac Main.java Utensil.java编译程序，结果将如同你还没有编写Dessert.java文件一样，输出pancake。程序的行为受源文件被传给编译器的顺序影响，这显然是让人无法接受的。

这个问题的修正方法很简单，只要把顶级类（在本例中是指Utensil和Dessert）分别放入独立的源文件即可。如果一定要把多个顶级类放进一个源文件中，就要考虑使用静态成员类，以此代替将这两个类分到独立的源文件中去。如果这些类服从于另一个类，那么将它们做成静态成员类通常比较好，因为这样增强了代码的可读性，如果将这些类声明为私有的，还可以使它们减少被读取的概率。以下就是做成静态成员类的范例：

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```

结论显而易见：`永远不要把多个顶级类或者接口放在一个源文件中`。遵循这个原则可以确保编译时一个类不会有多个定义。这么做反过来也能确保编译产生的类文件，以及程序结果的行为，都不会受到源文件被传给编译器的顺序的影响。

# 第26条：请不要使用原生态类型

声明中具有一个或者多个类型参数的类或者接口，就是泛型类或者接口。例如，List接口就只有单个类型参数E，表示列表的元素类型。这个接口的全称是List<E>，但是人们经常把它简称为List。泛型类和接口统称为泛型。

每一种泛型定义一组参数化的类型，构成格式为：先是类或者接口的名称，接着用尖括号（<>）把对应于泛型形式类型参数的实际类型参数列表括起来。例如，List<String>（读作“字符串列表”）是一个参数化的类型，表示元素为String的列表。（String是与形式的类型参数E相对应的实际类型参数。）

最后一点，每一种泛型都定义一个原生态类型，即不带任何实际类型参数的泛型名称。例如，与List<E>相对应的原生态类型是List。原生态类型就像从类型声明中删除了所有泛型信息一样。它们的存在主要是为了与泛型出现之前的代码相兼容。

在Java增加泛型之前，下面这个集合声明是值得参考的。从Java 9开始，它依然是合法，但是已经没什么参考价值了：

```java
private final Collection stamps = ...;
```

如果现在使用这条声明，并且不小心将一个coin放进了stamp集合中，这一错误的插入照样得以编译和运行，不会出错（不过编译器确实会发出一条模糊的警告信息）：

```java
stamps.add(new Coin(...));
```

直到从stamp集合中获取coin时才会收到一条错误提示：

```java
for (Iterator i = stamps.iterator(); i.hasNext(); ) {
    Stamp stamp = (Stamp) i.next();
    stamp.cancel();
}
```

出错之后应该尽早发现，最好是编译时就发现。在本例中，直到运行时才发现错误，已经出错很久了，而且它在代码中所处的位置，距离包含错误的这部分代码已经很远了。一旦发现ClassCastException，就必须搜索代码，查找将coin放进stamp集合的方法调用。此时编译器帮不上忙，因为它无法理解这种注释：“Contains only Stamp instances”（只包含Stamp实例）。

有个泛型之后，类型声明中可以包含以下信息，而不是注释：

```java
private final Collection<Stamp> stamps = ...;
```

通过这条声明，编译器知道stamps应该只包含Stamp实例，并给予保证，假设整个代码库在编译过程中都没有发出任何警告。当stamps利用一个参数化的类型进行声明时，错误的插入会产生一条编译时的错误消息，告诉你具体是哪里错误了。

从集合中检索元素的时候，编译器会替你插入隐式的转换，并确保它们不会失败（依然假设所有代码都没有产生或者隐瞒任何编译警告）。假设不小心将coin插入stamp集合，这显得有点牵强，但这类问题却是真实的。例如，很容易想象有人会不小心将一个BigInteger实例放进一个原本只包含BIgDecimal实例的集合中。

如上所述，使用原生态类型（没有类型参数的泛型）是合法的，但是永远不应该这么做。`如果使用原生态类型，就失掉了泛型在安全性和描述性方面的所有优势。`既然不应该使用原生态类型，为什么Java语言的设计者还要允许使用它们呢？这是为了提供兼容性。因为泛型出现的时候，Java平台即将进入它的第二个十年，已经存在大量没有使用泛型的Java代码。人们认为让所有这些代码保持合法，并且能够与使用泛型的新代码互用，这一点很重要。它必须合法才能将参数化类型的实例传递给那些被设计成使用普通类型的方法，反之亦然。这种需求被称作移植兼容性，促成了支持原生态类型，以及利用擦除实现泛型的决定。

虽然不应该在新代码中使用像List这样的原生态类型，使用参数化的类型以允许插入任意对象（比如List<Object>）是可行的。原生态类型List和参数化的类型List<Object>之间到底有什么区别呢？不严格地说，前者逃避了泛型检查，后者则明确告知编译器，它能够持有任意类型的对象。虽然可以将List<String>传递给类型List的参数，但是不能将它传给类型List<Object>的参数。泛型有子类型化的规则，List<String>是原生态类型List的一个子类型，而不是参数化类型List<Object>的子类型。因此，如果使用像List这样的原生态类型，就会失掉类型安全性，但是如果使用像List<Object>这样的参数化类型，则不会。

为了更具体地进行说明，请参考下面的程序：

```java
public static void main(String[] args) {
    List<String> strings = new ArrayList<>();
    unsafeAdd(strings, Integer.valueOf(42));
    String s = strings.get(0);
}

private static void unsafeAdd(List list, Object o) {
    list.add(o);
}
```

这段程序可以进行编译，但是因为它使用了原生态类型List，你会收到一条警告：

```java
Test.java:10:warning: [unchecked] unchecked call to add(E) as a memeber of the raw type list
    list.add(o);
```

实际上，如果运行这段程序，在程序试图将strings.get(0)的调用结果Integer转换成String时，你会收到一个ClassCastException异常。这是一个编译器生成的转换，因此一般保证会成功，但是我们在这个例子中忽略了一条编译器警告，为此付出了代价。

如果在unsafeAdd声明中用参数化类型List<Object>代替原生态类型List，并试着重新编译这段程序，会发现它无法再进行编译了，并发出以下错误消息：

```java
Test.java:5 error: incompatible types: List<String> cannot be converted to List<Object>
    unsafeAdd(strings, Integer.valueOf(42));
```

在不确定或者不在乎集合中的元素类型的情况下，你也许会使用原生态类型。例如，假设想要编写一个方法，它有两个集合，并从中返回它们共有元素的数量。如果你对泛型还不熟悉，可以参考以下方式来编写这种方法：

```java
static int numElementsInCommon(Set s1, Set s2) {
    int result = 0;
    for (Object o1 : s1) {
        if (s2.contains(o1)) {
            result++;
        }
    }
    return result;
}
```

这个方法可行，但它使用了原生态类型，这是很危险的。安全的替代做法是使用无限制的通配符类型。如果要使用泛型，但不确定或者不关心实际的类型参数，就可以用一个问号代替。例如，泛型Set<E>的无限制通配符类型为Set<?>（读作”某个类型的集合”）。这是最普通的参数化Set类型，可以持有任何集合。下面是numElementsInCommon方法使用了无限制通配符类型时的情形：

```java
static int numElementsInCommon(Set<?> s1, Set<?> s2) {}
```

无限制通配类型Set<?>和原生态类型Set之间有什么区别呢？这个问号真正起到作用了吗？这一点不需要赘述，但通配符类型是安全的，原生态类型则不安全。由于可以将任何元素放进使用原生态类型的集合中，因此很容易破坏该集合的类型约束条件；但`不能讲任何元素（除了null之外）放到Collection<?>中。`如果尝试这么做，将会产生一条像这样的编译时错误消息：

```java
WildCard.java:13: error: incompatible types: String cannot be converted to CAP#1
    c.add("verboten");
          ^
    where CAP#1 is a fresh type-variable:
    CAP#1 extends Object from capture of ?
```

这样的错误消息显然无法令人满意，但是编译器已经尽到了它的职责，防止你破坏集合的类型约束条件。你不仅无法将任何元素（除了null之外）放进Collection<?>中，而且根本无法猜测你会得到哪种类型的对象。要是无法接受这些限制，就可以使用泛型方法或者有限制的通配符类型。

不要使用原生态类型，这条规则有几个小小的例外。`必须在类文字中使用原生态类型。`规范不允许使用参数化类型（虽然允许数组类型和基本类型）。换句话说，List.class、String[].class和int.class都合法，但是List<String>.class和List<?>.class则不合法。

这条规则的第二个例外与instanceof操作符有关。由于泛型信息可以在运行时被擦除，因此在参数化类型而非无限制通配符类型上使用instanceof操作符是非法的。`用无限制通配符类型代替原生态类型，对instanceof操作符不会产生任何影响。`在这种情况下，尖括号（<>）和问号（?）就显得多余了。`下面是利用泛型来使用instanceof操作符的首选非法：`

```java
if (o instanceof Set) {
    Set<?> s = (Set<?>) o;
}
```

注意，一旦确定这个o是个Set，就必须将它转换成通配符类型Set<?>，而不是转换成原生态类型Set。这是个受检的转换，因此不会导致编译时警告。

总而言之，使用原生态类型会在运行时导致异常，因此不要使用。原生态类型只是为了与引入泛型之前的遗留代码进行兼容和互用而提供的。Set<Object>是个参数化类型，表示可以包含任何对象类型的一个集合；Set<?>则是一个通配符类型，表示只能包含某种未知对象类型的一个集合；Set是一个原生态类型，它脱离了泛型系统。前两种是安全的，最后一种不安全。

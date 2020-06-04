# Java Reflection

## 基础对象

```java
@Recommended("on class")
public class MyClass extends SuperClass implements Flyable {

    @Recommended("on field")
    private String name;

    private int age;

    private List<String> habits;

    public List<String> getHabits() {
        return habits;
    }

    public void setHabits(List<String> habits) {
        this.habits = habits;
    }

    @Override
    public void fly() {
        System.out.println("flying ....");
    }

    public MyClass() {
    }

    public MyClass(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Recommended("on method")
    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "MyObject{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    public static String doSomeThing() {
        return "I'm static method";
    }

    public void annotationSome(@Recommended("on  param") String msg) {
        System.out.println("annotation some");
    }

}
```



## 创建对象

```java
// 创建无参对象
Constructor<MyClass> constructor = MyClass.class.getConstructor();
constructor.newInstance();

// 创建有参对象
Constructor<MyClass> constructor = MyClass.class.getConstructor(String.class);
constructor.newInstance("msg");

```

## 调用对象的方法

```java
public class ReflectionMethodDemo {

    public static void main(String[] args) throws Exception {
        Class<MyClass> clazz = MyClass.class;
        MyClass myClass = clazz.getConstructor(String.class).newInstance("jack");

        List<Method> methods = Arrays.stream(clazz.getDeclaredMethods()).filter(e -> e.getName().startsWith("get")).collect(Collectors.toList());
        for (Method method : methods) {
            Object result = method.invoke(myClass);
            System.out.println("invoked " + method.getName() + " returned: " + result);
        }

        // invoke some method
        Method setAgeMethod = clazz.getDeclaredMethod("setAge", int.class);
        setAgeMethod.invoke(myClass, 18);
        Method getAgeMethod = clazz.getDeclaredMethod("getAge");
        Object result = getAgeMethod.invoke(myClass);
        System.out.println("age:" + result);

        // invoke static method
        Method doSomeThingMethod = clazz.getDeclaredMethod("doSomeThing");
        Object staticMethodInvokeResult = doSomeThingMethod.invoke(null);
        System.out.println("invoke static method, result= " + staticMethodInvokeResult);
    }

    public static boolean isSetter(Method method) {
        if (!method.getName().startsWith("set")) return false;
        if (method.getParameterTypes().length != 1) return false;
        return true;
    }

    public static boolean isGetter(Method method) {
        if (!method.getName().startsWith("get")) return false;
        if (method.getParameterTypes().length != 0) return false;
        if (Objects.equals(method.getReturnType(), void.class)) return false;
        return true;
    }
}

```



## 判断Method是否getter或者setter

```java
public static boolean isSetter(Method method) {
  if (!method.getName().startsWith("set")) return false;
  if (method.getParameterTypes().length != 1) return false;
  return true;
}

public static boolean isGetter(Method method) {
  if (!method.getName().startsWith("get")) return false;
  if (method.getParameterTypes().length != 0) return false;
  if (Objects.equals(method.getReturnType(), void.class)) return false;
  return true;
}
```

## 获取泛型信息

```java
public class ReflectionGenericDemo {

    public static void main(String[] args) throws Exception {
        Class<MyClass> clazz = MyClass.class;

        // get return type  generic
        Method getHabitsMethod = clazz.getMethod("getHabits");
        Type genericReturnType = getHabitsMethod.getGenericReturnType();
        if (genericReturnType instanceof ParameterizedType) {
            ParameterizedType parameterizedType = (ParameterizedType) genericReturnType;
            Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
            for (Type argument : actualTypeArguments) {
                Class<String> stringClass = (Class<String>) argument;
                System.out.println("return generic type: "+stringClass);
            }
        }

        // get parameter generic type
        Method setHabitsMethod = clazz.getMethod("setHabits", List.class);
        Type[] genericParameterTypes = setHabitsMethod.getGenericParameterTypes();
        for (Type genericParameterType : genericParameterTypes) {
            if (genericParameterType instanceof ParameterizedType) {
                ParameterizedType parameterizedType = (ParameterizedType) genericParameterType;
                Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
                for (Type arg : actualTypeArguments) {
                    Class<String> stringClass = (Class<String>) arg;
                    System.out.println("parameter generic type: "+stringClass);
                }
            }
        }

        // get field generic type
        Field habitsField = clazz.getDeclaredField("habits");
        Type genericType = habitsField.getGenericType();
        if (genericType instanceof ParameterizedType) {
            ParameterizedType parameterizedType = (ParameterizedType) genericType;
            Type[] actualTypeArguments = parameterizedType.getActualTypeArguments();
            for (Type arg : actualTypeArguments) {
                System.out.println("field generic type: "+ arg.getTypeName());
            }
        }

    }
}
```

## 获取注解信息

```java
public class ReflectionAnnotationDemo {

    public static void main(String[] args) throws Exception{
        Class<MyClass> clazz = MyClass.class;
        Recommended recommended = clazz.getDeclaredAnnotation(Recommended.class);
        if (recommended != null) {
            String value = recommended.value();
            System.out.println("class annotation value: " + value);
        }

        Method method = clazz.getMethod("getAge");
        Recommended methodAnnotation = method.getAnnotation(Recommended.class);
        if (methodAnnotation != null) {
            System.out.println("method annotation value: " + methodAnnotation.value());
        }

        Field field = clazz.getDeclaredField("name");
        Recommended fieldAnnotation = field.getAnnotation(Recommended.class);
        if (fieldAnnotation != null) {
            System.out.println("field annotation value: " + fieldAnnotation.value());
        }

        // get method parameter annotation
        Method annotationSomeMethod = clazz.getMethod("annotationSome", String.class);
        Annotation[][] parameterAnnotations = annotationSomeMethod.getParameterAnnotations();
        for (Annotation[] parameterAnnotation : parameterAnnotations) {
            for (Annotation annotation : parameterAnnotation) {
                if (annotation instanceof Recommended) {
                    Recommended r = (Recommended) annotation;
                    System.out.println("method parameter annotation value: " + r.value());
                }
            }
        }


    }
```



## 操作数组

```java
public class ReflectionArrayDemo {

    public static void main(String[] args) throws Exception {
        // create array via java reflection
        int[] arr = (int[]) Array.newInstance(int.class, 3);
        // set and get
        Array.set(arr, 0, 123);
        Array.set(arr, 1, 456);
        Array.set(arr, 2, 789);
        System.out.println(Arrays.toString(arr));

        System.out.println("第1个元素" + Array.get(arr, 0));
        System.out.println("第2个元素" + Array.get(arr, 1));
        System.out.println("第3个元素" + Array.get(arr, 2));

        // create int array
        Class<?> intClass = Class.forName("[I");
        System.out.println("is array:" + intClass.isArray());
        System.out.println(intClass.getComponentType());

        // create String array
        Class<?> stringClass = Class.forName("[Ljava.lang.String;");
        System.out.println("is array:" + stringClass.isArray());
        System.out.println(stringClass.getComponentType());

        // create Object array
        Class<?> myClassArrayClass = Class.forName("[Lorg.mickey.relection.MyClass;");
        System.out.println("myClassArrayClass's componentType: " + myClassArrayClass.getComponentType());

    }
}
```


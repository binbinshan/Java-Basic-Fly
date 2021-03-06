
# Java反射机制及应用场景

在运行状态的时候，对于任何一个类都能获取这个类的所有属性和方法，对于任何一个对象，都能获得这个对象的属性和方法。这种动态获取类和对象信息或者执行类和对象的方法的方式叫做Java反射。

反射的缺点和优点：
* 优点：运行期类型的判断，动态加载，提供了扩展性。
* 缺点：通过反射调用，性能会有损失。

应用场景：
* 通过动态代理，实现统计方法执行时长

```
动态代理就是基于JDK提供的反射包，动态的在内存中构建代理对象。

//定义一个接口，动态代理要求代理对象必须实现接口
public interface IUserDao {
    void save();
}

-------------
//定义目标对象
public class UserDao implements IUserDao {
    public void save() {
        System.out.println("----执行了方法sava----");
    }
}
-------------
//定义代理类
public class ProxyFactory{
    //维护一个目标对象
    private Object target;
    public ProxyFactory(Object target){
        this.target=target;
    }
   //给目标对象生成代理对象
   //Proxy.newProxyInstance的三个参数 
   //1.指定当前目标对象使用类加载器,获取加载器的方法是固定的
   //2.目标对象实现的接口的类型,使用泛型方式确认类型
   //3.事件处理,执行目标对象的方法时,会触发事件处理器的方法,会把当前执行目标对象的方法作为参数传入
 public Object getProxyInstance(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("记录开始时间！");
                        //执行目标对象方法
                        Object returnValue = method.invoke(target, args);
                        System.out.println("记录结束时间！");
                        return returnValue;
                    }
                }
        );
    }
}
-------------
//测试类
public class Test {
    public static void main(String[] args) {
        // 目标对象
        IUserDao target = new UserDao();
        // 【原始的类型 class cn.xxx.xxx.UserDao】
        System.out.println(target.getClass());

        // 给目标对象，创建代理对象
        IUserDao proxy = (IUserDao) new ProxyFactory(target).getProxyInstance();
        // class $Proxy0   内存中动态生成的代理对象
        System.out.println(proxy.getClass());

        // 执行方法   【代理对象】
        proxy.save();
    }
}
```






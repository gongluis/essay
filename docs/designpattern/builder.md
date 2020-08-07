### 定义  
将一个复杂对象的构建和表示分离，使得同样的构建过程可以构建不同的对象。

###   经典例子  
创建对象传递参数的时候往往通过构造函数来传，，随着构造函数的增加，代码会很难编写。  
例：生产组装一辆、台电脑  
```
public class computer{
    private final String display；
    private final String keyBoard；
    private final String mouse；
    
    public static class Builder{
    //必选字段
        private final String display；
        private final String keyBoard；
        //可选字段
        private final String mouse；
        
        public Builder(String display, String keyBoard){
            this.display = display;
            this.keyBoard = keyBoard;
            
        }
        public Builder mouse(String mouse){
            this.mouse = mouse;
            return this;
        }
        
        public Computer build(){
            return new Computer(this);
        }
    }
    private Computer (Builder builder){
        display = builder.display;
        keyBoard = builder.keyBoard;
        mouse = builder.mouse;
    }
}
```
测试代码  
```
Computer computer = new Computer.Builder("三星","华为")
.mouse("苹果")
.build();
```
Android 采用Builder模式的AlertDialog  
```
AlertDialog.Builder dialog = new AlertDialog.Nuilder(this);
dialog.setTitle("关于我们")
.setMessage("大家好!")
.creat()
.show();
```

### 优缺点
链式调用抑郁阅读和编写，出错难以debug
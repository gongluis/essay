```text
> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
> nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
> massa, nec semper lorem quam in massa.
> ```go
  import "fmt"
  func main() {
     fmt.Println("Hello, world")
  }
  ```
```

效果

> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
> nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
> massa, nec semper lorem quam in massa.
> ```go
  import "fmt"
  func main() {
     fmt.Println("Hello, world")
  }
  ```

可以看出，用>时候，要每行的>内容缩进对齐，不过，如果是一个代码块的话，只需要第一个>即可

但要注意, 上面这种方法不能有空行，如果有空行就会被认为终止，比如：
```text
> ```go
  import "fmt"
  func main() {
      fmt.Println("Hello, world")

      fmt.Println("last")
  }
  ```
```

![](./../../img/nest_error_example.png)

要解决这个问题，就要在空行之后的新行再用>表示未中断，比如：
```text
> ```go
  import "fmt"
  func main() {
      fmt.Println("Hello, world")

>     fmt.Println("last")
  }
  ```
```

> ```go
  import "fmt"
  func main() {
      fmt.Println("Hello, world")

>     fmt.Println("last")
  }
  ```

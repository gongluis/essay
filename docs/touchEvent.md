[TOC]
#### 事件分发  
##### 分发的是什么东西？即事件有哪些？  
```
1. MotionEvent.ACTION_DOWN
2. MotionEvent.ACTION_UP
3. MotionEvent.ACTION_MOVE
4. MotionEvent.ACTION_CANCEL
其中在一次滑动中Move事件会被多次触发
```  

##### 事件在谁之间传递？  
![event_object.jpg](https://i.loli.net/2020/08/29/J3giEAS2WnbaBOC.jpg)  


相关函数
```
Activity:
dispatchTouchEvent(MotionEvent event)
onTouchEvent(MotionEvent event)

ViewGroup:  
dispatchTouchEvent(MotionEvent event)
onInterCeptTouchEvent(MotionEvent event)
onTouchEvent(MotionEvent event)

View:
onTouchEvent(MotionEvent event)
dispatchTouchEvent(MotionEvent event)
```  

dispatchTouchEvent:事件传入时调用，分发事件  
OnTouchEvent:处理事件  
onInterceptTouchEvent是viewGroup特有的事件是否拦截事件  


##### 事件分发的过程  
![事件分发.png](https://i.loli.net/2020/08/31/Qxj3JPoeYIfcVUp.png)
 
可将U型图左边看成是分派阶段，右边是回溯阶段    
分派阶段：  

|期望行为|操作|  
|---|---|
|继续分配|调用super|
|消费事件，终止整个流程|dispatch return true|  
|终止分派，向上回溯|dispatch return false|
|终止分派，交给自己处理|ViewGroup: 拦截器返回 true; View: 调用 super|  

回溯阶段：  
|期望行为|操作|
|---|---|
|继续回溯|调用 super 或 return false|
|消费事件，终止回溯|return true|

例：activity中放一个linerlayout,linerlayout中放一个webview，事件传递如下：  
![event_dispatch.jpg](https://i.loli.net/2020/08/29/fc1wMDzkQEYbsrm.jpg)  
  
  事件的传递顺序：activity-viewGroup-view  
  事件处理的顺序：view-viewGroup-activity  
  
  
  1. 如果让viewGroup的onInterceptTouchEvnet返回true,事件不会再流向view.  
  ![onintercepeventtrure.jpg](https://i.loli.net/2020/08/29/P61MnKEGkZ879Jy.jpg)  
  
  2. 上面拦截了事件，使得事件不会传向view，但是并没有消费事件，事件最后还是流回了activity,如果要消费事件，一般要对控件添加setOnTouchListener的监听，并且返回true,表示事件就此消费，不再往回传递：  
  ```
   LinearLayout ll = findViewById(R.id.ll);
        ll.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                Log.d("TEST", "ViewGroup 消费事件" );
                return true;
            }
        });
  ```  
  
  3. 以上两条看到了viewGroup如何拦截和消费事件，view是如何消费事件？  
  ```
  TWebView tWebView = findViewById(R.id.tWebView);
       tWebView.setOnTouchListener(new View.OnTouchListener() {
           @Override
           public boolean onTouch(View v, MotionEvent event) {
               v.getParent().requestDisallowInterceptTouchEvent(true);
               Log.d("TEST TOUCH", "View 消费事件");
               return true;
           }
       });
  ```  
  
  同样的在setOnTouchListener中的onTouch方法中返回ture然后再加上requestDisallowInterceptTouchEvent(boolean disallowIntercept)请求父控件不要拦截事件，参数为ture，则不会拦截  
  ```
   TWebView tWebView = findViewById(R.id.tWebView);
       tWebView.setOnTouchListener(new View.OnTouchListener() {
           @Override
           public boolean onTouch(View v, MotionEvent event) {
               v.getParent().requestDisallowInterceptTouchEvent(true);
               Log.d("TEST TOUCH", "View 消费事件");
               return true;
           }
       });
  ```  
  
  ##### 经典例子  
  1. scrollView中嵌套viewpager(不推荐)  
  ```
  两种方法：  
  一、自定义父view scrollView，复写onInterceptTouchEvent方法，在MOVE中判断出横向移动不拦截事件。  
  
  public class MyScrollView extends ScrollView {
    private float mDownPosX = 0;
    private float mDownPosY = 0;

    public MyScrollView(Context context) {
        super(context);
    }

    public MyScrollView(Context context, AttributeSet attributeSet){
        super(context);
    }


    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {

        float x = ev.getRawX();
        float y = ev.getRawY();

        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mDownPosX = x;
                mDownPosY = y;

                break;
            case MotionEvent.ACTION_MOVE:
                float deltaX = Math.abs(x - mDownPosX);
                float deltaY = Math.abs(y - mDownPosY);
                //横向滑动不拦截
                if (deltaX > deltaY) {
                    return false;
                }
                break;

            case MotionEvent.ACTION_UP:
                break;

        }

        return super.onInterceptTouchEvent(ev);
    }
}
  
  二、自定义子view ViewPager,复写dispathTouchEvent,判断是横向移动请求父类不要拦截，保证子类能收到。
  
  public class MyViewPager extends ViewPager {
    private static final String TAG = MyViewPager.class.getSimpleName();

    int lastX = -1;
    int lastY = -1;

    public MyViewPager(@NonNull Context context) {
        super(context);
    }

    /**
     * 必须要复写，不然在xml中引用该控件会报inflateException
     * @param context
     * @param attrs
     */
    public MyViewPager(@NonNull Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {

        int x = (int) ev.getRawX();
        int y = (int) ev.getRawY();
        int dealtX = 0;
        int dealtY = 0;

        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                dealtX = 0;
                dealtY = 0;
                // 保证子View能够接收到Action_move事件
                getParent().requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_MOVE:
                dealtX += Math.abs(x - lastX);
                dealtY += Math.abs(y - lastY);
                Log.i(TAG, "dealtX:=" + dealtX);
                Log.i(TAG, "dealtY:=" + dealtY);
                // 这里是够拦截的判断依据是左右滑动，读者可根据自己的逻辑进行是否拦截
                if (dealtX >= dealtY) {
                    getParent().requestDisallowInterceptTouchEvent(true);
                } else {
                    getParent().requestDisallowInterceptTouchEvent(false);
                }
                lastX = x;
                lastY = y;
                break;
            case MotionEvent.ACTION_CANCEL:
                break;
            case MotionEvent.ACTION_UP:
                break;

        }

        return super.dispatchTouchEvent(ev);
    }
}
  
  ```  
  2. ViewPager中嵌套ViewPager  
  ```
  重写子View的dispatchTouchEvent方法，在子view需要拦截的时候进行拦截，否则交给父view处理。  
  
  当当前页面处于第一页或者最后一页的时候触摸事件交给父view处理。  
  public class ChildViewPager extends ViewPager {

    private static final String TAG = "xujun";
    public ChildViewPager(Context context) {
        super(context);
    }

    public ChildViewPager(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        int curPosition;

        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                getParent().requestDisallowInterceptTouchEvent(true);
                break;
            case MotionEvent.ACTION_MOVE:
                curPosition = this.getCurrentItem();
                int count = this.getAdapter().getCount();
                Log.i(TAG, "curPosition:=" +curPosition);
                // 当当前页面在最后一页和第0页的时候，由父View拦截触摸事件
                if (curPosition == count - 1|| curPosition==0) {
                    getParent().requestDisallowInterceptTouchEvent(false);
                } else {//其他情况，由子ViewPager拦截触摸事件
                    getParent().requestDisallowInterceptTouchEvent(true);
                }

        }
        return super.dispatchTouchEvent(ev);
    }
}
  ```  
  
  ##### 写在最后  
  采用ScrollView中嵌套viewpager和RecyclerView的方式问题比较多，如在fragment中Recyclerview抢占焦点，用户体验不好，可采用以下方式替换：  
  1. 使用ListView或者RecyclerView添加headView来实现。  
  [鸿洋优雅的为RecyclerView添加headerView](http://blog.csdn.net/lmj623565791/article/details/51854533)
  2. 使用SupportLibrary中的CoordinatorLayout等控件  
  [使用CoordinaryLayout](https://blog.csdn.net/gdutxiaoxu/article/details/52858598)
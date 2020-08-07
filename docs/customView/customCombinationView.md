1. 第一步定义你要复用的布局
```
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <TextView
        android:id="@+id/tv_left_attribute"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/tv_order_goods_name"
        android:textSize="@dimen/edge_14"
        android:textColor="@color/color_3f3f3f"
        android:layout_marginTop="@dimen/edge_15"
        android:layout_centerVertical="true"/>

    <TextView
        android:id="@+id/tv_right_attribute"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textColor="@color/color_808080"
        android:textSize="@dimen/edge_14"
        android:layout_alignParentRight="true"
        android:layout_centerVertical="true"/>
</RelativeLayout>
```  

2. 第二步定义自定义属性(value下新建attrs)
```
<resources>
    <declare-styleable name="SimpleAttribute">
        <attr name="left_text_color" format="color"/>
        <attr name="left_text_size" format="dimension"/>
        <attr name="left_text" format="string"/>
        <attr name="right_text_color" format="color"/>
        <attr name="right_text_size" format="dimension"/>
        <attr name="right_text" format="string"/>
    </declare-styleable>
</resources>
```  
3. 第三步自定义一个View根据需求继承不同的viewGroup,如：RelativeLayout、LinearLayout等,复写两个构造方法  
```
public class SimpleAttribute extends RelativeLayout {
```  
4. 初始化控件  
```
    private void initView(Context context) {
        View.inflate(context, R.layout.attribute_layout, this);
        mLeftTv = this.findViewById(R.id.tv_left_attribute);
        mRightTv = this.findViewById(R.id.tv_right_attribute);
    }
```  

5. 重写构造方法，在构造方法中初始化控件，并引用自定义属性。  
```
    private void useCustomAttribute(Context context, AttributeSet attrs) {
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.SimpleAttribute);

        int leftTextColor = typedArray.getColor(R.styleable.SimpleAttribute_left_text_color, getResources().getColor(R.color.black));
        setLeftTextColor(leftTextColor);
        float leftTextSize = typedArray.getDimension(R.styleable.SimpleAttribute_left_text_size, getResources().getDimension(R.dimen.edge_14));
        setLeftTextSize(leftTextSize);
        String leftText = typedArray.getString(R.styleable.SimpleAttribute_left_text);
        setLeftText(leftText);

        int rightTextColor = typedArray.getColor(R.styleable.SimpleAttribute_right_text_color, getResources().getColor(R.color.black));
        setRightTextColor(rightTextColor);
        float rightTextSize = typedArray.getDimension(R.styleable.SimpleAttribute_right_text_size, getResources().getDimension(R.dimen.edge_14));
        setRightTextSize(rightTextSize);
        String rightText = typedArray.getString(R.styleable.SimpleAttribute_right_text);
        setRightText(rightText);

    }
```  



完整代码：  
```

/**
 * author : luis
 * e-mail : luis.gong@cardinfolink.com
 * date   : 2020/7/17  19:39
 * desc   :
 */
public class SimpleAttribute extends RelativeLayout {

    private TextView mLeftTv;
    private TextView mRightTv;

    public SimpleAttribute(Context context) {
        super(context);
        initView(context);
    }

    public SimpleAttribute(Context context, AttributeSet attrs) {
        super(context, attrs);
        useCustomAttribute(context, attrs);
    }

    private void initView(Context context) {
        View.inflate(context, R.layout.attribute_layout, this);
        mLeftTv = this.findViewById(R.id.tv_left_attribute);
        mRightTv = this.findViewById(R.id.tv_right_attribute);
    }


    private void useCustomAttribute(Context context, AttributeSet attrs) {
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.SimpleAttribute);

        int leftTextColor = typedArray.getColor(R.styleable.SimpleAttribute_left_text_color, getResources().getColor(R.color.black));
        setLeftTextColor(leftTextColor);
        float leftTextSize = typedArray.getDimension(R.styleable.SimpleAttribute_left_text_size, getResources().getDimension(R.dimen.edge_14));
        setLeftTextSize(leftTextSize);
        String leftText = typedArray.getString(R.styleable.SimpleAttribute_left_text);
        setLeftText(leftText);

        int rightTextColor = typedArray.getColor(R.styleable.SimpleAttribute_right_text_color, getResources().getColor(R.color.black));
        setRightTextColor(rightTextColor);
        float rightTextSize = typedArray.getDimension(R.styleable.SimpleAttribute_right_text_size, getResources().getDimension(R.dimen.edge_14));
        setRightTextSize(rightTextSize);
        String rightText = typedArray.getString(R.styleable.SimpleAttribute_right_text);
        setRightText(rightText);

    }

    public void setLeftTextSize(float leftTextSize) {
        mLeftTv.setTextSize(leftTextSize);
    }

    public void setLeftTextColor(int leftTextColor) {
        mLeftTv.setTextColor(leftTextColor);
    }

    public void setLeftText(String leftText){
        mLeftTv.setText(leftText);
    }

    public void setRightTextSize(float rightTextSize) {
        mRightTv.setTextSize(rightTextSize);
    }

    public void setRightTextColor(int rightTextColor) {
        mRightTv.setTextColor(rightTextColor);
    }

    public void setRightText(String rightText){
        mRightTv.setText(rightText);
    }
}
```
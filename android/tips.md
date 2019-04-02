# 小知识
## 自定义属性
* 1.首先，在values文件夹下创建一个attrs.xml文件，并声明<declare-styleable>
```xml
  <declare-styleable name="PageView">
    <attr name="autoPlay" format="boolean"/>
  </declare-styleable>
```
* 2.获取自定义属性值
```java
  TypedArray typedArray = context.obtainStyledAttributes(attrs,R.styleable.PageView,defStyleAttr,0);
  boolean autoPlay=typedArray.getBoolean(R.styleable.PageView_autoPlay,false);
  typedArray.recycle();
```
## 自定义textView
* 文本对齐
```java
  Layotu layout=getLayout();//得到textView的layout
  int lineNum=0;//第一行
  int lineBaseline=layout.getLineBaseline(lineNum)-getPaddingTop();//得到指定行的文本基准线
  int lineStart=layout.getLineStart(lineNum);//得到指定行的第一个字符前的文本偏移量
  int lineEnd=layout.getLineEnd(lineNum);//得到指定行的最后一个字符后的文本偏移量
  float width=StaticLayout.getDesiredWidth(text,lineStart,lineEnd,getPaint());//得到指定开始结束范围文本的长度
  Sting subText=text.subString(lineStart,lineEnd);
  //计算字符之间的宽度
  float space=(getMeasureWidth()-width-getPaddingLeft()-getPaddingRight())/length;
  float x=getPaddingLeft();
  //重新绘制所有字符
  for(int i=0;i<subText.length();i++){
    String c=String.valueOf(line.charAt(i));
    float cw=StaticLayout.getDesiredWidth(c,this.getPaint());
    canvas.drawText(c,x,lineBaseline,getPaint());
    x+=cw+d;
  }
  
```



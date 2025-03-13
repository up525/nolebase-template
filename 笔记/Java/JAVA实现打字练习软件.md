​转眼已经学了一学期的java了，老师让我们根据所学知识点写一个打字练习软件的综合练习。一开始我也不是很有思路，我找了一下发现csdn上关于这个小项目的代码也不算很多，所以我最后自己在csdn查了一些资料，写了这么一个简略版本的打字练习软件（本人菜鸟，大佬勿喷），现在我把我写这个小项目的心路历程进行一下简单的总结。

首先建立TypeFrame包并在包下建立如下类容：

![](https://i-blog.csdnimg.cn/blog_migrate/b9c4d8dbed104c8ad18f6c18ddbd3881.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​
第一步对界面的初始化：

```java
    public TypeFrame(){
        this.setBounds(600,300,950,600);
        this.setLayout(new FlowLayout());
        this.setDefaultCloseOperation(3);
        this.setResizable(false);
        this.setVisible(true);
    }
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

第二步对界面的组件进行初始化

1.在成员部分进行声明

```java
    JMenuBar Bar;
    JMenu menu;
    JMenuItem item1;
    JMenuItem item2;
    JMenuItem item3;

    JLabel label1;
    JLabel label2;
    JLabel label3;
    JLabel label4;
    JLabel label5;
    JLabel label6;

    JTextField text1;
    JTextField text2;
    JTextField text3;
    JTextField text4;
    JTextField text5;

    JTextArea textArea1;
    JTextArea textArea2;

    int CorrectNum=0;//打字正确数
    int ErrorNum=0;//打字错误数
    int TypeNum=0;//打字总数

    Timer  time;
    int Time=0;//打字时间

    int v=0;//打字速度

    JFileChooser  chooser;
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

 2.在init函数内对其进行初始化

```java
 public void init(){
        Bar=new JMenuBar();
        menu=new JMenu("菜单");
        item1=new JMenuItem("导入文本");
        item1.addActionListener(this);
        item2=new JMenuItem("保存");
        item2.addActionListener(this);
        item3=new JMenuItem("退出");
        item3.addActionListener(this);

        Bar.add(menu);
        menu.add(item1);
        menu.add(item2);
        menu.add(item3);
        this.getContentPane().add(Bar);



        label1=new JLabel("用时：");
        label2=new JLabel("总字数：");
        label3=new JLabel("正确：");
        label4=new JLabel("错误：");
        label5=new JLabel("每分钟：");
        label6=new JLabel("个字");


        text1=new JTextField(10);
        text1.setHorizontalAlignment(JTextField.CENTER);//设置文本居中显示
        text2=new JTextField(10);
        text2.setHorizontalAlignment(JTextField.CENTER);
        text3=new JTextField(10);
        text3.setHorizontalAlignment(JTextField.CENTER);
        text4=new JTextField(10);
        text4.setHorizontalAlignment(JTextField.CENTER);
        text5=new JTextField(10);
        text5.setHorizontalAlignment(JTextField.CENTER);

        time =new Timer(1000, this);
        time.start();

        //这里比较难--既要控制确定的行数和列数还要保准写入的数字不会缩进
        // 是否自动换行，默认为 false
        //void setLineWrap(boolean wrap)
        //row--行数 columns--列数
        textArea1 = new JTextArea(11, 80);
        textArea1.setLineWrap(true);
        textArea1.setFont(new Font("隶书",Font.BOLD,20));
        textArea1.append("abcdefghijklmnopqrstuvwsyz");

        textArea2 = new JTextArea(11, 80);
        textArea2.setLineWrap(true);
        textArea2.setFont(new Font("隶书",Font.BOLD,20));
        textArea2.addKeyListener(this);
        try {
            BufferedReader br=new BufferedReader(new FileReader("C:\\Users\\略略略\\IdeaProjects\\out\\TypeFrame\\src\\a.txt"));
            String str;
            textArea2.setText(null);
            while ((str=br.readLine())!=null){
                textArea2.append(str);
            }
        } catch (IOException ex) {
            throw new RuntimeException(ex);
        }



        //如何对textArea进行监听？

        chooser=new JFileChooser();


        this.add(label1);
        this.add(text1);
        this.add(label2);
        this.add(text2);
        this.add(label3);
        this.add(text3);
        this.add(label4);
        this.add(text4);
        this.add(label5);
        this.add(text5);
        this.add(label6);

        this.getContentPane().add(textArea1);
        this.getContentPane().add(textArea2);

        this.setVisible(true);
    }
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

 第三步对TextArea，item1,2,3组件进行监听

```java
textArea2.addKeyListener(this);
        try {
            BufferedReader br=new BufferedReader(new FileReader("C:\\Users\\略略略\\IdeaProjects\\out\\TypeFrame\\src\\a.txt"));
            String str;
            textArea2.setText(null);
            while ((str=br.readLine())!=null){
                textArea2.append(str);
            }
        } catch (IOException ex) {
            throw new RuntimeException(ex);
        }
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

```java
  @Override
    public void actionPerformed(ActionEvent e) {
        if(e.getSource()==item1){
            InitData();
            Time=0;
            InitText();
            textArea2.setText(null);
            int result = chooser.showOpenDialog(null);
            // 如果用户选择了文件或目录，获取用户所选的文件路径并输出
            if (result == JFileChooser.APPROVE_OPTION) {
                String filePath = chooser.getSelectedFile().getAbsolutePath();
                try {
                    BufferedReader br=new BufferedReader(new FileReader(filePath));
                    String str;
                    textArea1.setText(null);
                    while ((str=br.readLine())!=null){
                        textArea1.append(str);
                    }
                    Time=0;
                } catch (IOException ex) {
                    throw new RuntimeException(ex);
                }
                // System.out.println("用户选择的文件路径为：" + filePath);
            }
        }else if(e.getSource()==item2){
            String str=textArea2.getText();
            try {
                BufferedWriter bw=new BufferedWriter(new FileWriter("C:\\Users\\略略略\\IdeaProjects\\out\\TypeFrame\\src\\a.txt"));
                bw.write(str);
                bw.close();//不关流是保存不了文本的
            } catch (IOException ex) {
                throw new RuntimeException(ex);
            }
            System.out.println(str);
            System.out.println("保存成功");
        } else if(e.getSource()==item3){
            System.exit(0);
        }


        Time++;
        text1.setText(Time+"");

        v=TypeNum*60/Time;
        text5.setText(v+"");

    }
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

补充一下 InitData()，InitText()两个函数

```java
    public void InitText() {
        text2.setText(TypeNum+"");
        text3.setText(CorrectNum+"");
        text4.setText(ErrorNum+"");
        text5.setText(v+"");
    }

    public void InitData() {
        CorrectNum=0;
        ErrorNum=0;
        TypeNum=0;
        v=0;
    }
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

完整代码如下：

```java
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;
import java.io.*;

public class TypeFrame extends JFrame implements ActionListener, KeyListener {
    JMenuBar Bar;
    JMenu menu;
    JMenuItem item1;
    JMenuItem item2;
    JMenuItem item3;

    JLabel label1;
    JLabel label2;
    JLabel label3;
    JLabel label4;
    JLabel label5;
    JLabel label6;

    JTextField text1;
    JTextField text2;
    JTextField text3;
    JTextField text4;
    JTextField text5;

    JTextArea textArea1;
    JTextArea textArea2;

    int CorrectNum=0;//打字正确数
    int ErrorNum=0;//打字错误数
    int TypeNum=0;//打字总数

    Timer  time;
    int Time=0;//打字时间

    int v=0;//打字速度

    JFileChooser  chooser;


    public TypeFrame(){
        this.setBounds(600,300,950,600);
        this.setLayout(new FlowLayout());
        this.setDefaultCloseOperation(3);
        this.setResizable(false);
        this.setVisible(true);
    }

    public void init(){
        Bar=new JMenuBar();
        menu=new JMenu("菜单");
        item1=new JMenuItem("导入文本");
        item1.addActionListener(this);
        item2=new JMenuItem("保存");
        item2.addActionListener(this);
        item3=new JMenuItem("退出");
        item3.addActionListener(this);

        Bar.add(menu);
        menu.add(item1);
        menu.add(item2);
        menu.add(item3);
        this.getContentPane().add(Bar);



        label1=new JLabel("用时：");
        label2=new JLabel("总字数：");
        label3=new JLabel("正确：");
        label4=new JLabel("错误：");
        label5=new JLabel("每分钟：");
        label6=new JLabel("个字");


        text1=new JTextField(10);
        text1.setHorizontalAlignment(JTextField.CENTER);//设置文本居中显示
        text2=new JTextField(10);
        text2.setHorizontalAlignment(JTextField.CENTER);
        text3=new JTextField(10);
        text3.setHorizontalAlignment(JTextField.CENTER);
        text4=new JTextField(10);
        text4.setHorizontalAlignment(JTextField.CENTER);
        text5=new JTextField(10);
        text5.setHorizontalAlignment(JTextField.CENTER);

        time =new Timer(1000, this);
        time.start();

        //这里比较难--既要控制确定的行数和列数还要保准写入的数字不会缩进
        // 是否自动换行，默认为 false
        //void setLineWrap(boolean wrap)
        //row--行数 columns--列数
        textArea1 = new JTextArea(11, 80);
        textArea1.setLineWrap(true);
        textArea1.setFont(new Font("隶书",Font.BOLD,20));
        textArea1.append("abcdefghijklmnopqrstuvwsyz");

        textArea2 = new JTextArea(11, 80);
        textArea2.setLineWrap(true);
        textArea2.setFont(new Font("隶书",Font.BOLD,20));
        textArea2.addKeyListener(this);
        try {
            BufferedReader br=new BufferedReader(new FileReader("C:\\Users\\略略略\\IdeaProjects\\out\\TypeFrame\\src\\a.txt"));
            String str;
            textArea2.setText(null);
            while ((str=br.readLine())!=null){
                textArea2.append(str);
            }
        } catch (IOException ex) {
            throw new RuntimeException(ex);
        }



        //如何对textArea进行监听？

        chooser=new JFileChooser();


        this.add(label1);
        this.add(text1);
        this.add(label2);
        this.add(text2);
        this.add(label3);
        this.add(text3);
        this.add(label4);
        this.add(text4);
        this.add(label5);
        this.add(text5);
        this.add(label6);

        this.getContentPane().add(textArea1);
        this.getContentPane().add(textArea2);

        this.setVisible(true);
    }


    @Override
    public void actionPerformed(ActionEvent e) {
        if(e.getSource()==item1){
            InitData();
            Time=0;
            InitText();
            textArea2.setText(null);
            int result = chooser.showOpenDialog(null);
            // 如果用户选择了文件或目录，获取用户所选的文件路径并输出
            if (result == JFileChooser.APPROVE_OPTION) {
                String filePath = chooser.getSelectedFile().getAbsolutePath();
                try {
                    BufferedReader br=new BufferedReader(new FileReader(filePath));
                    String str;
                    textArea1.setText(null);
                    while ((str=br.readLine())!=null){
                        textArea1.append(str);
                    }
                    Time=0;
                } catch (IOException ex) {
                    throw new RuntimeException(ex);
                }
                // System.out.println("用户选择的文件路径为：" + filePath);
            }
        }else if(e.getSource()==item2){
            String str=textArea2.getText();
            try {
                BufferedWriter bw=new BufferedWriter(new FileWriter("C:\\Users\\略略略\\IdeaProjects\\out\\TypeFrame\\src\\a.txt"));
                bw.write(str);
                bw.close();//不关流是保存不了文本的
            } catch (IOException ex) {
                throw new RuntimeException(ex);
            }
            System.out.println(str);
            System.out.println("保存成功");
        } else if(e.getSource()==item3){
            System.exit(0);
        }


        Time++;
        text1.setText(Time+"");

        v=TypeNum*60/Time;
        text5.setText(v+"");

    }

    public void InitText() {
        text2.setText(TypeNum+"");
        text3.setText(CorrectNum+"");
        text4.setText(ErrorNum+"");
        text5.setText(v+"");
    }

    public void InitData() {
        CorrectNum=0;
        ErrorNum=0;
        TypeNum=0;
        v=0;
    }


    @Override
    public void keyTyped(KeyEvent e) {

    }

    @Override
    public void keyPressed(KeyEvent e) {
        if(textArea2.getText().length()<=textArea1.getText().length()){
            InitData();
            for (int i=0;i<textArea2.getText().length();i++){
                if(textArea2.getText().charAt(i)==textArea1.getText().charAt(i)){
                    CorrectNum++;
                    TypeNum++;
                    text2.setText(TypeNum+"");
                    text3.setText(CorrectNum+"");

                }else{
                    ErrorNum++;
                    TypeNum++;
                    text2.setText(TypeNum+"");
                    text4.setText(ErrorNum+"");
                }
            }
        }else{
            JOptionPane.showMessageDialog(null, "已超出数字范围！", "提示",JOptionPane.PLAIN_MESSAGE);
        }
        InitText();
    }

    @Override
    public void keyReleased(KeyEvent e) {

    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

```java
public class APP {
    public static void main(String[] args) {
        TypeFrame app=new TypeFrame();
        app.init();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")

 a.txt

功能简介

![](https://i-blog.csdnimg.cn/blog_migrate/0f31f5eb2c65be21f2cf4c5946293796.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

导入文本--可以向第一个文本写入数据

保存--可以把“我”写的文本保存进入a.txt中，下一次打开这个软件时会自动写入保存在a.txt文本的内容。

退出--点击退出，app会直接关闭。

![](https://i-blog.csdnimg.cn/blog_migrate/7c1891164b4ba2d1a275e5835afd0c02.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

事先在桌面建立1.txt文本，并向里面写入数据

 点击第二个Desktop

 ![](https://i-blog.csdnimg.cn/blog_migrate/0717629138b52960945daffe31b8a38f.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

 ![](https://i-blog.csdnimg.cn/blog_migrate/9c05dfb31267ae1288b17855b9b47ccb.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

![](https://i-blog.csdnimg.cn/blog_migrate/6ce3d5f0498ae86be1c512b68ef3d29a.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

 ![](https://i-blog.csdnimg.cn/blog_migrate/8af92e8e9dede924ec472586f7844e84.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

![](https://i-blog.csdnimg.cn/blog_migrate/4a5406db4b05b3c153527db33ef6c47a.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​

 希望看到最后的你，如果觉得这篇文章对你有一点点帮助或者启示的话，求点赞，求收藏，求关注，谢谢啦！！！

  

​
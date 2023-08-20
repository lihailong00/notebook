# Java实现http服务器

[toc]



## Data

```java
package com.lee.httpd.data;

import java.io.File;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @author lhl
 */
public class Data {
    // 表示服务器是否可以正常工作
    public static boolean isRun = true;

    // 服务是否是暂停状态
    public static boolean isPush = false;

    // 存储服务器可以获取的静态资源的位置
    public static String resoursePath = new File("").getAbsolutePath() + "\\src\\main\\resources";

    public static final Map<String, String> DOC_HEADLINE = new HashMap<>();

    public static final List<String> IMAGE = new ArrayList<>();
    static {
        DOC_HEADLINE.put("html", "Content-Type: text/html;charset=utf-8");
        DOC_HEADLINE.put("css", "Content-Type: text/css");
        DOC_HEADLINE.put("js", "Content-Type: application/x-javascript");
        DOC_HEADLINE.put("json", "Content-Type: application/json;charset=utf-8");

        IMAGE.add("jpg");
        IMAGE.add("png");
        IMAGE.add("gif");
    }
}

```



## RequestExecute

```java
package com.lee.httpd.service;

import com.lee.httpd.data.Data;

import java.io.*;
import java.net.Socket;
import java.nio.charset.StandardCharsets;

/**
 * @description 请求处理线程类
 * @author lhl
 */
public class RequestExecute extends Thread {
    /**
     * 将socket定义为成员变量，利用构造方法初始化
     */
    private Socket socket;

    public RequestExecute(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run() {
        try {
            /**
             * 从socket中取出输入流，然后从输入流中取出数据
             * 将字节输入流转换为缓冲字符输入流
             */
            InputStream inputStream = socket.getInputStream();
            InputStreamReader inputStreamReader = new InputStreamReader(inputStream);
            BufferedReader bufferedReader = new BufferedReader(inputStreamReader);

            /**
             * 声明输出流，指向客户端，将字节输出流包装成字符流
             */
            OutputStream outputStream = socket.getOutputStream();
            PrintWriter printWriter = new PrintWriter(outputStream);

            // 判断服务器是否处于暂停状态
            if (Data.isPush) {
                printWriter.println("HTTP/1.1 200 OK");
                printWriter.println("Content-Type: text/html;charset=utf-8");
                // 输出空行
                printWriter.println();
                printWriter.println("<h1>服务器正处于暂停状态~</h1>");
                printWriter.flush();
                return;
            }

            // 循环从字符流中获取字符
            String line = null;
            // 记录请求行数
            int lineNum = 1;
            // 存储请求路径
            String reqPath = "";
            // 记录服务器ip
            String host = "";
            while ((line = bufferedReader.readLine()) != null) {
//                System.out.println(line);
                if (lineNum == 1) {
                    String[] infos = line.split(" ");
                    if (infos.length > 2) {
                        reqPath = infos[1];
                    }
                    else {
                        throw new RuntimeException("请求行解析错误");
                    }
                }
                else {
                    // 解析其他行，主要解析第2行
                    String[] infos = line.split(": ");
                    if ("Host".equals(infos[0])) {
                        host = infos[1];
                    }
                }
                // 读取到空行，需要退出。因为http请求是长连接，无法读取到文件末尾
                if ("".equals(line)) {
                    break;
                }
                lineNum++;
            }

            // 输出请求地址
//            System.out.println("处理请求：http://" + host + reqPath);

            // 根据请求响应客户端
            // 如果路径是 / 直接返回 hello,world!
            // 如果路径包含文件名，则返回对应文件
            if ("/".equals(reqPath)) {
                // 输出响应行
                printWriter.println("HTTP/1.1 200 OK");
                printWriter.println("Content-Type: text/html;charset=utf-8");
                // 输出空行
                printWriter.println("");
                // 响应头结束，开始响应内容
                printWriter.println("<h1>hello, world!</h1>");
                printWriter.flush();
            }
            else {
                // 查找对应的资源（html,css,js,jpg...）
                StringBuilder filePath = new StringBuilder(Data.resoursePath);
                String[] folders = reqPath.split("/");
                String ext = reqPath.substring(reqPath.lastIndexOf(".") + 1);
                for (int i = 1; i < folders.length; i++) {
                    String name = folders[i];
                    filePath.append("\\").append(name);
                }
                File file = new File(filePath.toString());
                if (file.isFile()) {
                    response200(outputStream, filePath.toString(), ext);
                }
                else {
                    response404(outputStream);
                }
            }
            inputStream.close();
            inputStreamReader.close();
            bufferedReader.close();
            printWriter.close();
            outputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 将指定的文件放入输出流中
     * @param outputStream
     * @param filePath 文件路径
     * @param ext 文件后缀
     */
    private void response200(OutputStream outputStream, String filePath, String ext) {
        InputStream inputStream = null;
        PrintWriter printWriter = null;
        InputStreamReader inputStreamReader = null;
        BufferedReader bufferedReader = null;
        try {
            inputStream = new FileInputStream(filePath);
            outputStream.write("HTTP/1.1 200 OK\r\n".getBytes(StandardCharsets.UTF_8));
            // 图片采用字节输出流
            if (Data.IMAGE.contains(ext)) {
                outputStream.write(("Content-Type: image/" + ext + "\r\n").getBytes());
                // 输出空行，表示响应头已结束
                outputStream.write("\r\n".getBytes());
                int len = -1;
                byte[] buff = new byte[1024];
                while ((len = inputStream.read(buff)) != -1) {
                    outputStream.write(buff, 0, len);
                    outputStream.flush();
                }
            }
            else if (Data.DOC_HEADLINE.containsKey(ext)) {
                // 文档采用字符输出流
                printWriter = new PrintWriter(outputStream);
                printWriter.println("HTTP/1.1 200 OK");
                printWriter.println(Data.DOC_HEADLINE.get(ext));
                // 输出空行，表示响应头已结束
                printWriter.println("");
                // 初始化输入流
                inputStreamReader = new InputStreamReader(inputStream);
                bufferedReader = new BufferedReader(inputStreamReader);
                // 写出数据
                String line;
                while ((line = bufferedReader.readLine()) != null) {
                    printWriter.println(line);
                    printWriter.flush();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (inputStream != null) {
                    inputStream.close();
                }
                if (printWriter != null) {
                    printWriter.close();
                }
                if (inputStreamReader != null) {
                    inputStreamReader.close();
                }
                if (bufferedReader != null) {
                    bufferedReader.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }

    /**
     * 响应404未找到界面
     * @param outputStream
     */
    private void response404(OutputStream outputStream) {
        PrintWriter printWriter = new PrintWriter(outputStream);
        // 输出响应行
        printWriter.println("HTTP/1.1 404");
        printWriter.println("Content-Type: text/html;charset=utf-8");
        // 输出空行
        printWriter.println("");  // 响应头结束，开始响应内容
        printWriter.println("<h1>sorry~404!</h1>");
        printWriter.flush();
        printWriter.close();
        try {
            outputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```



## Server

```java
package com.lee.httpd.service;

import com.lee.httpd.data.Data;
import com.lee.httpd.ul.MainFrame;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @description 服务监听
 * @author lhl
 */
public class Server implements Runnable {
    /**
     * 监听端口，默认端口为8888
     */
    private int port = 8888;

    private MainFrame mainFrame;

    public Server(int port, MainFrame mainFrame) {
        this.port = port;
        this.mainFrame = mainFrame;
    }

    @Override
    public void run() {
        try {
            ServerSocket serverSocket = new ServerSocket(port);
            // 开始监听，输出日志
            mainFrame.printLog("========开始监听服务=======");
            mainFrame.printLog("监听端口:" + port);
            mainFrame.printLog("静态资源路径是：" + Data.resoursePath);
            while (Data.isRun) {
                Socket socket = serverSocket.accept();
                // 将socket交给RequestExecute处理
                RequestExecute requestExecute = new RequestExecute(socket);
                System.out.println("开启线程：" + requestExecute.getName());
                System.out.println("当前活跃线程数量：" + Thread.activeCount());
                requestExecute.start();
            }
            serverSocket.close();
            mainFrame.printLog("======服务停止监听======");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```



## MainFrame

```java
package com.lee.httpd.ul;

import com.lee.httpd.data.Data;
import com.lee.httpd.service.Server;

import javax.swing.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.File;
import java.io.IOException;
import java.net.Socket;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @author lhl
 */
public class MainFrame extends JFrame {
    private JLabel labPort;
    private JLabel labInfo;
    private JLabel labPath;
    private JTextField textPort;
    private JTextField textPath;
    private JButton btnStartServer;
    private JButton btnPushServer;
    private JButton btnStopServer;
    private JButton btnSetPath;
    private JPanel contentPanel;
    private JScrollPane scrollPane;
    private JTextArea textArea;

    public MainFrame() {
        init();
    }

    private void init() {
        this.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
        this.setBounds(200, 200, 800, 500);
        this.setTitle("海龙Http服务器");
        this.setResizable(false);

        // 设置主面板
        contentPanel = new JPanel();
        contentPanel.setLayout(null);
        this.setContentPane(contentPanel);

        // 设置端口
        labPort = new JLabel("监听端口号");
        labPort.setBounds(15, 10, 100, 25);
        contentPanel.add(labPort);

        textPort = new JTextField("8888");
        textPort.setBounds(120, 10, 150, 25);
        contentPanel.add(textPort);

        // 三个按钮
        btnStartServer = new JButton("启动服务");
        btnStartServer.setBounds(300, 10, 120, 25);
        btnStartServer.setEnabled(true);
        contentPanel.add(btnStartServer);

        btnPushServer = new JButton("暂停服务");
        btnPushServer.setBounds(440, 10, 120, 25);
        btnPushServer.setEnabled(false);
        contentPanel.add(btnPushServer);

        btnStopServer = new JButton("停止服务");
        btnStopServer.setBounds(580, 10, 120, 25);
        btnStopServer.setEnabled(false);
        contentPanel.add(btnStopServer);

        // 资源路径设置
        labPath = new JLabel("资源路径设置：");
        // 设置默认路径
        labPath.setText(Data.resoursePath);
        labPath.setBounds(15, 45, 100, 25);
        contentPanel.add(labPath);

        textPath = new JTextField("");
        textPath.setBounds(130, 45, 500, 25);
        contentPanel.add(textPath);

        btnSetPath = new JButton("设置资源位置");
        btnSetPath.setBounds(640, 45, 120, 25);
        contentPanel.add(btnSetPath);

        // 控制台
        textArea = new JTextArea("--控制台--\r\n");
        textArea.setLineWrap(true);
        scrollPane = new JScrollPane(textArea,
                JScrollPane.VERTICAL_SCROLLBAR_ALWAYS,
                JScrollPane.HORIZONTAL_SCROLLBAR_ALWAYS);
        scrollPane.setBounds(15, 80, 770, 350);
        contentPanel.add(scrollPane);

        // 设置资源文件夹按钮事件
        btnSetPath.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                JFileChooser jFileChooser = new JFileChooser();
                jFileChooser.setFileSelectionMode(JFileChooser.DIRECTORIES_ONLY);
                int op = jFileChooser.showDialog(MainFrame.this, "请选择静态资源文件夹");
                if (op == JFileChooser.APPROVE_OPTION) {
                    File file = jFileChooser.getSelectedFile();
                    // 取出绝对路径
                    String filePath = file.getAbsolutePath();
                    textPath.setText(filePath);
                    // 修改Data中的静态变量
                    Data.resoursePath = filePath;
                }
            }
        });

        // 启动按钮事件
        btnStartServer.addActionListener(e -> {
            Data.isRun = true;
            Data.isPush = false;
            // 获取端口
            int port = 8888;
            try {
                port = new Integer(textPort.getText().trim());
            } catch (Exception ex) {
                ex.printStackTrace();
            }
            // 创建server
            Server server = new Server(port, MainFrame.this);
            new Thread(server).start();
            // 修改按钮的状态
            btnStartServer.setEnabled(false);
            btnStopServer.setEnabled(true);
            btnPushServer.setEnabled(true);
        });

        // 暂停按钮事件
        btnPushServer.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                if (Data.isPush) {
                    Data.isPush = false;
                    btnPushServer.setText("暂停服务");
                    printLog("服务器继续运行");
                }
                else {
                    Data.isPush = true;
                    btnPushServer.setText("继续运行");
                    printLog("服务器暂停运行");
                }
            }
        });

        // 停止按钮事件
        btnStopServer.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                Data.isRun = false;
                Data.isPush = false;

                // 获取端口
                int port = 8888;
                try {
                    port = new Integer(textPort.getText().trim());
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
                // 修改后，服务仍会监听一次。此时主动向服务器发送一次请求，结束程序。
                try {
                    // 此时会自定请求一次服务器
                    Socket socket = new Socket("127.0.0.1", port);
                    socket.close();
                } catch (IOException ex) {
                    ex.printStackTrace();
                }

                // 修改按钮状态
                btnStartServer.setEnabled(true);
                btnPushServer.setEnabled(false);
                btnStopServer.setEnabled(false);
            }
        });

        this.setVisible(true);
    }

    public void printLog(final String msg) {
        new Thread() {
            @Override
            public void run() {
                String date = new SimpleDateFormat("[yyyy0MM0dd HH:mm:ss]").format(new Date());
                // 可使用StringBuffer优化
                String info  = textArea.getText() + date + msg + "\r\n";
                textArea.setText(info);
            }
        }.start();
    }

    public static void main(String[] args) {
        MainFrame mainFrame = new MainFrame();
    }
}

```


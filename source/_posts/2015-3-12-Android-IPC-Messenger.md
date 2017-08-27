---
title: Android-IPC-Messenger
date: 2015-03-12 16:38:10
tags:
- Android
- IPC
- Messenger
categories:
- Work
- Technology
- Note
---

# Messenger:信使

> 官方文档解释：它引用了一个Handler对象，以便others能够向它发送消息(使用mMessenger.send(Message msg)方法)。该类允许跨进程间基于Message通信(即两个进程间可以通过Message进行通信)。简单理解，就是在服务端使用Handler创建一个Messenger，客户端持有这个Messenger就可以与服务端通信了。

以前我们使用Handler+Message的方式进行通信，都是在同一个进程中，从线程持有一个主线程的Handler对象，并向主线程发送消息。

Android可以使用bindler机制进行跨进行通信，所以我们就可以将Handler与bindler结合起来进行跨进程发送消息。查看API就可以发现，Messenger就是通过这种方式的实现。

一般使用方法如下：

1. 远程通过
> mMessenger = new Messenger(mHandler)
创建一个信使对象

2. 客户端使用bindlerService请求连接远程

3. 远程onBind方法返回一个bindler
> return mMessenger.getBinder();

4. 客户端使用远程返回的bindler得到一个信使（即得到远程信使）
```
public void onServiceConnected(ComponentName name, IBinder service) {
     rMessenger = new Messenger(service);    
     ......
}
```
这里虽然是new了一个Messenger，但我们查看它的实现
public Messenger(IBinder target) { mTarget = IMessenger.Stub.asInterface(target); }

发现它的mTarget是通过AIDL得到的，实际上就是远程创建的那个。

5. 客户端可以使用这个远程信使对象向远程发送消息：
> rMessenger.send(msg);

这样远程服务端的Handler对象就能收到消息了，然后可以在其handlerMessage(Message msg)方法中进行处理。（该Handler对象就是第一步服务端创建Messenger时使用的参数mHandler）。

经过这5个步骤貌似只有客户端向服务端发送消息，这样的消息传递是单向的，那么如何实现双向传递呢？

首先需要在第5步稍加修改，在send(msg)前通过msm.replyTo = mMessenger将自己的信使设置到消息中，这样服务端接收到消息时同时也得到了客户端的信使对象了，然后服务端可以通过得到客户端的信使对象，并向它发送消息。
> cMessenger = msg.replyTo;
cMessenger.send(message);

即完成了从服务端向客户端发送消息的功能，这样客服端可以在自己的Handler对象的handlerMessage方法中接收服务端发送来的message进行处理。

双向通信宣告完成。

# Messenger工作原理

![Messenger的工作原理](/images/Android/Messenger_Work_Principle.png)

# 以下代码来自ApiDemo
Service code:
```
public class MessengerService extends Service {  
    /** For showing and hiding our notification. */  
    NotificationManager mNM;  
    /** Keeps track of all current registered clients. */  
    ArrayList<Messenger> mClients = new ArrayList<Messenger>();  
    /** Holds last value set by a client. */  
    int mValue = 0;  

    /**
     * Command to the service to register a client, receiving callbacks
     * from the service.  The Message's replyTo field must be a Messenger of
     * the client where callbacks should be sent.
     */  
    static final int MSG_REGISTER_CLIENT = 1;  

    /**
     * Command to the service to unregister a client, ot stop receiving callbacks
     * from the service.  The Message's replyTo field must be a Messenger of
     * the client as previously given with MSG_REGISTER_CLIENT.
     */  
    static final int MSG_UNREGISTER_CLIENT = 2;  

    /**
     * Command to service to set a new value.  This can be sent to the
     * service to supply a new value, and will be sent by the service to
     * any registered clients with the new value.
     */  
    static final int MSG_SET_VALUE = 3;  

    /**
     * Handler of incoming messages from clients.
     */  
    class IncomingHandler extends Handler {  
        @Override  
        public void handleMessage(Message msg) {  
            switch (msg.what) {  
                case MSG_REGISTER_CLIENT:  
                    mClients.add(msg.replyTo);  
                    break;  
                case MSG_UNREGISTER_CLIENT:  
                    mClients.remove(msg.replyTo);  
                    break;  
                case MSG_SET_VALUE:  
                    mValue = msg.arg1;  
                    for (int i = mClients.size() - 1; i >= 0; i --) {  
                        try {  
                            mClients.get(i).send(Message.obtain(null,  
                                    MSG_SET_VALUE, mValue, 0));  
                        } catch (RemoteException e) {  
                            // The client is dead.  Remove it from the list;   
                            // we are going through the list from back to front   
                            // so this is safe to do inside the loop.   
                            mClients.remove(i);  
                        }  
                    }  
                    break;  
                default:  
                    super.handleMessage(msg);  
            }  
        }  
    }  

    /**
     * Target we publish for clients to send messages to IncomingHandler.
     */  
    final Messenger mMessenger = new Messenger(new IncomingHandler());  

    @Override  
    public void onCreate() {  
        mNM = (NotificationManager)getSystemService(NOTIFICATION_SERVICE);  

        // Display a notification about us starting.   
        showNotification();  
    }  

    @Override  
    public void onDestroy() {  
        // Cancel the persistent notification.   
        mNM.cancel(R.string.remote_service_started);  

        // Tell the user we stopped.   
        Toast.makeText(this, R.string.remote_service_stopped, Toast.LENGTH_SHORT).show();  
    }  

    /**
     * When binding to the service, we return an interface to our messenger
     * for sending messages to the service.
     */  
    @Override  
    public IBinder onBind(Intent intent) {  
        return mMessenger.getBinder();  
    }  

    /**
     * Show a notification while this service is running.
     */  
    private void showNotification() {  
        // In this sample, we'll use the same text for the ticker and the expanded notification   
        CharSequence text = getText(R.string.remote_service_started);  

        // Set the icon, scrolling text and timestamp   
        Notification notification = new Notification(R.drawable.stat_sample, text,  
                System.currentTimeMillis());  

        // The PendingIntent to launch our activity if the user selects this notification   
        PendingIntent contentIntent = PendingIntent.getActivity(this, 0,  
                new Intent(this, Controller.class), 0);  

        // Set the info for the views that show in the notification panel.   
        notification.setLatestEventInfo(this, getText(R.string.remote_service_label),  
                       text, contentIntent);  

        // Send the notification.   
        // We use a string id because it is a unique number.  We use it later to cancel.   
        mNM.notify(R.string.remote_service_started, notification);  
    }  
}
```

# 自己的DEMO：
1. MyActivity Code
```
public class MyActivity extends Activity {

    protected static final int HELLO_CLIENT = 0;
    private MyServiceConnection conn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    protected void onStart() {
        super.onStart();
        Intent service = new Intent(this, MyService.class);
        conn = new MyServiceConnection();
        bindService(service, conn, BIND_AUTO_CREATE);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(conn);
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }

    private Handler handler = new Handler() {
        public void handleMessage(Message msg) {
            switch(msg.what) {
            case HELLO_CLIENT:
                Log.d("wangyanan", ">>>>>>>>>> hello client <<<<<<<<<<");
                break;
            }
        };
    };

    private Messenger clientMessenger;
    private Messenger serviceMessenger;

    class MyServiceConnection implements ServiceConnection {

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {

            clientMessenger = new Messenger(handler);
            serviceMessenger = new Messenger(service);
            sendServiceMessage(MyService.HELLO_SERVICE);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    }

    public void sendServiceMessage(int what) {
        try {
            Message message = Message.obtain(null, MyService.HELLO_SERVICE);
            message.replyTo = clientMessenger;
            serviceMessenger.send(message);
        } catch (RemoteException e) {
            e.printStackTrace();
            Toast.makeText(MyActivity.this, "与服务器连接失败", Toast.LENGTH_SHORT).show();
        }
    }
}
```

2. MyService Code
```
public class MyService extends Service {

    protected static final int HELLO_SERVICE = 0;

    @Override
    public IBinder onBind(Intent intent) {
        Log.d("wangyanan", ">>>>>>>>>> MyService onBind......");
        return serviceMessenger.getBinder();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.d("wangyanan", ">>>>>>>>>> MyService onStartComand......");
        return super.onStartCommand(intent, flags, startId);
    }

    private Handler handler = new Handler() {

        public void handleMessage(Message msg) {

            switch(msg.what) {
            case HELLO_SERVICE:
                Log.d("wangyanan", ">>>>>>>>>> hello service <<<<<<<<<<");
                clientMessenger = msg.replyTo;
                sendClientMessage(MyActivity.HELLO_CLIENT);
                break;
            }
        }
    };

    private Messenger serviceMessenger = new Messenger(handler);
    private Messenger clientMessenger = new Messenger(handler);

    private void sendClientMessage(int what) {
        try {
            Message message = Message.obtain(null, what);
            clientMessenger.send(message);
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    };
}
```

3. Log打印结果
```
07-27 10:46:54.049: D/wangyanan(8876): >>>>>>>>>> MyService onBind......
07-27 10:46:54.220: D/wangyanan(8876): >>>>>>>>>> hello service <<<<<<<<<<
07-27 10:46:54.220: D/wangyanan(8876): >>>>>>>>>> hello client <<<<<<<<<<
```

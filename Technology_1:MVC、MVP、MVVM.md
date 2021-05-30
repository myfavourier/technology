# MVC、MVP、MVVM

## MVC

### 概述

mvc分别指model、view、controller

视图层(View) 对应于 xml 布局文件和 java 代码动态 view 部分
控制层(Controller) MVC 中 Android 的控制层是由 Activity 来承担的,Activity 本来
主要是作为初始化页面,展示数据的操作,但是因为 XML 视图功能太弱,所以
Activity 既要负责视图的显示又要加入控制逻辑,承担的功能过多。
模型层(Model) 针对业务模型,建立的数据结构和相关的类,它主要负责网络请
求,数据库处理,I/O 的操作。

### 模型图

![](/home/alecs/桌面/(FDP{M}LH~_BR@[Z70`E_CG.png)

### Sample

```
public class MvcLoginActivity extends AppCompatActivity {
    private EditText userNameEt;
    private EditText passwordEt;
    private User user;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_mvc_login);
 
        user = new User();
        userNameEt = findViewById(R.id.user_name_et);
        passwordEt = findViewById(R.id.password_et);
        Button loginBtn = findViewById(R.id.login_btn);
 
        loginBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                login(userNameEt.getText().toString(), passwordEt.getText().toString());
            }
        });
    }
 
    private void login(String userName, String password) {
        if (userName.equals("jere") && password.equals("123")) {
            user.setUserName(userName);
            user.setPassword(password);
            Toast.makeText(MvcLoginActivity.this,
                    userName + " Login Successful",
                    Toast.LENGTH_SHORT)
                    .show();
        } else {
            Toast.makeText(MvcLoginActivity.this,
                    "Login Failed",
                    Toast.LENGTH_SHORT)
                    .show();
        }
    }
}
```

### 结论

- 具有一定的分层，model 彻底解耦，controller 和 view 并没有解耦
- 层与层之间的交互尽量使用回调或者去使用消息机制去完成，尽量避免直接持有
-  controller 和 view 在 android 中无法做到彻底分离，但在代码逻辑层面一定要分清
-  业务逻辑被放置在 model 层，能够更好的复用和修改增加业务

## MVP

### 概述

mvp是指model、view、presenter

MVP 跟 MVC 很相像，文章开头列出了很多种 MVC 的设计图，所以根据 MVC 的
发展来看，我们把 MVP 当成 MVC 来看也不为过，因为 MVP 也是三层，唯一的
差别是 Model 和 View 之间不进行通讯，都是通过 Presenter 完成。

### 模型图

![](/home/alecs/桌面/RFP){(MHEM9~5V0E5(DQZY5.png)

### Sample

model层

```
public interface IUserBiz {
    boolean login(String userName, String password);
}
```

```
/**
 * User业务逻辑实现
 */
public class UserBiz implements IUserBiz {
 
    @Override
    public boolean login(String userName, String password) {
 
        if (userName.equals("jere") && password.equals("123")) {
            User user = new User();
            user.setUserName(userName);
            user.setPassword(password);
            return true;
        }
        return false;
    }
}
```

Presenter层

```
public class LoginPresenter{
    private UserBiz userBiz;
    private IMvpLoginView iMvpLoginView;
 
    public LoginPresenter(IMvpLoginView iMvpLoginView) {
        this.iMvpLoginView = iMvpLoginView;
        this.userBiz = new UserBiz();
    }
 
    public void login() {
        String userName = iMvpLoginView.getUserName();
        String password = iMvpLoginView.getPassword();
        boolean isLoginSuccessful = userBiz.login(userName, password);
        iMvpLoginView.onLoginResult(isLoginSuccessful);
    }
 
 
}
```

View层

```
public interface IMvpLoginView {
    String getUserName();
 
    String getPassword();
 
    void onLoginResult(Boolean isLoginSuccess);
}
```

```
public class MvpLoginActivity extends AppCompatActivity implements IMvpLoginView{
    private EditText userNameEt;
    private EditText passwordEt;
    private LoginPresenter loginPresenter;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_mvp_login);
 
        userNameEt = findViewById(R.id.user_name_et);
        passwordEt = findViewById(R.id.password_et);
        Button loginBtn = findViewById(R.id.login_btn);
 
        loginPresenter = new LoginPresenter(this);
        loginBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                loginPresenter.login();
            }
        });
    }
 
    @Override
    public String getUserName() {
        return userNameEt.getText().toString();
    }
 
    @Override
    public String getPassword() {
        return passwordEt.getText().toString();
    }
 
    @Override
    public void onLoginResult(Boolean isLoginSuccess) {
        if (isLoginSuccess) {
            Toast.makeText(MvpLoginActivity.this,
                    getUserName() + " Login Successful",
                    Toast.LENGTH_SHORT)
                    .show();
        } else {
            Toast.makeText(MvpLoginActivity.this,
                    "Login Failed",
                    Toast.LENGTH_SHORT).show();
        }
    }
}
```



### 结论

通过引入接口 ，让相应的视图组件如 Activity，Fragment 去实现
接口，实现了视图层的独立，通过中间层 Preseter 实现了 Model 和 View 的
完全解耦。MVP 彻底解决了 MVC 中 View 和 Controller 傻傻分不清楚的问题，但
是随着业务逻辑的增加，一个页面可能会非常复杂，UI 的改变是非常多，会有
非常多的 case，这样就会造成 View 的接口会很庞大。

## MVVM

### 概述

mvvm是指model、view、viewmodel

MVP 中我们说过随着业务逻辑的增加，UI 的改变多的情况下，会有非常多的跟UI 相关的 case，这样就会造成 View 的接口会很庞大。而 MVVM 就解决了这个问
题，通过双向绑定的机制，实现数据和 UI 内容，只要想改其中一方，另一方都能够及时更新的一种设计理念，这样就省去了很多在 View 层中写很多 case 的情
况，只需要改变数据就行。

### 模型图

![](/home/alecs/桌面/F77OK~BB5S5MRONERU0G8H7.png)

### Sample

ViewModel层

```
public class LoginViewModel extends ViewModel {
    private User user;
    private MutableLiveData<Boolean> isLoginSuccessfulLD;
 
    public LoginViewModel() {
        this.isLoginSuccessfulLD = new MutableLiveData<>();
        user = new User();
    }
 
    public MutableLiveData<Boolean> getIsLoginSuccessfulLD() {
        return isLoginSuccessfulLD;
    }
 
    public void setIsLoginSuccessfulLD(boolean isLoginSuccessful) {
        isLoginSuccessfulLD.postValue(isLoginSuccessful);
    }
 
    public void login(String userName, String password) {
        if (userName.equals("jere") && password.equals("123")) {
            user.setUserName(userName);
            user.setPassword(password);
            setIsLoginSuccessfulLD(true);
        } else {
            setIsLoginSuccessfulLD(false);
        }
    }
 
    public String getUserName() {
        return user.getUserName();
    }
}
```

View层

```
public class MvvmLoginActivity extends AppCompatActivity {
    private LoginViewModel loginVM;
    private EditText userNameEt;
    private EditText passwordEt;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_mvvm_login);
 
        userNameEt = findViewById(R.id.user_name_et);
        passwordEt = findViewById(R.id.password_et);
        Button loginBtn = findViewById(R.id.login_btn);
        loginBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                loginVM.login(userNameEt.getText().toString(), passwordEt.getText().toString());
            }
        });
 
        loginVM = ViewModelProviders.of(this).get(LoginViewModel.class);
        loginVM.getIsLoginSuccessfulLD().observe(this, loginObserver);
    }
 
    private Observer<Boolean> loginObserver = new Observer<Boolean>() {
        @Override
        public void onChanged(@Nullable Boolean isLoginSuccessFul) {
            if (isLoginSuccessFul) {
                Toast.makeText(MvvmLoginActivity.this,
                        loginVM.getUserName() + " Login Successful",
                        Toast.LENGTH_SHORT)
                        .show();
            } else {
                Toast.makeText(MvvmLoginActivity.this,
                        "Login Failed",
                        Toast.LENGTH_SHORT)
                        .show();
            }
        }
    };
}
```



### 结论

 MVVM 很好的解决了 MVC 和 MVP 的不足，但是由于数据和视图的双向绑定，导致出现问题时不太好定位来源，有可能数据问题导致，也有可能业务逻辑中对视图属性的修改导致。如果项目中打算用 MVVM 的话可以考虑使用官方的架构组件 ViewModel、LiveData、DataBinding 去实现 MVVM
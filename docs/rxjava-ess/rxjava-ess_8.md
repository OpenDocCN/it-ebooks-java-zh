# 与 REST 无缝结合-RxJava 和 Retrofit

在上一章中，我们学习了如何使用调度器在不同于 UI 线程的线程上操作。我们学习了如何高效的运行 I/O 任务而不用阻塞 UI 以及如何运行耗时的计算任务而不耗损应用性能。在最后一章中，我们将创建一个最终版的应用实例，用 Retrofit 映射远程 API,异步查询数据，轻松创造一个丰富的 UI。

# 项目目标

我们将在已有的例子中创建一个新的`Activity`。这个`Activity`将通过 StackExchange API 从 stackoverflow 检索出最活跃的 10 位用户。App 使用这些信息来展示一个包含用户头像、姓名、名望数以及住址的列表。对每一位用户，app 使用 OpenWeatherMap API 来检索该用户住址当地的天气预报，并显示一个小天气图标。基于从 StackOverflow 检索的信息，app 对列表中的每一位用户提供一个`onClick`事件，打开他们在个人信息中设定的个人网站或者 Stack Overflow 的个人主页。

# Retrofit

Retrofit 是 Square 公司专为 Android 和 Java 设计的一个类型安全的 REST 客户端。它帮助你轻松地与任意 REST API 交互，并完美兼容 RxJava：所有的 JSON 响应对象都被映射成原始的 Java 对象，并且所有的网络调用都基于 Rxjava Observable 这些对象。

使用 API 文档，我们可以定义我们从服务器接收的 JSON 响应数据。为了很容易的将 JSON 响应数据映射为我们的 Java 代码，我们将使用[jsonschema2pojo](http://www.jsonschema2pojo.org "官网"), 这个服务将灵活地生成所有与 JSON 响应数据相映射的 Java 类。

当我们把所有的 Java model 准备好后，我们就可以开始建立 Retrofit。Retrofi 使用标准的 Java 接口来映射 API 路由。例如例子中，我们将使用来自 API 的一个路由，下面是我们 Retrofit 的接口:

```java
public interface StackExchangeService {
    @GET("/2.2/users?order=desc&sort=reputation&site=stackoverflow")
    Observable<User sResponse> getMostPopularSOusers(@Query("pagesize") int howmany);
} 
```

`interface`接口只包含一个方法，即`getMostPopularSOusers`。这个方法用整型`howmany`作为一个参数并返回`UserResponse`的 Observable。

当我们有了`interface`，我们可以创建`RestAdapter`类，为了更清楚的组织我们的代码，我们创建一个`SeApiManager`函数提供一种更适当的方式来和 StackExchange API 交互。

```java
public class SeApiManager {
    private final StackExchangeService mStackExchangeService;

    public SeApiManager() {
        RestAdapter restAdapter = new RestAdapter.Builder()
            .setEndpoint("https://api.stackexchange.com")
            .setLogLevel(RestAdapter.LogLevel.BASIC)
            .build();
        mStackExchangeService = restAdapter.create(StackExchangeService.class);
    }

    public Observable<List<User>> getMostPopularSOusers(int howmany) {
        return mStackExchangeService
        .getMostPopularSOusers(howmany)
        .map(UsersResponse::getUsers)
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread());
    }
} 
```

为了简化例子，我们不再将这个类设计为它本该设计成的单例。使用依赖注入解决方案，如 Dagger2，可使代码质量更高。

创建`RestAdapter`类，需要对客户端 API 设置几个重要的方面。这个例子中，我们设置了`endpoint`和`log level`。由于这个例子中 URL 只是硬编码，像这样使用外部资源来存储数据很重要。避免在代码中硬编码字符串是一个好的实践。

Retrofit 把`RestAdapter`类和我们的 API 接口绑定在一起后就完成了创建。它返回给我们一个对象用来请求 API。我们可以选择直接暴露这个对象，或者以某种封装方式来限制对它的访问。在这个例子中，我们封装它并只暴露`getMostPopularSOusers`方法。这个方法执行查询，使用 Retrofit 解析 JSON 响应数据。获得用户列表，并返回给订阅者。如你所见，使用 Retrofit、RxJava 和 Retrolambda，我们几乎没有模板代码：它非常紧凑而且可读性很高。

现在，我们已经有一个 API 管理者来提供一个响应式的方法，它从远程 API 获取数据并给 I/O 调度器，解析映射最后为我们的消费者提供一个简洁的用户列表。

```java

# App 架构

我们不使用任何 MVC，MVP，或者 MVVM 模式。因为那不是这本书的目的，因此我们的`Activity`类将包含我们需要创建和展示用户列表的所有逻辑。

# 创建 Activity 类

我们将在`onCreate()`方法里创建`SwipeRefreshLayout`和`RecyclerView`；我们有一个`refreshList()`方法来处理用户列表的获取和展示，`showRefreshing()`方法来管理进度条和`RecyclerView`的显示。

我们的`refreshList()`函数看起来如下：

```
private void refreshList() { 
    showRefresh(true);
    mSeApiManager.getMostPopularSOusers(10)
        .subscribe(users -> { 
            showRefresh(false);
            mAdapter.updateUsers(users);
        }, error -> { 
            App.L.error(error.toString());
            showRefresh(false);
        });
} 
```java

我们显示了进度条，从 StackExchange API 管理器观测用户列表。一旦获取到列表数据，我们开始展示它并更新`Adapter`的内容并让`RecyclerView`显示为可见。

# 创建 RecyclerView Adapter

我们从 REST API 获取到数据后，我们需要把它绑定 View 上，并用一个适配器填充列表。我们的 RecyclerView 适配器是标准的。它继承于`RecyclerView.Adapter`并指定它自己的`ViewHolder`：

```
public static class ViewHolder extends RecyclerView.ViewHolder {
    @InjectView(R.id.name) TextView name;
    @InjectView(R.id.city) TextView city;
    @InjectView(R.id.reputation) TextView reputation;
    @InjectView(R.id.user_image) ImageView user_image;
    public ViewHolder(View view) { 
        super(view);
        ButterKnife.inject(this, view); 
    }
} 
```java

我们一旦收到来自 API 管理器的数据，我们可以设置界面上所有的标签：`name`,`city`和`reputation`。

为了展示用户的头像，我们将使用 Sergey Tarasevich 写的[Universal Image Loader](https://github.com/nostra13/Android-Universal-ImageLoader)。实践证明，UIL 是非常有名的好用的图片管理库。我们也可以使用 Square 公司的 Picasso，Glide 或者 Facebook 公司的 Fresco。取决于你自己的喜好。最关键的是无需重复造轮子：库能够方便开发者并让他们更快速实现目标。

在我们的适配器中，我们可以这样：

```
@Override
public void onBindViewHolder(SoAdapter.ViewHolder holder, int position) {
    User user = mUsers.get(position);
    holder.setUser(user); 
} 
```java

在`ViewHolder`，我们可以这样：

```
public void setUser(User user) { 
    name.setText(user.getDisplayName());
    city.setText(user.getLocation());
    reputation.setText(String.valueOf(user.getReputation()));

    ImageLoader.getInstance().displayImage(user.getProfileImage(), user_image);
} 
```java

此时，我们可以允许代码获得一个用户列表，正如下图所示：

![](img/chapter8_1.png)

### 检索天气预报

我们加大难度，将当地城市的天气加入列表中。**OpenWeatherMap**是一个灵活公共在线天气 API，我们可以查询许多有用的预报信息。

和往常一样，我们将使用 Retrofit 映射到 API 然后通过 RxJava 来访问它。至于 StackExchange API，我们将创建`interface`，`RestAdapter`和一个灵活的管理器：

```
public interface OpenWeatherMapService {
    @GET("data2.5/weather")
    Observable<WeatherResponse> getForecastByCity(@Query("q") String city);
} 
```java

这个方法用城市名字作为参数提供当地的预报信息。我们像下面这样将接口和`RestAdapter`类绑定在一起：

```
RestAdapter restAdapter = new RestAdapter.Builder()
        .setEndpoint("http://api.openweathermap.org")
        .setLogLevel(RestAdapter.LogLevel.BASIC)
        .build();
mOpenWeatherMapService = restAdapter.create(OpenWeatherMapService.class); 
```java

像以前一样，我们只有两件事需要立马去做：设置 API 端口和 log 级别。

`OpenWeatherMapApiManager`类将提供下面的方法：

```
public Observable<WeatherResponse> getForecastByCity(String city) {
    return mOpenWeatherMapService.getForecastByCity(city)
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread());
} 
```java

现在，我们有了用户列表，我们可以根据城市名来查询 OpenWeatherMap 获得天气预报信息。下一步是修改我们的`ViewHolder`类来为每位用户展示相应的天气图标。

我们使用这些工具方法先验证用户主页信息并获得一个合法的城市名字：

```
private boolean isCityValid(String location) {
    int separatorPosition = getSeparatorPosition(location);
    return !"".equals(location) && separatorPosition > -1; 
}

private int getSeparatorPosition(String location) { 
    int separatorPosition = -1;
    if (location != null) {
        separatorPosition = location.indexOf(","); 
    }
    return separatorPosition; 
}

private String getCity(String location, int position) {
    if (location != null) {
        return location.substring(0, position); 
    } else {
        return ""; 
    }
} 
```java

借助一个有效的城市名，我们可以用下面命令来获得我们所需要天气的所有数据：

```
OpenWeatherMapApiManager.getInstance().getForecastByCity(city) 
```java

用天气响应的结果，我们可以获得天气图标的 URL：

```
getWeatherIconUrl(weatherResponse); 
```java

用图标 URL，我们可以检索到图标本身：

```
private Observable<Bitmap> loadBitmap(String url) {
    return Observable.create(subscriber -> {
        ImageLoader.getInstance().displayImage(url,city_image, new ImageLoadingListener() { 
            @Override
            public void onLoadingStarted(String imageUri, View view) {
            }

            @Override
            public void onLoadingFailed(String imageUri, View view, FailReason failReason) {
                subscriber.onError(failReason.getCause()); 
            }

            @Override
            public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
                subscriber.onNext(loadedImage);
                subscriber.onCompleted(); 
            }

            @Override
            public void onLoadingCancelled(String imageUri, View view) {
                subscriber.onError(new Throwable("Image loading cancelled"));
            }
         });
    });
} 
```java

这个`loadBitmap()`返回的 Observable 可以链接前面一个，并且最后我们可以为这个任务返回一个单独的 Observable：

```
if (isCityValid(location)) {
    String city = getCity(location, separatorPosition);
    OpenWeatherMapApiManager.getInstance().getForecastByCity(city)
        .filter(response -> response != null)
        .filter(response -> response.getWeather().size() > 0)
        .flatMap(response -> {
            String url = getWeatherIconUrl(response);
            return loadBitmap(url);
        })
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<Bitmap>() {

        @Override
        public void onCompleted() {
        }

        @Override
        public void onError(Throwable e) {
            App.L.error(e.toString()); 
        }

        @Override
        public void onNext(Bitmap icon) {
            city_image.setImageBitmap(icon); 
        }
    });
} 
```java

运行代码，我们可以在下面列表中为每个用户获得新的天气图标： ![](img/chapter8_2.png)

### 打开网站

使用用户主页包含的信息，我们将会创建一个`onClick`监听器来导航到用户 web 页面，如果有的话，否则打开在 Stack Overflow 上的个人主页。

为了实现它，我们简单实现`Activity`类的接口，用来在适配器触发 Android 的`onClick`事件。

我们的`Adapter ViewHolder`指定这个接口：

```
public interface OpenProfileListener {
    public void open(String url); 
} 
```java

`Activity`实现它：

```
[...] implements SoAdapter.ViewHolder.OpenProfileListener { [...]
    mAdapter.setOpenProfileListener(this); 
[...]

@Override
public void open(String url) {
    Intent i = new Intent(Intent.ACTION_VIEW);
    i.setData(Uri.parse(url)); 
    startActivity(i);
} 
```java

`Activity`收到 URL 并用外部 Android 浏览器打开它。我们的`ViewHolder`负责在用户列表的每个卡片上创建`OnClickListener`并检查我们是打开 Stack Overflow 用户主页还是外部个人站：

```
mView.setOnClickListener(view -> { 
    if (mProfileListener != null) {
        String url = user.getWebsiteUrl();
        if (url != null && !url.equals("") && !url.contains("search")) {
            mProfileListener.open(url); 
        } else {
            mProfileListener.open(user.getLink()); 
        }
    }
)}； 
```java

一旦我们点击了，我们将直接重定向到预期的网站。在 Android 上，我们可以用 RxAndroid 的一种特殊形式（ViewObservable）以更加响应式的方式实现同样的结果。

```
ViewObservable.clicks(mView)
    .subscribe(onClickEvent -> {
    if (mProfileListener != null) {
        String url = user.getWebsiteUrl();
        if (url != null && !url.equals("") && !url.contains("search")) { 
            mProfileListener.open(url);
        } else {
            mProfileListener.open(user.getLink());
        } 
    }
}); 
```java

上面两块代码片段是等价的，你可以选择最喜欢的方式来实现。

# 总结

我们的旅程结束了。相信你已经准备好将你的 Java 应用带到一个新的代码质量水平。你可以享受一个新的编程模式并把更流畅的思维方式应用到日常编程生活中。RxJava 提供了一种以面向时序的方式考虑数据的机会：所有事情都是持续变化的，数据在更新，事件在触发，然后你就可以创建事件响应式的、灵活的、运行流畅的 App。

刚开始切换到 RxJava 看起来困难并且耗时，但我们已经体验到了如何通过响应式的方式有效地处理日常问题。现在你可以把你的旧代码迁移到 RxJava 上:给这些同步`getters`一种新的响应式。

RxJava 是一个正在不断发展和扩大的世界。还有许多方法我们还没有去探索。有些方法甚至还没有，通过 RxJava，你可以创建你自己的操作符并把他们发展地更远。

Android 是一个好玩的地方，但是它也有局限性。作为一个 Android 开发者，你可以用 RxJava 和 RxAndroid 克服其中的许多。我们用 AndroidScheduler 只简单提了下 RxAndroid,除了在最后一章，你了解了`ViewObservable`。RxAndroid 给了你许多：例如，`WidgetObservable`，`LifecycleObservable`。往后将它发展地更长远的任务就取决于你了。

谨记可观测序列就像一条河：它们是流动的。你可以“过滤”(filter)一条河，你可以“转换”(transform)一条河，你可以将两条河合并(combine)成一个，然后依然畅流如初。最后，它就成了你想要的那条河。

```
“Be Water，my friend”

            --Bruce Lee 
```
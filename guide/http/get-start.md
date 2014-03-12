# LiteHttp
我们来看一个普通android网络应用上，最基础的http连接和json解析方式，一个简单例子，从服务器获取id=168的用户的信息，User的json结构：
```json
{
	"api": "com.xx.get.userinfo",
	"v": "1.0",
	"result": {
		"code": 200,
		"message": "success"
	},
	"data": {
		"age": 18,
		"name": "qingtianzhu",
		"girl_friends": [
			"xiaoli",
			"fengjie",
			"lucy"
		]
	}
}
```
api, v, result为基础信息，完成这个任务一般分三步：
1. 组织api的url地址和参数（这个很简单），比如：
```java
String url = "http://litesuits.github.io/mockdata/user?userid=168";
```
2. http连接网络获取api信息（这个通过封装后还算简单），典型代码
```java
/**
 * 关闭流顺序：先打开的后关闭；被依赖的后关闭。
 * @return string info
 */
public static String sendHttpRequst(String apiUrl) {
	HttpURLConnection conn = null;
	InputStream is = null;
	ByteArrayOutputStream baos = null;
	try {
		URL url = new URL(apiUrl);
		conn = (HttpURLConnection) url.openConnection();
		conn.connect();
		is = conn.getInputStream();
		int len = conn.getContentLength();
		if (len < 1) len = 1024;
		baos = new ByteArrayOutputStream(len);
		byte[] buffer = new byte[1024];
		len = 0;
		while ((len = is.read(buffer)) != -1) {
			baos.write(buffer, 0, len);
		}
		return baos.toString();
	} catch (MalformedURLException e) {
		e.printStackTrace();
	} catch (IOException e) {
		e.printStackTrace();
	} finally {
		try {
			if (baos != null) baos.close();
			if (is != null) is.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
		if (conn != null) conn.disconnect();
	}
	return null;
}
```
3.解析服务器反馈的string（这是很头疼的）：
```java
public static User parseJsonToUser(String json) {
	User user = new User();
	if (json != null) {
		try {
			JSONObject jsonObj = new JSONObject(json);
			JSONObject result = jsonObj.optJSONObject("result");
			JSONObject data = jsonObj.optJSONObject("data");
			JSONArray arr = null;
			if (data != null) arr = data.optJSONArray("girl_friends");

			user.result = new User.Result();
			user.data = new User.UserInfo();
			user.data.girl_friends = new ArrayList<String>();

			if (jsonObj != null) {
				user.api = jsonObj.optString("api");
				user.v = jsonObj.optString("v");
			}
			if (result != null) {
				user.result.code = result.optInt("code");
				user.result.message = result.optString("message");
			}
			if (data != null) {
				user.data.age = data.optInt("age");
				user.data.name = data.optString("name");
			}
			if (arr != null) {
				for (int i = 0, size = arr.length(); i < size; i++) {
					user.data.girl_friends.add(arr.optString(i));
				}
			}
		} catch (JSONException e) {
			e.printStackTrace();
		}
	}
	return user;
}
```
那么我们看到在User这个类仅仅有三个属性的情况下，仅第3步就写了约37行代码，如果User有几十个属性，代码量将会暴增，这种方式相对会耗费较大的劳动力。
那么，就此转入正题，看看LiteHttp是怎么完成这个任务的：
```java
User user = client.get(url, null, User.class);
```
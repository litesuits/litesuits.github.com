litesuits.github.com
====================

hello world

# LiteHttp

```java
@SuppressWarnings("unchecked")
	private Result getResultFromCache() {
		ObjectInputStream ois = null;
		try {
			ois = new ObjectInputStream(new FileInputStream(new File(cachePath, key)));
			Object obj = ois.readObject();

			if (obj != null) {
				if (Log.isPrint) Log.d(TAG, "getResultFromCache: sucess");
				return (Result) obj;
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (ois != null) try {
				ois.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		if (Log.isPrint) Log.d(TAG, "getResultFromCache: fail ");
		return null;
	}
```

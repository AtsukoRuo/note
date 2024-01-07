# ContentProvider

文件存储和`SharedPreferences`存储中提供了`MODE_WORLD_READABLE`和`MODE_WORLD_WRITEABLE`这两种操作模式，用于给其他应用程序访问当前应用的数据。但出于安全性的考量，这两种模式在Android 4.2版本中都已被废弃了。官方更推荐使用安全可靠的`ContentProvider`技术。



### 权限

对于普通权限，Android会自动帮我们授权。而对于危险权限，用户必须手动授权。

对于开发者来说，使用危险权限时除了要在AndroidManifest.xml中声明，还必须进行运行时权限处理，下面以拨打电话的例子来说明一下：

~~~xml
<uses-feature
    android:name="android.hardware.telephony"
    android:required="true" />

<uses-permission 
    android:name="android.permission.CALL_PHONE"/>
~~~

~~~kotlin
// UI层
Button(onClick = {
    if (ContextCompat.checkSelfPermission(context, Manifest.permission.CALL_PHONE) != PackageManager.PERMISSION_GRANTED) {
        ActivityCompat.requestPermissions(context, arrayOf(Manifest.permission.CALL_PHONE), 1)
    }
}) {

}
~~~

~~~kotlin
//这个方法已经@Deprecated
override fun onRequestPermissionsResult(
    requestCode: Int,
    permissions: Array<out String>,
    grantResults: IntArray
) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)
    when (requestCode) {
        1 -> {
            if (grantResults.isNotEmpty() &&
                grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                call()
            } else {
                Toast.makeText(this, "You denied the permission",
                    Toast.LENGTH_SHORT).show()
            }
        }
    }
}

private fun call() {
    try {
        val intent = Intent(Intent.ACTION_CALL)
        intent.data = Uri.parse("tel:10086")
        startActivity(intent)
    } catch (e: SecurityException) {
        e.printStackTrace()
    }
}
~~~

推荐使用 `registerForActivityResult()` 来代替已经废除的`onRequestPermissionsResult`



~~~kotlin
val requestPermissionLauncher = registerForActivityResult(
    ActivityResultContracts.RequestPermission()
) { isGranted ->
    if (isGranted) {
        try {
            val intent = Intent(Intent.ACTION_CALL)
            intent.data = Uri.parse("tel:10086")
            startActivity(intent)
        } catch (e: SecurityException) {
            e.printStackTrace()
        }
    } else {
        Toast.makeText(this, "You denied the permission",
            Toast.LENGTH_SHORT).show()
    }
}
~~~

~~~kotlin
Button(onClick = {
   requestPermissionLauncher.launch(Manifest.permission.CALL_PHONE)
}) {

}
~~~



申请多个权限的例子

~~~kotlin
val requestMultiplePermissions = registerForActivityResult(ActivityResultContracts.RequestMultiplePermissions()) { 
    permissions: Map<String, Boolean> ->
    permissions.entries.forEach {
        val permissionName = it.key
        if (it.value) {
            
        } else {
            //未同意授权
        }
    }
}

~~~



## ContentResolver

ContentResolver类用于访问ContentProvider中共享的数据。可以通过Context中的`getContentResolver()`方法获取该类的实例。它通过「内容URI」来定位数据。内容URI由两部分组成：

- `authority`：用于区分不同程序的，一般是包名
- `path`：资源路径

~~~kotlin
val uri = Uri.parse("content://com.example.app.provider/table1")
~~~





ContentResolver中提供了一系列的方法用于对数据进行增删改查操作，

- `insert()`方法用于添加数据

  ~~~kotlin
  val values = contentValuesOf("column1" to "text", "column2" to 1)
  contentResolver.insert(uri, values)
  ~~~

- `update()`方法用于更新数据

  ~~~kotlin
  val values = contentValuesOf("column1" to "")
  contentResolver.update(uri, values, "column1 = ? and column2 = ?", arrayOf("text", "1"))
  ~~~

- `delete()`方法用于删除数据

  ~~~kotlin
  contentResolver.delete(uri, "column2 = ?", arrayOf("1"))
  ~~~

  

- `query()`方法用于查询数据：

  | `query()`方法参数 | 对应SQL部分                 | 描述                             |
  | :---------------- | :-------------------------- | :------------------------------- |
  | `uri`             | `from table_name`           | 指定查询某个应用程序下的某一张表 |
  | `projection`      | `select column1, column2`   | 指定查询的列名                   |
  | `selection`       | `where column = value`      | 指定`where`的约束条件            |
  | `selectionArgs`   | -                           | 为`where`中的占位符提供具体的值  |
  | `sortOrder`       | `order by column1, column2` | 指定查询结果的排序方式           |

  ~~~kotlin
  val cursor = contentResolver.query(
      uri,
      projection,
      selection,
      selectionArgs,
      sortOrder
  )
  
  while (cursor.moveToNext()) {
      val column1 = cursor.getString(cursor.getColumnIndex("column1"))
      val column2 = cursor.getInt(cursor.getColumnIndex("column2"))
  }
  cursor.close()
  ~~~

  

## 创建ContentProvider

首先继承`ContentProvider`类，并覆写如下6个方法。然后通过解析URI以及相应参数，并配合着Database对象/内存数据/File/SharedPreference等等，来完成处理共享数据的逻辑

~~~kotlin
class MyProvider : ContentProvider() {

    override fun onCreate(): Boolean {
        // 返回true表示ContentProvider初始化成功，返回false则表示失败。
        return false
    }

    override fun query(uri: Uri, projection: Array<String>?, selection: String?,
            selectionArgs: Array<String>?, sortOrder: String?): Cursor? {
        return null
    }

    override fun insert(uri: Uri, values: ContentValues?): Uri? {
        return null
    }

    override fun update(uri: Uri, values: ContentValues?, selection: String?,
            selectionArgs: Array<String>?): Int {
        return 0
    }

    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<String>?): Int {
        return 0
    }

    override fun getType(uri: Uri): String? {
        return null
    }

}
~~~

这些`uri`参数用于指定查询哪张表。此外，URI格式还有几个变种：

- 匹配table1表中的id为1的数据。

  ~~~kotlin
  content://com.example.app.provider/table1/1
  ~~~

- 匹配table1表中任意一行数据的内容

  ~~~kotlin
  content://com.example.app.provider/table1/#
  ~~~

- 匹配任意表的内容

  ~~~kotlin
  content://com.example.app.provider/*
  ~~~

我们借助`UriMatcher`这个类可以支持上述几个变种：

~~~kotlin
class MyProvider : ContentProvider() {
    private val table1Dir = 0
    private val table1Item = 1
    private val table2Dir = 2
    private val table2Item = 3

    private val uriMatcher = UriMatcher(UriMatcher.NO_MATCH)

    init {
        uriMatcher.addURI("com.example.app.provider", "table1", table1Dir)
        uriMatcher.addURI("com.example.app.provider ", "table1/#", table1Item)
        uriMatcher.addURI("com.example.app.provider ", "table2", table2Dir)
        uriMatcher.addURI("com.example.app.provider ", "table2/#", table2Item)
    }
    //...
    override fun query(uri: Uri, projection: Array<String>?, selection: String?,
            selectionArgs: Array<String>?, sortOrder: String?): Cursor? = dbHelper?.let { 
        when (uriMatcher.match(uri)) {
            table1Dir -> {
                // 查询table1表中的所有数据
            }
            table1Item -> {
                // 查询table1表中的单条数据
            }
            table2Dir -> {
                // 查询table2表中的所有数据
            }
            table2Item -> {
                // 查询table2表中的单条数据
            }
        }
        null;
    }
}
~~~

为什么不直接做字符串匹配呢？通过`uriMatcher`来提高可重用性。

`getType()`方法用于获取`Uri`对象所对应的`MIME`类型，它由三部分组成

- 必须以`vnd`开头。
- 如果内容URI以路径结尾，则后接`android.cursor.dir/`；如果内容URI以id结尾，则后接`android.cursor.item/`。
- 最后接上`vnd.<authority>.<path>`。

对于content://com.example.app.provider/table1这个内容URI，它所对应的`MIME`类型就可以写成：

~~~xml
vnd.android.cursor.dir/vnd.com.example.app.provider.table1
~~~

对于content://com.example.app.provider/table1/1这个内容URI，它所对应的`MIME`类型就可以写成：

~~~xml
vnd.android.cursor.item/vnd.com.example.app.provider.table1
~~~



最后在`AndroidManifest.xml`注册即可：

~~~xml
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:label="@string/app_name"
    android:supportsRtl="true"
    android:theme="@style/AppTheme">
    ...
    <provider
        android:name=".DatabaseProvider"
        android:authorities="com.example.databasetest.provider"
        android:enabled="true"
        android:exported="true">
    </provider>
</application>
~~~


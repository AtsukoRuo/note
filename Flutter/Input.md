# Input

Form

TextFormField

DropdownButtonFormField

~~~dart
class NewItem extends StatefulWidget {
	const NewItem({super.key});

	@override
	State<NewItem> createState() {
		return _NewItemState();
	}
}

class _NewItemState extends State<NewItem> {
	final _formKey = GlobalKey<FormState>();							//通过全局键，可以引用对应的state、element	
	var _enterName = "";
	var _enterQuantity = 0;
	var _selectedCategory = categories[Categories.vegetables]!;

	void _saveItem() {			//点击后触发这段代码
        //全局Key绑定了对应的Form
		if (_formKey.currentState!.validate()) {			//调用Form widget中所有的validate函数
			_formKey.currentState!.save();				   //调用Form widget所有的onSave函数
			Navigator.of(context).pop(
				GroceryItem(
					id: DateTime.now().toString(), 
					name: _enterName,
					quantity: _enterQuantity,
					category: _selectedCategory
				)
			);
		}
	}

	@override
	Widget build(BuildContext context) {
		return Scaffold(
			appBar: AppBar(
				title: const Text('Add a new item'),
			),
			body: Padding(
				padding: const EdgeInsets.all(12),
				child: Form(
					key : _formKey,
					child : Column(
						children: [
							//注意不是TextField()
							TextFormField(
								maxLength: 50,			//最大长度
								decoration: const InputDecoration(		//装饰
									label: Text("name"),
								),
								validator: (value) {			//验证
									if (value == null 
									|| value.isEmpty 
									|| value.trim().length <= 1
									|| value.trim().length > 50) {
										return "Must be between 1 an d 50 characters";
									}
									return null;
								},
								onSaved: (value) {				//保存
									_enterName = value!;
								},
							),
							Row(
								crossAxisAlignment: CrossAxisAlignment.end,
								children: [
									Expanded(
										child : TextFormField(
											decoration: const InputDecoration(
												label: Text("Quantity")
											),
											initialValue: '1',
											validator: (value) {
												if (value == null 
												|| value.isEmpty 
												|| int.tryParse(value) == null
												|| int.tryParse(value)! <= 0) {
													return "value must be number that is positive";
												}
												return null;
											},
											onSaved: (value) {
												_enterQuantity = int.parse(value!);
											},
										),
										
									),
									const SizedBox(width : 8),
									Expanded(
										child :DropdownButtonFormField(
										value :  _selectedCategory,
										items: [
											for (final category in categories.entries)
												DropdownMenuItem(
													value : category.value,			//显示值
													child: Row(						//表项
														children: [
															Container(
															width: 16,
															height: 16,
															color : category.value.color
															),
															const SizedBox(width: 6),
															Text(category.value.title)
														],
													),
												)
										], 
										onChanged: (value) {
											setState(() {
											  _selectedCategory = value!;
											});
										})
									)
								],
							),
							const SizedBox(
								height: 18,
							),
							Row(
								mainAxisAlignment: MainAxisAlignment.end,
								crossAxisAlignment: CrossAxisAlignment.start,
								children: [
									TextButton(
										onPressed: () {
											_formKey.currentState!.reset();
										},
										child : const Text("Reset"),
									),
									ElevatedButton(
										onPressed: _saveItem,
										child : const Text("Add Item"),
									)
								],
							)
						],
					)
				),
			),
		);
	}
}
~~~





## Key

LocalKey 直接继承至 Key，它应用于拥有相同父 Element 的小部件进行比较的情况

Localkey 派生出了许多子类 key：

- ValueKey : ValueKey('String')
- ObjectKey : ObjectKey(Object)
- UniqueKey : UniqueKey()

GlobalKey 使用了一个静态常量 Map 来保存它对应的 Element。

你可以通过 GlobalKey 找到持有该GlobalKey的 **Widget**，**State** 和 **Element**。



## HTTP

https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Overview

https://firebase.google.com/?gad=1&gclid=CjwKCAjwzo2mBhAUEiwAf7wjkvpHdVRnN8EBdtqmwczgd62LbOK41PTuexxkI6oDJH6TfUhyTaV3XhoCS48QAvD_BwE&gclsrc=aw.ds&hl=zh-cn

https://firebase.google.com/docs/reference/rest/database?hl=zh-cn

https://pub.dev/packages/http/install



## POST

~~~dart
final url = Uri.https("flutter-prep-637d1-default-rtdb.firebaseio.com", "shopping-list.json");
final response = await http.post(
    url, 
    headers: {
        'Content-Type' : 'application/json',
    },
    body : json.encode({
        'name': _enterName,
        'quantity': _enterQuantity,
        'category': _selectedCategory.title 
    })
);

print(response.body);
print(response.statusCode);
~~~



## GET

~~~dart

final url = Uri.https("flutter-prep-637d1-default-rtdb.firebaseio.com", "shopping-list.json");
//get可能抛出异常
final response = await http.get(url);
//response可能没有内容
//JSON格式转换为Map或者List
final Map<String, dynamic>ListData = json.decode(response.body);
final List<GroceryItem> _loadItems = [];
//将Map转换为实际的对象
for (final item in ListData.entries) {
    _loadItems.add(GroceryItem(
        id: item.key, 
        name: item.value['name'], 
        quantity: item.value['quantity'], 
        category: categories.entries.firstWhere(
            (element) => element.value.title == item.value['category']
        ).value
    ));
}
setState(() {
  _groceryItems = _loadItems;
});
~~~



## Other

禁用按钮

~~~dart
TextButton(
    onPressed: _isSending? null : () {
        _formKey.currentState!.reset();
    },
    child : const Text("Reset"),
),
~~~



加载条

~~~dart
CircularProgressIndicator();
~~~



通过回调函数，可以向用户暴露一些状态，供用户配置修改这些状态



## 异步UI更新

FutureBuilder、StreamBuilder适用于依赖一些异步数据来动态更新UI的场景。它简化了在StatefulWidget中做相同的工作（状态读取 + IF判断切换Widget）

### FutureBuilder

~~~dart
FutureBuilder({
  this.future,
  this.initialData,
  required this.builder,
})
~~~

- `future`：`FutureBuilder`依赖的`Future`，通常是一个异步耗时任务

- `initialData`：初始数据，用户设置默认数据。

- `builder`：Widget构建器

  ~~~dart
  Function (BuildContext context, AsyncSnapshot snapshot) 
  ~~~

  - `snapshot`会包含当前异步任务的状态信息及结果信息



在Future就绪时，会调用builder。以及在setState时，会重新执行整个FutureBuilder

一个例子：

~~~dart
Future<List<GroceryItem>> _loadItems() async {
    final url = Uri.https("flutter-prep-637d1-default-rtdb.firebaseio.com", "shopping-list.json");
    response = await http.get(url);
    if (response.statusCode >= 400) {
        throw Exception("Failed to fetch grocy.");
    }
    if (response.body == 'null') {
        return [];
    }
    final List<GroceryItem> _loadItems = [];
    //Transform Data
    return _loadItems;
}


FutureBuilder(
    future: _loadItems(),					//这里最好使用一个变量来代替函数调用
    builder: (context, snapshot) {
        //还在异步计算
        if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
        }
        //抛出异常
        if (snapshot.hasError) {
            return Center(child : Text(snapshot.error.toString()));
        }

        if (snapshot.data!.isEmpty) {
            return const Center(
                child : Text("No items added yet.")
            );
        }
        return ListView.builder();
    },
)
~~~

## Image_picker

~~~dart
class ImageInput extends StatefulWidget {
  const ImageInput({super.key});

  @override
  State<ImageInput> createState() => _ImageInputState();
}

class _ImageInputState extends State<ImageInput> {
  File? _selectedImage;
  
  void _takePicture() async {
    final imagePicker = ImagePicker();
    final pickedImage = await imagePicker.pickImage(source: ImageSource.camera, maxWidth: 600,);
    if (pickedImage == null) {
      return ;
    }
    setState(() {
      _selectedImage = File(pickedImage.path);
    });
    
  }
  @override
  Widget build(BuildContext context) {
    Widget content = TextButton.icon(
      icon : const Icon(Icons.camera),
      label : const Text("Take Picture"),
      onPressed: _takePicture,
    );

    if (_selectedImage != null) {
      content = GestureDetector(
        onTap : _takePicture,
        child : Image.file(
          _selectedImage!, 
          fit: BoxFit.cover, 
          width : double.infinity,
          height: double.infinity,
        )
      ); 
    }
    return Container(
      decoration: BoxDecoration(
        border : Border.all(
          width : 1,
          color : Theme.of(context).colorScheme.primary.withOpacity(0.2)
        )
      ),
      height : 250,
      width: double.infinity,             //让子元素尽可能地大
      alignment: Alignment.center,
      child : content
    );
  }
}
~~~



GestureDetector：可以为某个Widget添加监听事件

~~~dart

CircleAvatar(
  radius: 26,
  backgroundImage: FileImage(
    places[index].image
  ),
  child：
),
~~~



flutter pub add location

https://pub.dev/packages/location

注意要在android/app/build.gradle中修改ext.kotlin_version为最新版本

https://console.cloud.google.com/google/maps-apis/discover?hl=zh-cn&project=awesome-griffin-394304

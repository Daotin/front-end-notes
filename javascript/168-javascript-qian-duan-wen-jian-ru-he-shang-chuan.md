# 168 javascript-前端文件如何上传

前端无法直接操作本地文件，所以需要用户触发。常见的有三种触发方式：

* 通过`<input type="file" id="file-input"/>` 选择文件
* 通过拖拽的方式把文件拖过来
* 在编辑框里面复制粘贴

## 设置文件上传的样式

因为`<input type="file" id="file-input"/>` 文件不好修改样式，一般我们会自己做一个上传的按钮来代替原生上传按钮。

然后，可以在自定义按钮上绑定点击事件，在这个点击事件里面对原生上传按钮进行操作，可以像下面这样：

```javascript
let file = document.querySelector('#fileInput');
file.click();
```

也可以将原生按钮覆盖在自定义按钮上面，然后将原生按钮和自定义按钮设置相同的大小，然后将原生按钮定位在自定义按钮之上，最后设置原生按钮的`opacity`为0即可。

## 点击按钮获取文件

### 第一种普通上传方式

```javascript
$("#file-input").on("change", function() {
    console.log(`文件名称： ${e.target.value}`); // C:\fakepath\1111.jpg

    // 创建一个formData对象，后期通过ajax上传到服务器
    let formData = new FormData();
    formData.append("iFile", this.files[0]);

    // ajax上传到服务器代码略...
});

// 后面再次获取到这个formData文件，就可以得到formData对象的myFileName文件（C:\fakepath\1111.jpg）
let file = formData.get('iFile'); 

// file类型数据件内容：
// {
//  lastModified: 1594620655781
//  lastModifiedDate: Mon Jul 13 2020 14:10:55 GMT+0800 (中国标准时间) {}
//  name: "1111.jpg"
//  size: 29848
//  type: "image/jpeg"
//  webkitRelativePath: ""
// }
```

这个file文件，如果是图片的话，需要在前端显示。但是file文件是二进制文件，没法直接查看，需要进一步转换。这个可以通过`FileReader`对象就可以做到。

通过实例化一个`FileReader`，调它的`readAsDataURL`并把File对象传给它，监听它的`onload`事件，load完读取的结果就在它的`result`属性里了。）它是一个base64格式的，可直接赋值给一个img的src）。

```javascript
// 后期取到file文件
let reader = new FileReader();
let fileType = file.type;

// reader读取完成
reader.onload = function (e) {
    if(/^image\/[jpeg|png|gif]/.test(fileType)) {
        let img = document.createElement("img");
        img.src = e.target.result;
        document.body.appendChild(img); // 将图片插入body中
    }
}

// 调用reader进行读取
reader.readAsDataURL(file);
```

### 第二种拖拽的方式

我们需要监听拖拽事件：

```javascript
$(".img-container").on("dragover", function (event) {
    event.preventDefault();
}).on("drop", function(event) {
    event.preventDefault();
    // 数据在event的dataTransfer对象里
    let file = event.originalEvent.dataTransfer.files[0];

    // 然后就可以使用FileReader进行操作
    fileReader.readAsDataURL(file);

    // 或者是添加到一个FormData
    let formData = new FormData();
    formData.append("fileContent", file);
})
```

### 第三种粘贴的方式

```javascript
// 粘贴的数据是在event.clipboardData.files里面：
$("#editor").on("paste", function(event) {
    let file = event.originalEvent.clipboardData.files[0];
});
```

> 注意： 上面，我们使用了三种方式获取文件内容，最后得到：
>
> * FormData格式
> * FileReader读取得到的base64二进制格式
>
> 如果不使用jQuery，没有问题，直接使用ajax发送就好；如果使用jQuery，要设置两个属性为false，因为jQuery会自动把内容做一些转义，并且根据data自动设置请求mime类型，这里告诉jQuery直接用xhr.send发出去就行了。：
>
> ```javascript
> $.ajax({
>     url: "/upload",
>     type: "POST",
>     data: formData,
>     processData: false,  // 不处理数据
>     contentType: false   // 不设置内容类型
> });
> ```

## 参考链接

* [https://juejin.im/post/5a193b4bf265da43052e528a](https://juejin.im/post/5a193b4bf265da43052e528a)
* [https://github.com/Daotin/notes/issues/70](https://github.com/Daotin/notes/issues/70)


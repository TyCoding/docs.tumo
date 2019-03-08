Vue+ElementUI+SpringMVC实现图片上传和table回显

在之前我们已经讲过了 [Vue+ElementUI+SpringMVC实现分页](http://tycoding.cn/2018/07/30/vue-6/#more) 。

而我们也常遇到表单中包含图片上传的需求，并且需要在table中显示图片，所以这里我就讲一下结合后端的SpringMVC框架如何实现图片上传并提交到表单中，在table表格中回显照片。


本案例对应的**开源项目地址**请看我的GitHub仓库：

* [优雅的入门SpringBoot+Mybatis，实现简单的CRUD ](https://github.com/TyCoding/spring-boot)

* [优雅的实现电商项目中搜索功能，整合SSM+Redis+Shiro+Solr框架，教你使用Vue+ElementUI写一个炫酷的后端页面 ](https://github.com/TyCoding/ssm-redis-solr)

<!--more-->

<br/>

**写在前面**

本篇博文主要讲Vue.js+ElementUI如何实现图片上传和提交表单，前端技术会讲多一点，因此：

* 如果你对SpringMVC文件上传和下载不是很清楚，请查看我这篇博文： [SpringMVC实现文件上传和下载](http://tycoding.cn/2018/05/31/Spring-6/#more)
* 因为案例基于SSM框架，如果你你对SSM框架不是很清楚，请查看我这篇博文：[SSM框架整合](http://tycoding.cn/2018/06/05/ssm-2/#more)  [GitHub](https://github.com/TyCoding/ssm)



# 准备

**首先**，请一定阅读一下我的 [SpringMVC实现文件上传和下载](http://tycoding.cn/2018/05/31/Spring-6/) 本篇博文将不在详细讲这部分内容。

**前端：**



> 你会用到以下技术：
>
> Vue.js
>
> Vue-resource.js
>
> ElementUI



我们将实现的效果是什么呢？

*图片上传：*

![](https://tycoding.cn/2018/08/05/vue-7/1.png)

*table展示：*

![](https://tycoding.cn/2018/08/05/vue-7/2.png)

# 思路分析

想要实现图片上传和table的回显，让我们先分析以下实现思路：

## 图片上传和表单提交

那么你就要明白图片上传和表单提交是两个功能，其对应不同的接口，表单中并不是保存了这个图片，而仅仅是保存了储存图片的路径地址。我们需要分析以下几点：



**1、图片如何上传，什么时候上传？**

图片应该在点击upload上传组件的时候就触发了对应的事件，当选择了要上传的图片，点击确定的时候就请求了后端的接口保存了图片。也就是说你在浏览器中弹出的选择框中选择了要上传的图片，当你点击确定的一瞬间就已将图片保存到了服务器上；而再点击提交表单的时候，储存在表单中的图片数据仅仅是刚才上传的图片存储地址。



**2、如何获取到已经上传的图片的储存地址？**

因为在浏览器上传选择框被确定选择的瞬间已经请求了后端接口保存了图片，我们该怎么知道图片在哪里储存呢？

* **前端：** 比如我们使用了ElementUI提供的上传组件，其就存在一个上传成功的回调函数：`on-success`，这个回调函数被触发的时间点就是图片成功上传后的瞬间，我们就是要在这个回调函数触发的时候获取到图片储存的地址。
* **后端：** 上面讲了获取地址，这个**地址**就是后端返回给前端的数据（JSON格式）。因为后端图片上传接口配置图片储存的地址，如果图片上传成功，就将图片储存的地址以JSON格式返回给前端。



**3、如何提交表单**

说如何提交表单，这就显得很简单了，因为上面我们已经完成了：1、图片成功上传；2、获取到了图片在服务器上的储存地址。利用Vue的双向绑定思想，在图片成功上传的回调函数`on-success`中获取到后端返回的图片储存地址，将这个地址赋值给Vue实例`data(){}`中定义的表单对象。这样在提交表单的时候仅需要将这个表单对象发送给后端，保存到数据库就行了。



## 图片在table的回显

想要将图片回显到table表格中其实很简单，前提只要你在数据库中保存了正确的图片储存地址；在table表格中我们仅需要在`<td>`列中新定义一列`<td><img src="图片的地址"/></td>`即可完成图片回显。渲染table数据的时候循环给`<img>`中的`src`赋值数据库中保存的图片url即可。

<br/>

# 后端实现

<br/>

## 图片上传接口

**注意：** 关于SpringMVC如何实现文件上传和下载，请看我的博文： [SpringMVC实现文件上传和下载](http://tycoding.cn/2018/05/31/Spring-6/) 。这里我给出代码，就不再解释了(#^.^#)：

这里我将文件上传和下载接口单独抽离在一个Controller类中：

```java
import com.instrument.entity.Result;

@RestController
public class UploadDownController {

    /**
     * 文件上传
     * @param picture
     * @param request
     * @return
     */
    @RequestMapping("/upload")
    public Result upload(@RequestParam("picture") MultipartFile picture, HttpServletRequest request) {

        //获取文件在服务器的储存位置
        String path = request.getSession().getServletContext().getRealPath("/upload");
        File filePath = new File(path);
        System.out.println("文件的保存路径：" + path);
        if (!filePath.exists() && !filePath.isDirectory()) {
            System.out.println("目录不存在，创建目录:" + filePath);
            filePath.mkdir();
        }

        //获取原始文件名称(包含格式)
        String originalFileName = picture.getOriginalFilename();
        System.out.println("原始文件名称：" + originalFileName);

        //获取文件类型，以最后一个`.`为标识
        String type = originalFileName.substring(originalFileName.lastIndexOf(".") + 1);
        System.out.println("文件类型：" + type);
        //获取文件名称（不包含格式）
        String name = originalFileName.substring(0, originalFileName.lastIndexOf("."));

        //设置文件新名称: 当前时间+文件名称（不包含格式）
        Date d = new Date();
        SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddHHmmss");
        String date = sdf.format(d);
        String fileName = date + name + "." + type;
        System.out.println("新文件名称：" + fileName);

        //在指定路径下创建一个文件
        File targetFile = new File(path, fileName);

        //将文件保存到服务器指定位置
        try {
            picture.transferTo(targetFile);
            System.out.println("上传成功");
            //将文件在服务器的存储路径返回
            return new Result(true,"/upload/" + fileName);
        } catch (IOException e) {
            System.out.println("上传失败");
            e.printStackTrace();
            return new Result(false, "上传失败");
        }
    }
}
```

**为什么返回一个Result数据类型？**

注意这个`Result`是我自己声明的一个实体类，用于封装返回的结果信息，配合`@RestController`注解实现将封装的信息以JSON格式return给前端，最后看下我定义的`Result`:

```java
public class Result implements Serializable {

    //判断结果
    private boolean success;
    //返回信息
    private String message;

    public Result(boolean success, String message) {
        this.success = success;
        this.message = message;
    }
    
    public boolean isSuccess() {
        return success;
    }
    
    setter/getter...
}
```





## 表单提交接口

表单提交大家都比较熟悉了，配合图片上传，仅仅是在实体类中多了一个字段存放图片的URL地址：

```java
@RestController
@RequestMapping("/instrument")
public class InstrumentController {

    //注入
    @Autowired
    private InstrumentService instrumentService;

    /**
     * 添加
     *
     * @param instrument
     * @return
     */
    @RequestMapping("/save")
    public Result save(Instrument instrument) {
        if(instrument != null){
            try{
                instrumentService.save(instrument);
                return new Result(true,"添加成功");
            }catch (Exception e){
                e.printStackTrace();
            }
        }
        return new Result(false, "发生未知错误");
    }
}
```



**如上**

大家可能会疑惑这个为什么返回Result类型的数据？ 答：为了前端方便判断接口执行成功与否。因为我前端使用的是**HTML页面**，想要从后端域对象中取数据显然就有点不现实了。

我写Controller的时候定义了全局的`@RestController`注解，和`@Controller`注解的区别是，前者多了`@ResponseBody`注解，这样整合Controller类返回的数据都将给自动转换成JSON格式。



<br/>

# 前端实现

<br/>

## 实现图片上传

这里我使用了ElementUI的文件上传组件： [官方文档](http://element-cn.eleme.io/#/zh-CN/component/upload) 

配合ElementUI的上传组件，我们会这样定义(这是form表单中的一部分)：

```html
<el-form-item label="图片">
    <el-upload
               ref="upload"
               action="/upload.do"
               name="picture"
               list-type="picture-card"
               :limit="1"
               :file-list="fileList"
               :on-exceed="onExceed"
               :before-upload="beforeUpload"
               :on-preview="handlePreview"
               :on-success="handleSuccess"
               :on-remove="handleRemove">
        <i class="el-icon-plus"></i>
    </el-upload>
    <el-dialog :visible.sync="dialogVisible">
        <img width="100%" :src="dialogImageUrl" alt="">
    </el-dialog>
</el-form-item>
```

注意，我这里仅展示了文件上传的`form-item`，ElementUI的表单声明是：`<el-form>` **注意** 表单中不需要指定`enctype="multipart/form-data"`这个参数，与我们普通的文件上传表单是不同的。

了解几个参数：

* **ref** `ref`是Vue原生参数，用来给组件注册引用信息。引用信息将会注册到父组件的`$refs`对象上，如果定义在普通的DOM元素上，那么`$refs`指向的就是DOM元素。



* **action** `action`表示此上传组件对应的上传接口，此时我们使用的是后端Controller定义的接口



* **name** `name`表示当前组件上传的文件字段名，需要和后端的上传接口字段名相同 。



* **list-type** 文件列表的类型，主要是文件列表的样式定义。这里是卡片化。



* **:limit** 最大允许上传的文件个数。



* **file-list** 上传的文件列表，这个参数用于在这个上传组件中回显图片，包含两个参数：`name、url`如果你想在这个文件上传组件中咱叔图片，赋值对应的参数即可显示，比如更新数据时，其表单样式完全和添加表单是相同的。但是table中回显图片是完全不需要用这个方式的。



* **:on-exceed** 上传文件超出个数时的钩子函数。



* **:before-upload** 上传文件前的钩子函数，参数为上传的文件，返回false，就停止上传。



* **:on-preview** 点击文件列表中已上传的文件时的钩子函数



* **:on-success** 文件上传成功的钩子函数



* **:on-remove** 文件列表移除时的钩子函数



* **:src** 图片上传的URL。



**JS部分**

```javascript
//设置全局表单提交格式
Vue.http.options.emulateJSON = true;

new Vue({
    el: '#app',
    data(){
        return{
            //文件上传的参数
            dialogImageUrl: '',
            dialogVisible: false,
            //图片列表（用于在上传组件中回显图片）
            fileList: [{name: '', url: ''}],
        }
    },
    methods(){
   		//文件上传成功的钩子函数
        handleSuccess(res, file) {
            this.$message({
                type: 'info',
                message: '图片上传成功',
                duration: 6000
            });
            if (file.response.success) {
                this.editor.picture = file.response.message; //将返回的文件储存路径赋值picture字段
            }
        },
        //删除文件之前的钩子函数
        handleRemove(file, fileList) {
            this.$message({
                type: 'info',
                message: '已删除原有图片',
                duration: 6000
            });
        },
        //点击列表中已上传的文件事的钩子函数
        handlePreview(file) {
        },
        //上传的文件个数超出设定时触发的函数
        onExceed(files, fileList) {
            this.$message({
                type: 'info',
                message: '最多只能上传一个图片',
                duration: 6000
            });
        },
        //文件上传前的前的钩子函数
        //参数是上传的文件，若返回false，或返回Primary且被reject，则停止上传
        beforeUpload(file) {
            const isJPG = file.type === 'image/jpeg';
            const isGIF = file.type === 'image/gif';
            const isPNG = file.type === 'image/png';
            const isBMP = file.type === 'image/bmp';
            const isLt2M = file.size / 1024 / 1024 < 2;

            if (!isJPG && !isGIF && !isPNG && !isBMP) {
                this.$message.error('上传图片必须是JPG/GIF/PNG/BMP 格式!');
            }
            if (!isLt2M) {
                this.$message.error('上传图片大小不能超过 2MB!');
            }
            return (isJPG || isBMP || isGIF || isPNG) && isLt2M;
        },     
    }
});
```

**解释**

如上的JS代码，主要是定义一些钩子函数，这里我么里梳理一下逻辑：

1、点击ElementUI的上传组件，浏览器自动弹出文件上传选择窗口，我们选择要上传的图片。

2、选择好了要上传的图片，点击弹窗右下角的确定按钮触发JS中定义的钩子函数。

3、首先触发的钩子函数是`beforeUpload(file)`函数，其中的参数`file`即代表当前上传的文件对象，`beforeUpload()`定义了对上传文件格式校验。如果不是允许的格式就弹出错误信息，并阻止文件上传，若我那件格式允许，则继续执行。

4、通过了`beforeUpload()`函数的校验，文件开始调用后端接口将数据发送给后端。文件的字段名：`picture`，格式：`multipart/form-data`，虽然我们的表单没有定义`enctype="multipart/form-data"`属性，但是HTTP请求头会自动设置为`multipart/form-data`类型。

![](https://tycoding.cn/2018/08/05/vue-7/3.png)

5、这时，如果后端逻辑没有错误，已经正常的将图片上传到服务器上了，可以在指定文件夹中查看到已上传的图片，那么此时JS中会自动调用`handleSuccess()`钩子函数，因为我们设置后端上传接口上传成功返回的数据是文件的保存路径：

![](https://tycoding.cn/2018/08/05/vue-7/4.png)

那我们就将这个路径通过Vue的双向绑定，赋值给表单对象的字段`picture`，那么提交表单的时候，该字段对应的值就是这个路径了。

6、如果我们再点击上传文件按钮，就会触发`onExceed()`函数，因为我们设置的`limit`最多上传一个。

7、如果点击图片中的删除按钮，就会触发`handleRemove()`函数，并删除此图片。

8、如果点击了已上传的文件列表，就会触发`handlePreview()`函数。



## 实现表单提交

表单提交就比较简单了，就是触发对应的click事件，触发其中定义的函数，将已在`data(){}`中定义的表单数据发送给后端接口：

![](https://tycoding.cn/2018/08/05/vue-7/5.png)

提交数据：

![](https://tycoding.cn/2018/08/05/vue-7/6.png)

**后端接口**

```java
@RequestMapping("/save")
public Result save(Instrument instrument) {
    if(instrument != null){
        try{
            instrumentService.save(instrument);
            return new Result(true,"添加成功");
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    return new Result(false, "发生未知错误");
}
```

数据库中保存的数据：

![](https://tycoding.cn/2018/08/05/vue-7/7.png)



<br/>

# 实现table回显图片

table回显图片也是很简单的，仅需要在列中增加一列：

```html
<el-table :data="instrument">
    <el-table-column label="图片" width="130">
        <template scope="scope">
            <img :src="scope.row.picture" class="picture"/>
        </template>
    </el-table-column>
    <el-table-column
         label="运行状态"
         width="80"
         prop="operatingStatus">
    </el-table-column>
</el-table>
```

因为使用Vue，根据其双向绑定的思想，再结合Element-UI提供渲染表格的方式是在`<el-table>`的`:data`中指定对应要渲染的数据即可。

**注意** ElementUI渲染table的方式是：1、`<el-table>`中定义`:data`；2、`<el-table-column>`中定义`prop="data中的参数"`。但是因为我们要显示的是图片而不是文本数据，所以要在`<img>`中定义`:src="data中的变量"`即可实现渲染。

<br/>

**后端**就是正常的查询数据库数据即可了，为什么数据库中保存了这个URL图片就能直接显示到HTML中，请看我这篇博文： [SpringMVC实现文件上传和下载](http://tycoding.cn/2018/05/31/Spring-6/) 
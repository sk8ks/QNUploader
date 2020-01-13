# QNUploader
基于七牛云 jssdk 1.x 版本封装的上传库，优化各个生命周期交互。


## 初始化
在业务中引入七牛云官方jssdk及本库文件'uploader.js'。
```js
// 这里使用vue组件示例:
import QNUploader from '路径/uploader.js';
// 组件中使用示例
export default {
    ...,
    mounted () {
        this.uploader = QNUploader({
            browse_button:          'J_Uploader',           // 上传选择的点选按钮，可以是id或者元素本身，必需
            dragdrop:               false,                  // 是否可拖曳上传      
            drop_element:           '',                     // 拖拽区域元素
            domain:                 '',                     // bucket域名，下载资源时用到，必需
            uptoken:                '',                     // 上传凭证
            get_new_uptoken:        false,                  // 设置是否每次都重新获取新的uptoken
            fetch_upload_token:     function(){},           // 获取上传token方法
            file_type:              'image',                // 上传文件类型
            unique_names:           true,                   // 是否开启自动文件名
            save_key:               false,                  // 是否服务端生成uptoken
            key_func:               function(){},           // 自定义文件名方法
            multi_selection:        true,                   // 是否多选
            max_file_size:          '5mb',                  // 文件上传大小限制
            chunk_size:             '5mb',                  // 分块上传时，每块的体积
            type_reg:               {},                     // 文件类型校验配置
            max_files:              5,                      // 最大上传数
            fileFiltered:           this.fileFiltered,                  // 文件添加前事件
            filesAdded:             this.filesAdded,                    // 文件添加后事件
            beforeUpload:           this.beforeUpload,                  // 上传前事件
            uploadProgress:         this.fileUploadProgress,            // 上传进行中
            fileUploaded:           this.fileUploaded,                  // 上传完事件
            uploadFail:             this.fileUploadFail,                // 上传失败事件
            uploadComplete:         this.fileUploadComplete,            // 上传结束事件
            
        });
    },
}
```

## 配置详情
属性配置信息：

| 属性 | 类型 | 必填 | 默认值 | 平台 | 说明 |
| - |:-: | :-:| :-:| -:| :- |
| <em class="req">*</em>browse_button | string,HtmlElement | 是 | upload | `pc,mobile` | 上传选择的点选按钮，可以是id或者元素本身 |
| <em class="req">*</em>domain | string | 是 | | `pc,mobile` | 七牛bucket域名，下载资源时用到 |
| uptoken | string | 否 | | `pc,mobile` | 上传token，字符串，与fetch_upload_token二选一 |
| fetch_upload_token | function | 否 | | `pc,mobile` | 获取上传token方法，如有uptoken，则优先使用uptoken，与uptoken二选一 |
| get_new_uptoken | boolean | 否 | true | `pc,mobile` | 设置上传文件的时候是否每次都重新获取新的uptoken |
| unique_names | boolean | 否 | false | `pc,mobile` | 是否开启自动文件名，开启时由插件创建上传文件名 |
| save_key | boolean | 否 | false | `pc,mobile` | 默认false。若在服务端生成uptoken的上传策略中指定了saveKey，则开启，SDK在前端将不对key进行任何处理 |
| dragdrop | boolean | 否 | true | `pc` | 是否可拖曳上传 |
| drop_element | string,HtmlElement | 否 | | `pc` | 拖曳上传区域元素的ID，拖曳文件或文件夹后可触发上传 |
| chunk_size | string | 否 | '5mb'或'5' | `pc,mobile` | 分块上传时，每块的体积 |
| file_type | string | 否 | image | `pc,mobile` | 上传文件类型, 字符串，多个类型逗号隔开: 'image, doc' |
| key_func | function | 否 | | `pc,mobile` | 自定义文件名方法，只有unique_names为false有效 |
| multi_selection | boolean | 否 | false | `pc,mobile` | 是否开启多文件上传，布尔 |
| max_file_size | string | 否 | 5mb | `pc,mobile` | 单文件上传大小限制，单位为mb，字符串，数值型字符串或者数值+后缀'mb'，例如：'1mb'、'0.5' |
| max_files | number | 否 | 10 | `pc,mobile` | 最大上传文件数量 |
| type_reg | object | 否 | {} | `pc,mobile` | 文件类型校验配置，一个{key: value}对象，key表示上传类型，value：1、一个校验方法，参数是后缀名，返回布尔值；2、一个正则表达式。例如：{image: function(ext){return !0}, execl: /(\.xls[x]?)$/} |

fetch_upload_token方法使用示例
```js
/**
 * @desc    获取七牛token方法，如成功会执行七牛设置token并开始七牛上传流程
 * @param   {Object}     up     七牛上传对象
 * @param   {Function}    next    执行后续上传流程
 */
fetchUploadToken (up, next) {
    // 获取上传token的请求，开发者自行处理
    uploadTokenAction()
        .then( ( { data, msg } ) => {
            if ( data && typeof data.up_token === 'string' && data.up_token.trim() ) {
                // 传入token，并执行后续上传流程
                next(data.up_token);
            } else {
                // token获取失败的处理
                console.warn(`token获取失败`);
            }
        })
        .catch( err => {
            console.warn(`请求失败，请重新尝试`);
        } );
}
```


事件配置信息：

| 事件 | 说明 | 参数  | 参数说明 |
| ------------- |:-------------|:-------------:|:-------------|
| fileFiltered | 文件添加前事件 | uploader, file, error | 上传对象, 当前上传文件, 错误对象 |
| filesAdded | 文件添加后事件 | uploader, files, error | 上传对象, 当前上传文件列表（数组）, 错误对象 |
| beforeUpload | 上传前事件 | uploader, file, error | 上传对象, 当前上传文件, 错误对象 |
| uploadProgress | 上传进行中 | uploader, file | 上传对象, 当前上传文件 |
| fileUploaded | 上传成功事件 | uploader, file, info | 上传对象, 当前上传文件, 返回的上传数据信息 |
| uploadFail | 上传失败事件 | uploader, error | 上传对象, 返回的错误对象 |
| uploadComplete | 上传完成事件，不论失败或成功 | uploader | 上传对象 |


```js
/**
 * @desc    文件添加前，一般对自定义文件名及文件类型校验
 * @param  {[type]}     up      七牛上传对象
 * @param  {[type]}     file    上传文件
 * @param   {Object}    error    当前事件错误信息
 */
fileFiltered ( up, file, error) {
    if (error) {
        // 错误信息处理
        console.warn(error.msg);
        return;
    }
    // 没有错误信息就执行后续操作
}
```
```js
/**
 * @desc 文件添加后
 * @param   {Object}    up      七牛上传对象
 * @param   {Object}    files    上传文件列表
 * @param   {Object}    error    当前事件错误信息
 */
filesAdded (up, files, error) {
    if (error) {
        // 错误信息处理
        console.warn(error.msg);
        return;
    }
    // 没有错误信息就执行后续操作
}
```
```js
/**
 * @desc 上传前事件
 * @param   {Object}    up      七牛上传对象
 * @param   {Object}    file    上传文件
 * @param   {Object}   error     当前事件错误信息
 */
beforeUpload (up, file, error) {
    if (error) {
        // 错误信息处理
        console.warn(error.msg);
        return;
    }
    // 没有错误信息就执行后续操作
}
```
```js
/**
 * @desc 文件上传进行中
 * @param   {Object}    up      七牛上传对象
 * @param   {Object}    file    上传文件
 */
fileUploadProgress (up, file) {
    // 处理上传进度
    console.log(file.percent);
}
```
```js
/**
 * @desc 文件上传成功
 * @param {Object} up   七牛上传对象
 * @param {Object} file   上传文件
 * @param {Object} info   上传信息
 */
fileUploaded (up, file, info) {
    // 上传成功后操作
    let key = info.key;     // 七牛文件名
    let fileName = file.oriName;       // 原始文件名
}
```
```js
/**
 * @desc 文件上传错误回调
 * @param {Object}  up  上传对象
 * @param {Object}  error   上传错误信息
 */
fileUploadFail (up, error) {
    // 处理上传失败...
    console.warn(error.msg);
}
```
```js
/**
 * @desc 文件上传完成（不管成功与失败）
 * @param {Object}  up  上传对象
 */
fileUploadComplete (up) {
    // 上传流程完成
}
```

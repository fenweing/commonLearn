1、form-data: 

                   就是http请求中的multipart/form-data,它会将表单的数据处理为一条消息，以标签为单元，用分隔符分开。既可以上传键值对，也可以上传文件。当上传的字段是文件时，会有Content-Type来表名文件类型；content-disposition，用来说明字段的一些信息；

由于有boundary隔离，所以multipart/form-data既可以上传文件，也可以上传键值对，它采用了键值对的方式，所以可以上传多个文件。

![image](https://user-images.githubusercontent.com/15857347/118907199-b1b74680-b951-11eb-9f2c-e76c16380c08.png)


![image](https://user-images.githubusercontent.com/15857347/118907225-bc71db80-b951-11eb-934e-dd13f3700d2a.png)

                 

2、x-www-form-urlencoded：

             就是application/x-www-from-urlencoded,会将表单内的数据转换为键值对，比如,name=java&age = 23
![image](https://user-images.githubusercontent.com/15857347/118907313-df03f480-b951-11eb-87a0-ea448cc1f705.png)


![image](https://user-images.githubusercontent.com/15857347/118907314-df9c8b00-b951-11eb-9505-1705cfea7d79.png)


3、raw

            可以上传任意格式的文本，可以上传text、json、xml、html等
![image](https://user-images.githubusercontent.com/15857347/118907328-e4f9d580-b951-11eb-8681-e3ecd2247484.png)

![image](https://user-images.githubusercontent.com/15857347/118907340-e9be8980-b951-11eb-9efd-7ce3ffc9c168.png)


4、binary

          相当于Content-Type:application/octet-stream,从字面意思得知，只可以上传二进制数据，通常用来上传文件，由于没有键值，所以，一次只能上传一个文件。



multipart/form-data与x-www-form-urlencoded区别

               multipart/form-data：既可以上传文件等二进制数据，也可以上传表单键值对，只是最后会转化为一条信息；

               x-www-form-urlencoded：只能上传键值对，并且键值对都是间隔分开的。

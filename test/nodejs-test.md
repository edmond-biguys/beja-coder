本地测试环境部署，使用node.js，先来配置node.js的环境，如下，是node的官网，  
[node.js官网(https://nodejs.org/en/download)

在上一步，安装好nodejs基础上执行下边的功能，  
express使用
安装express：```npm install express -g```
安装express generator：```npm install express-generator -g```

-g 表示全局安装，-global

输入 ```express mockserver```生成一个名为mockserver的项目

输入```npm start```启动服务
在浏览器输入 http://localhost:3000/ 就可以看到页面了。

如果要新加接口，可以模仿users.js（routes目录下）的内容，修改app.js（根目录下）下的内容，代码大致如下，  
```var usersRouter = require('./routes/users');```  
```app.use('/users', usersRouter);```

访问users.js，输入内容为http://localhost:3000/users

users.js下的代码如下，  
```
var express = require('express');
var router = express.Router();

/* GET users listing. */
router.get('/', function(req, res, next) {
	
	console.log('12345');
	console.log(req.params);
	
	
	var data = {
		'name':'name1',
		'id':12
	}
	
  //res.send('respond with a resource a');
  //res.send('abc');
  
  /*
  res.send({
    meta: {
      message: 'success'
    },
    status: true,
    data: data
  });
  */
  
  res.send(data);
});

module.exports = router;
```

# Project

## 1 创建项目

前后端分离 -- 后端职位前端提供接口服务 后端和前端面向接口编程。

前后端商量好接口api  和接口参数，用json的方式发送数据。

1. 新建background 文件夹
2. 新建frount 文件夹

## api

### user

属性：username   password avatar email createAT updateAT likeMovies 

login : /api/user/login      post

username password 

{ message:"success",  

​	success:true,

​	statusCode:200,

​		data:{

​		username,

​		userID

​	}

}

{ message:"fail",

​	success:false,

​	statusCode:200

}

regist: /api/user/regist   post

username password  email avatar 

{ message:"success",

​	success:true,

​	statusCode:200,

​		data:{

​		username,

​		userID

​	}

}

{ message:"fail",

​	success:false,

​	statusCode:200

}

likeMovie : /api/user/likeMovie   post

userID  movieID

{message:"success",

​	success:true,

​	statusCode:200

}

{ message:"fail",

​	success:false,

​	statusCode:200

}

### 电影

属性 ： title info pics score type releaseDate length

getallmovie : /api/movie/movies / 参数     get   所有人都可以看的  不需要登录

根据  获取的参数给你电影   可以根据参数动态查询电影   10  ==》 10  

{ message:"success",

​	success:true,

​	statusCode:200,

​		data:{

​		movies  // todo

​	}

}

{ message:"fail",

​	success:false,

​	statusCode:200

}



### 分类

type 

动作  爱情








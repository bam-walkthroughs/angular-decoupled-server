# Setting Up a Server in Node/Express

### Objective:
The purpose of this walkthrough is to build the server and the API that will allow our front end to communicate with the database we created in the previous walkthrough.

## SERVER SETUP

1) Using the [express generator](https://expressjs.com/en/starter/generator.html), build your files and structure.
 	In your terminal type `express .`

 -- This will make files
 
	 - Bin
	 - Public
	 - Routes
	 - Views  
	 - App.js
	 - Package.json
	 
2) Rename folder 'routes' to 'api'. This is where we will make the routes for our api calls.

#####(2.1)

- In the routes directory remove users.js we will not be using this.
- Remove the 'views' directory, we are not rendering anything.
- Remove the 'public' directory.

3) Go into app.js file and on line 8 Change

`var routes = require('./routes/index'); `

to

`var api = require('./api/index');`

- Find the following in app.js (it should be around lines 27-28)
`app.use(‘/’ index)`
to
`app.use(‘/api’,api);`

4) Remove the following from the app.js file.
#####(4.1)
 ```
 var users = require('./routes/users');
	-// view engine setup
```	
```
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');
app.use(express.static(path.join(__dirname, 'public')));
```


```  
app.use('/users', users);
```



5) In the app.js file change `res.render` to `res.json` in the following code blocks.

#####(5.1)
development error handler

  ```
if (app.get('env') === 'development') {
  app.use(function(err, req, res, next) {
 		   res.status(err.status || 500);
   		 res.json({
     		 message: err.message,
    		  error: err
   		 });
  });
}
```
production error handler

```
	app.use(function(err, req, res, next) {
 	 res.status(err.status || 500);
 		 res.json( {
  		  message: err.message,
   		 error: {}
  });
});
```


6) In terminal `npm initstall`, this will make the node_modules in your root directory.

7) In terminal `npm install gitignore -g` (if you did the first walkthrough, you should have gitignore installed globally)

8) In terminal `knex init`

  - This creates a knexfile.js in your root directory.

#####(8.1)
Go in to knexfile.js and change it to be the same as your knexfile.js in your database repo.

*Example*
	
  ```
	require('dotenv').config();
module.exports = {
  		development: {
   		 client: 'pg',
    		connection: 'postgres://localhost/YOUR DATABASE NAME '
 	 },
 	 production:{
    		client: 'pg',
    		connection: process.env.DATABASE_URL + '?ssl=true'
  		}
};
```


9) Next in your terminal `npm install knex pg dotenv --save`

- 'dotenv' installs the dotenv module and adds it to the dependencies in package.json (dotenv stores and loads the environment variables).
- 'pg' and 'knex' installs them and adds the two to the dependencies in the package.json file.



10) Initialize a git repo by typing `git init` in your terminal.


11) In your terminal `echo node_modules > .gitignore && touch .env && echo .env >> .gitignore`
		- This moves node_modules to a .gitignore file, makes an empty .env file and then adds that file to the .gitignore.


12) In your terminal make folder db `mkdir db && cd db && touch knex.js && atom . `

13) In knex.js copy the following.

```
var environment = process.env.NODE_ENV || 'development';
var config = require('../knexfile')[environment];
var knex = require('knex')(config);
module.exports= knex;
```


## Setting Up the API

1) In directory 'api' make file a file called 'list.js'

2) In your root make a folder called 'queires' and inside of it make a file called 'apiQueries.js'


3) Require files in index.js

```
var express = require('express');
var router = express.Router();
var knex = require('../db/knex');
var queries = require('../queries/apiQueries')
//change to use the list.js file
var list = require('./list');
router.use('/list', list);
module.exports = router;
```

4) In list.js add the follwing requirements

```
var express = require('express');
var router = express.Router();
var knex = require('../db/knex');
var queries = require('../queries/apiQueries');
module.exports = router;
```


5) In 'apiQueries.js' inside of the 'queries' directory add the following

```
var knex = require('../db/knex');
module.exports ={
// write functions for queries here (we will go over that later)
};
```

6) Writing a test function inside of 'apiQueries.js' this is to insure that all files are hooked up correctly and all of our requires are in place. Write the queiry inside of the module.exports.

```
test: function(){
  console.log("hello");
}
```
Then in list.js add the following, this will call the function when we make a get request to the '/' route.

```
router.get('/', function(req, res, next){
  	queries.test().then(function(data){
 	 	res.json({test:'hello'});
  			});
 	 	});
```
 	 	
(6.1) Start nodemon and then In postman (an app for testing APIs available through the Chrome App Store) hit the url params.

http://localhost:3000/api/list
If the test worked you will see 'hello' in the console

(6.2) in index.js remove the query and add the following:

```
router.get('/', function(req, res, next){
   		 res.json({test:'hello 2'});
  });
  ```
Then in postman hit url http://localhost:3000/api/list if it worked you will get 'hello 2' back in the json object under test

##ROUTES SETUP

1) Inside of apiQueries.js make a getData function

```
getData : function(){
		Return knex(‘users’).innerJoin(‘list”, “user_id”, “user.id”);
	}
```
	
2) Inside of list.js call getData() in a `.get`

```
	router.get('/', function(req, res, next){
  queries.getData()
  .then(function(data){
    		  res.json({list: data});
 		 });
});
```

3) Inside of apiQueries.js make getOneData() function

```
	 getOneData : function(id){
 		 return knex('users').innerJoin('list', 'users_id', 'users.id').where({users_id : id});
  }
```  
4) Inside of list.js use getOneData() function inside of a `.get`

```
router.get('/:id', function(req, res, next){
 	 queries.getOneData(req.params.id)
  	.then(function(data){
   	   res.json({list: data});
  });
});
```
5) Inside of apiQueries.js make addListItem() function.

```
	addListItem : function(list, user){
  		return knex('list').insert({'list': list, 'users_id': user});
},
```
6) Inside list.js use the addListItem function in a `.post`

```
	router.post('/', function(req, res, next){
  queries.addListItem(req.query.list, req.query.users_id)
  .then(function(data){
      res.json({message: 'list item added'});
  });
});
```
7) Inside apiQueries.js make deleteItem() function

```
deleteItem : function(body){
  return knex('list').del().where({id: body});
```  

8) Inside list.js use the deleteItem() function in the `.delete` route.

```
	router.delete('/:id', function(req, res, next){
  	queries.deleteItem(req.query.list_id)
 	 .then(function(data){
     	 res.json({list: data});
 	 });
});
```
9) Inside of apiQueries.js make editItem() function.

```
editItem : function(id, editListItem){
  return knex('list').update({list: editListItem}).where({id: id});
},
```

10) Inside list.js use the editItem() function in the `.put` route.

```
router.put('/:id', function(req, res, next){
  queries.editItem(req.query.list_id, req.query.editListItem)
  .then(function(data){
      res.json({list: data});
  });
});
```


##SETTING UP AUTH

1) In your terminal  `npm install jsonwebtoken  bcrypt  dotenv --save`

2) In your root directory  make folder 'auth' and inside of 'auth' make two files 'index.js' and 'helpers.js'. 

```
mkdir auth && cd auth && touch index.js helpers.js

```

3) Go into the 'auth' directory and inside of 'index.js' add:

```
	require('dotenv').config();
var express = require('express');
var router = express.Router();
var queries = require('./queries/apiQueries')
var auth = require('./helpers.js');
var bcrypt = require('bcrypt');
var jwt = require('jsonwebtoken');

module.exports = router;

```

4) Go in to helpers.js and add the following

```
var queries = require('../queries/apiQueries');
var bcrypt = require('bcrypt');
var jwt = require('jsonwebtoken');
var knex= require('../db/knex');
var express = require('express');
```

5) Go to app.js and add the following: 

```
	var auth = require('./auth/index')
	app.use('/auth', auth); //make sure this is above the app.use(‘/api’,api)
```	

6) Go to 'helpers.js' inside of the 'auth' directory.

```
module.exports={
	 createUser : function(body){
    	//take the password from the body of the request, hashing it
    	//then setting the req.body.password to the hashed password
    	var hash = bcrypt.hashSync(body.password, 8);
    	body.password = hash;
    	//adds the user to the database with the hashed password
    	return queries.addUser(body)
    	.then(function(user){
      	return user.id;
    	});
  	}
 } 
  ```
7) Inside of 'index.js' in the auth directory make the following route:

```
	router.post('/signup', function(req, res, next){
  queries.findUserByUserName(req.query.userName)
  .then(function(user){
    if(user){
      res.json({
        error : "user already exist try another name"
      });
    }else {
      auth.createUser(req.query)
      .then(function(user){
        res.json({
          message : 'you are a new user yay'
        });
      });
    }
  });
});
```

8) Next make a route for login.

```
router.post('/login', function(req, res, next){
  queries.findUserByUserName(req.query.userName)
  .then(function(user){
    var plainTextPassword = req.query.password;
    if(user && bcrypt.compareSync(plainTextPassword, user.password)){
      jwt.sign(user, process.env.TOKEN_SECRET, {expiresIn: '1d'}, function(err, token){
        if(err){
          res.json({
            message: 'error creating token'
          });
        }else{
          res.json({
            token : token,
            userId: user.id
          });
        }
      });
      }else{
      res.status(401);
      res.json({
        message : 'unauthorized'
      });
    }
  });
});
```






9) In 'helper.js' inside of the 'auth' directory make a function authMiddleWare.

```
authMiddleWare : function(req, res, next){
    //Authorization is part of the header that is being sent
    
    var token = req.get('Authorization');
    if(token){
      //removing "Bearer " frome the token string, leaving only the token
      token = token.substring(7);
      //a method of JWT that verifies the payload, token, and takes a callback
      jwt.verify(token, process.env.TOKEN_SECRET, function(error, decoded){
        if(error){
          next();
        }else{
          req.user = decoded;
          next();
        }
      });
    }else{
      next();
    }
  },
 ``` 
10) Next make a function ensureauthenticated.

```
  ensureauthenticated : function(req, res, next){
    if(req.user){
      next();
    }else{
      res.json({
        message : "you cant come here buddy"
      });
    }
  }
};
```

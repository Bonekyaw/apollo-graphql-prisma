# This is Simple Nodejs Express Starter Kit

#### Apollo server + Prisma ORM + SDL modulerized - graphql api 

You can use it for your project. If it is useful for you,  
don't forget to give me a **GitHub star**, please.

In this template

   - Apollo server 
   - DB - MySQL or PostgreSQL 
   - Prisma ORM
   - SDL-first modulerized ( graphql.js, graphql-tools) 
   - JWT auth
   - bcrypt
   - validator 
   - error handler 
   - Authorization
   - Pagination ( offset-based & cursor-based ) etc.

In order to use it,

**Create** a .env file and add this.  
For **MySQL**

```
DATABASE_URL="mysql://username:password@localhost:3306/mydb"
TOKEN_SECRET="something hard to guess"

```
For **PostgreSQL**

```
DATABASE_URL="postgresql://username:password@localhost:5432/mydb?schema=public"
TOKEN_SECRET="something hard to guess"

```
Please note.   
*TOKEN_SECRET* should be complex and hard to guess.  
  
Warning - every `*.graphql & *.resolver.js` must be inside three nested folders `./**/**/**/file` .  
It's ok `./src/graphql/auth/auth.graphql` or `./src/graphql/product/product.graphql` or something like that.  

## Step by Step Installation

```bash
mkdir lucky
cd lucky
git clone https://github.com/Bonekyaw/apollo-graphql-prisma.git .
rm -rf .git
npm install
npm start

```  
Before you run, make sure you've created .env file and completed required information.  

I'm trying best to provide the **latest** version. But some packages may not be latest after some months. If so, you can upgrade manually one after one, or you can upgrade all at once. 

```bash
npm install -g npm-check-updates
npm outdated
ncu --upgrade
npm install
```
If you find some codes not working well, please let me know your problems.   

For Graphql Query, use Postman.  
You will see everything about graphql queries. Thanks, Postman. 
```graphql
mutation Register {
    register(phone: "0977*******7") {
        message
        phone
        token
    }
}

mutation VerifyOtp {
    verifyOtp(
        userInput: { token: "3llh4zb6rkygbrah5demt7", phone: "77******7", otp: "123456" }
    ) {
        message
        phone
        token
    }
}

mutation ConfirmPassword {
    confirmPassword(
        token: "xdyj8leue6ndwqoxc9lzaxl16enm0gkn"
        userInput: { phone: "77*******7", password: "12345678" }
    ) {
        message
        token
        phone
        userId
        randomToken
    }
}

mutation Login {
    login(userInput: { phone: "0977******7", password: "12345678" }) {
        message
        token
        phone
        userId
        randomToken
    }
}

mutation RefreshToken {
    refreshToken(
        userInput: { userId: "1", randomToken: "b6x9na0z5abc7wix1t2ojj5hdkk7aosm6" }
    ) {
        message
        token
        userId
        randomToken
    }
}

mutation UploadProfile {
    uploadProfile(userInput: { imageUrl: "uploads/images/abc.png" }) {
        message
        imageUrl
    }
}

query PaginateAdmins {
    paginateAdmins(page: 1, limit: 10) {
        total
        data {
            id
            name
            phone
            role
            status
            lastLogin
            profile
            createdAt
        }
        pageInfo {
            currentPage
            previousPage
            nextPage
            lastPage
            countPerPage
            nextCursor
            hasNextPage
        }
    }
}
```
### How to develop your own products using this Starter Kits  

Authorization
```javascript
const authorise = require("../../utils/authorise");
...
authorise(true, user, "admin");
```
`true, "admin"` means the account is allowed only if its role is "admin". `false, "user"` means the account is not allowed if its role is "user".  
`ture, "admin"` === `false, "user", "supplier"`  
`false, "user"` === `true, "admin", "supplier"`
  
`true, user, "admin"` In these parameters, user param is an instance model of the database table.  

In this graphql starter kit,  
I built authentication & authorization with the help of composeResolvers tool.
```javascript
const isAuthenticated = () => (next) => (parent, args, context, info) => {
  checkAdminExist(context.authHeader);
  let token = context.token.split(" ")[1]; // Hey take care!
  if (validator.isEmpty(token.trim()) || !validator.isJWT(token)) {
    throw new GraphQLError("Token must not be invalid.", {
      extensions: {
        code: "BAD REQUEST",
        http: { status: 400 },
      },
    });
  }
  token = validator.escape(token);
  const adminId = isAuth(token);
  info.adminId = adminId;

  return next(parent, args, context, info);
};

const hasRole = (...role) => (next) =>
  asyncHandler(async (root, args, context, info) => {
    let adminId = info.adminId;
    const admin = await Admin.findById(adminId);
    checkAdminExist(admin);
    authorise(false, admin, ...role);
    info.admin = admin;

    return next(root, args, context, info);
  });

const resolversComposition = {
  "Mutation.uploadProfile": [isAuthenticated(), hasRole("user")],
  "Query.paginateAdmins": [isAuthenticated(), hasRole("user")],
};

const composedResolvers = composeResolvers(resolvers, resolversComposition);
module.exports = composedResolvers;
```

### Pagination
There are two ways in pagination: **offset-based** and **cursor-based**. You can read more about pros and cons [here](https://www.prisma.io/docs/orm/prisma-client/queries/pagination). But you can use my pagination logic very easily.  

For offset-based 

```javascript
const { offset, noCount, cursor } = require("./../utils/paginate");
...
let { page, cursors, limit } = args;

const filters = { status: "active" };
const order = { id: "desc" };

return offset(prisma.admin, page, limit, filters, order);
```
For cursor-based
```javascript
const { offset, noCount, cursor } = require("./../utils/paginate");
...
let { page, cursors, limit } = args;

const filters = { status: "active" };
const order = { id: "desc" };

return cursor(prisma.admin, cursors, limit, filters, order);

```   

I promise new features will come in the future if I have much time.

If you have something hard to solve,
DM  
<phonenai2014@gmail.com>  
<https://www.facebook.com/phonenyo1986/>  
<https://www.linkedin.com/in/phone-nyo-704596135>  

#### Find more other Starter kits of mine ?   

`My Kits For REST Api`
  
  [Express + Prisma ORM + mongodb - rest api](https://github.com/Bonekyaw/node-express-prisma-mongodb)  
  [Express + Prisma ORM + SQL - rest api](https://github.com/Bonekyaw/node-express-prisma-rest)  
  [Express + mongodb - rest api](https://github.com/Bonekyaw/node-express-mongodb-rest)  
  [Express + mongoose ODM - rest api](https://github.com/Bonekyaw/node-express-nosql-rest)  
  [Express + sequelize ORM - rest api](https://github.com/Bonekyaw/node-express-sql-rest)  

`My Kits For Graphql Api`

  [Apollo server + Prisma ORM + SDL modulerized - graphql api](https://github.com/Bonekyaw/apollo-graphql-prisma) - Now you are here  
  [Express + Prisma ORM + graphql js SDL modulerized - graphql api](https://github.com/Bonekyaw/node-express-graphql-prisma)  
  [Express + Apollo server + mongoose - graphql api](https://github.com/Bonekyaw/node-express-apollo-nosql)  
  [Express + graphql js + mongoose - graphql api](https://github.com/Bonekyaw/node-express-nosql-graphql)  
  [Express + graphql js + sequelize ORM - graphql api](https://github.com/Bonekyaw/node-express-sql-graphql)  

`Mobile App Development`

[React Native Expo](https://github.com/Bonekyaw/react-native-expo)

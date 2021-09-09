# Rick and Morty

This repository holds two submodules that implements the application requested by Haufe Group challenge. Particularly, the application, must consume the public API of Rick and Morty found at <https://rickandmortyapi.com>, this API needs to be consumed from the back end of our application, so it is fair to think of our back end as a wrapper of the API previously mentioned, we need, however, to add authentication to the service and give our users the capability of having favourite characters. Regarding to the front end it needs to allow a user to authenticate (and optionaly register), show the characters list and a single character view where the user must have the ability to mark a character as favourite. Additionaly, a "not found" view needs to be implemented.

To setup all the system we need to clone this repository like:

```bash
git clone --recursive https://github.com/szz-dvl/rick-and-morty.git
```

Next we need to copy the back end environment file to the backend root folder (feel free to change anything you want):

```bash
cp .env rick-and-morty-back/
```

Now, we need to install the dependencies of both front and back:

```bash
cd rick-and-morty-back/
npm install
npm install --only=dev

cd ../rick-and-morty-front/
yarn install
cd ..
```

Then start the server:

```bash
cd rick-and-morty-back/
npm run serve
```

And finally the front end:

```bash
cd rick-and-morty-front/
yarn start
```

You may want to try with a [production build](https://create-react-app.dev/docs/deployment/) if you have an HTTP server available!

I will now explain the main design of the implemented app.

## [Front End](https://github.com/szz-dvl/rick-and-morty-front)

As required by the specification, the front end must be coded with [React](https://reactjs.org/) and the state must be handled with [Redux](https://redux.js.org/). So the first step I took was to generate a scaffold for the SPA with the following command:

```bash
npx create-react-app frontend --template redux-typescript
```

I choosed to use Typescript because, in my opinion, it is a good practice that any new JS project use Typescript, it may give a little more work at some point, but, when the project grows it will save you a lot of runtime errors. With the scaffold ready, I started with the main structure of the app.

First I wrote a little piece of code to handle the user authentication data in the front side. Some time ago, I used [this](https://www.npmjs.com/package/redux-react-session) npm module to deal with user session, however seems to be a little bit discontinued, so, inspiring myself in this module I wrote a very small [class](https://github.com/szz-dvl/rick-and-morty-front/blob/master/src/session/SessionService.ts) that helped me to handle all the storage stuff. All the methods are static for convenience, the class is just a nice encapsulation method, but we actually don't need neither we want an instance that much this time. Associated to this class you will find a redux [state slice](https://github.com/szz-dvl/rick-and-morty-front/blob/master/src/session/sessionSlice.ts) that will replicate the storage data into the state, to make it fully accessible to the app. Notice, that in the root folder of the front app you will find an enviroment file ([.env](https://github.com/szz-dvl/rick-and-morty-front/blob/master/.env)) with a variable ```REACT_APP_STORAGE``` that you can set to either "local" (to use localStorage) or "session" (to use sessionStorage), being this last one the default option. By the way, environment variables are imported using [dotenv](https://www.npmjs.com/package/dotenv).

At this point we are able to keep users data to autheticate them requests, so I defined our app [containers](https://github.com/szz-dvl/rick-and-morty-front/tree/master/src/containers) structure to address the navigation and routing of our app. To this purpose I used the popular npm module [react-router-dom](https://reactrouter.com/). To handle private routes, it is, routes only accessible for authenticated users I've wrote a [component](https://github.com/szz-dvl/rick-and-morty-front/blob/master/src/components/PrivateRoute.tsx) wich again is inspired in the npm module mentioned above (redux-react-session). With the routing pretty much solved I started to code the main layout of the project.

### [Layout](https://github.com/szz-dvl/rick-and-morty-front/blob/master/src/components/Layout.tsx)

This component implements a very simple layout for the app, it will give the user the ability to register if he is not logged into the application or to logout from the system if he is logged in. It will wrap all the containers that will follow.

### [Login](https://github.com/szz-dvl/rick-and-morty-front/tree/master/src/containers/Login)

This container is a simple form that will allow a user to authenticate in the application. It will give an error if the account is not found, the password don't match or no data is provided. There is not much to comment about this container, it just a simple login without further pretense, but this nick name label seems to be hidding something ...

### [Register](https://github.com/szz-dvl/rick-and-morty-front/tree/master/src/containers/Register)

Again, this container does not innovate, it is just meant to be a simple form allowing users to register into the platform. It will throw an error if the choosen name is bussy or the passwords do not match. Still don't trust this nick name label ...

### [List](https://github.com/szz-dvl/rick-and-morty-front/tree/master/src/containers/List)

This is probably the main component of our application, It implements the list of characters available in the system, actually, in the provided API. The list layout is handled by means of a [grid](https://www.w3schools.com/css/css_grid.asp), in the top of the page the user will find a slider that will allow him to chose how many columns he want in each row, from a minimum of one to a maximum of four, depending on the screen the user is using. The peculiarity of our list is that pagination (an optional point of the challenge) is implemented via an infinite scroll. I choose this implementation because nowadays the web is being consumed mainly from mobile devices, so, try to develop more simple, but very functional interfaces seems to be the way to go. This, however, have some complications, specially when the user scrolls fast the time to fetch a new page is crucial. When going down the list behave pretty well, going up is more critical, because we are stacking pages in top of pages held. If one think about how a navigator will render this, can quickly realise that, if we hit the top of the first held page, the previous one will render abruptly. To solve or, at least mitigate, this problem some scroll snaping technics are strategicaly applied, so the list do not jump when a previius page is fetched. List elements are implemented in the component "[CharacterCard](https://github.com/szz-dvl/rick-and-morty-front/tree/master/src/components/CharacterCard)", it is a fully responsive component that will allow the user to navigate to a single character view and to mark or unmark the character as favourite. You may miss some @media queries in the styles of this component however, as the user have the ability to change the number of columns of the grid, it may lead to some situations where we need to restructure our elements that do not depend only on the screen width, and this is the main reason styling is handled with extra classes depending of component props. When returning from the single character view the list will fetch the page where the character was found (previously provided in the route state) and scrolls to the concrete character, provided again in the route state. To avoid strange page fetchs, while this scrolling is happening a reference to the target character is held, and, using [Intersecion Observer](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API), when the character is in place, scroll is listened again. To do so I'm using [this](https://www.npmjs.com/package/react-intersection-observer) npm module.

### [Character](https://github.com/szz-dvl/rick-and-morty-front/tree/master/src/containers/Character)

This is the single character container, it is a fully responsive container that will allow the user to mark or unmark the character as favourite. Due to the lack of information of a character, I'm showing the episodes where the character appeared. If one takes a look at the provided API will quickly realise that there is no request to get a list of episodes (given a list of ids), so we will need to do as many requests as episodes the character appeared in. this may be pretty expensive, specialy for main characters like Rick or Morty, we try to mitigate this by setting the option "force-cache" for [fecth](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch). Since the data we are fetching is not very variable it may save us some extra requests, and, the more episodes we request, the faster the app will behave.

### [Error](https://github.com/szz-dvl/rick-and-morty-front/tree/master/src/components/Error)

This is the component in charge to handle errors, it will mainly deal with 404, however if a requested character is not found in te API, it will be rendered too, so it is implemented in a more generic way.

## [Back End](https://github.com/szz-dvl/rick-and-morty-back)

The back end of the application is a nodeJS HTTP server implemented with the framework [express](https://www.npmjs.com/package/express), as required in the specification of the project. For some unkown reason I assumed that the goal was to write a REST API, I always tend to think in Apollo Server for GraphQL, so the provided server set up is a typical REST API, and being honest, I've more experience writing REST APIs than GraphQL, anyway, I used the official [express app generator](https://expressjs.com/en/starter/generator.html), the following command was used to generate a scaffold of the server:

```bash
express --no-view --git backend
```

With the scaffold ready I adapted the generated code to use Typescript, again, I think this is the way to go for any new project written in JS. Once the server was adapted I set up my [tsconfig.json](https://github.com/szz-dvl/rick-and-morty-back/blob/master/tsconfig.json) and [tslint.json](https://github.com/szz-dvl/rick-and-morty-back/blob/master/tslint.json) and finally added some scripts to [package.json](https://github.com/szz-dvl/rick-and-morty-back/blob/master/package.json) build and run the server. With the environment ready, I started to set up the server, the middlewares used in the main app are the following:

```javascript
app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(express.json())
app.use(cookieParser());
app.use(cors());
```

The set up for the cors middleware is the default one, in a real environment, must be more restrictive, but I think it was enough for this app. Next, the main three routers used in the app:

```javascript
app.use('/user', user);
app.use('/auth', auth);
app.use('/character', character);
```

We will deep into each one later on. Then the general error handler:

```javascript
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {

    /**
     * I'm logging to console, but anything goes here: mails, Logstash, Sentry, etc, etc ...
     */
    console.error(err.stack);
    res.status(StatusCodes.SERVER_ERROR).json({ err: "Internal Server Error" });
});
```

As you can see it is a very simple error handler, as mentioned in the code, in a real production environment we would set up something more powerful. Finally the DB conection.

### The DB

I choosed to use [Mongo DB](https://www.mongodb.com/), to be honest I really like mongo DB and the flexibility it provides, but, this time, it was no effort to choose a non relational DB, more focused on big document than in related records, the data model seems to be perfect for the job. We are using [mongoose](https://mongoosejs.com/) as the DB driver, and, to define our models I'm using [typegoose](https://typegoose.github.io/typegoose/docs/guides/quick-start-guide/), this is a mongoose wrapper which come in handy when using typescript, and leads to very intuitive model definitions. Now the code to connect to a mongo instance is the following:

```javascript
mongoose.connect(
    `mongodb://${process.env.DB_HOST}:${process.env.DB_PORT}/${process.env.DB_NAME}`,
    {
        useNewUrlParser: true,
        useUnifiedTopology: true,
        useCreateIndex: true
    }
);
```

You can find the environment variables in the file we copied before (.env), again, define this variables as you need. One more time, the enviroment variables are imported using dotenv. The only model used in the application is the [user model](https://github.com/szz-dvl/rick-and-morty-back/blob/master/models/user.ts), that will hold all the data of a user, including the list of favourite characters.

### Authentication

To handle user authentication the server will make use of JSON web tokens. To this purpose, I'm using the combo [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) and [passportJS](http://www.passportjs.org/). You can find the strategy used [here](https://github.com/szz-dvl/rick-and-morty-back/blob/master/strategies/jwt.ts) as well as the middleware to protect routes that needs authentication. Now, the routes to authenticate and register a user are found in the file [auth.ts](https://github.com/szz-dvl/rick-and-morty-back/blob/master/routes/auth.ts). The routes defined in this file are the following:

```javascript
POST /auth/login
POST /auth/register
```

In this file you will find the token generation too. The passwords are hashed before being stored in the database, this is done in a mongoose pre save hook, and the module used to achieve so is [bcrypt](https://www.npmjs.com/package/bcrypt).

### [/user](https://github.com/szz-dvl/rick-and-morty-back/blob/master/routes/user.ts)

This router is in charge of getting the favorite characters for a user, store a new favorite and remove a favorite from the list, as all this routes require a user to be uthenticated, the firs statement of the file will be:

```javascript
router.use(needsToken);
```

And the routes defined in this file are:

```javascript
POST /user/favorite
DELETE /user/favorite/:id
GET /user/favorites
```

As you can see, I try to follow the main design pattern for REST APIs.

### [/character](https://github.com/szz-dvl/rick-and-morty-back/blob/master/routes/character.ts)

This is the router in charge of dealing with the external API that will feed us. The comunication is done with the ["official" module](https://www.npmjs.com/package/rickmortyapi) of the Rick and Morty API, that will deal with the external API for us, saving me some work. Again to protect the routes from not autheticated users, the first statement will be:

```javascript
router.use(needsToken);
```

The routes defined in this file are the following:

```javascript
GET /character/list
GET /character/:id
GET /character/episode/:id
```

## Colofon

Well, at this point, I think that the implementation provided for the application is pretty well introduced. I would like to add that no testing is provided, because I'm a little bit short of time these days, but I really care about testing, and if you ask how and where, probably I would say that [Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) are the proper place to launch tests, probably before comitting to master. Regarding to the front end, There is no image loader as such implemented, however, I think the pagination method handle this image loading pretty good. Besides, point out that I know that the images in the front app are not provided in optimized formats (webp or avif), I think it is important too. Finally, I just would like to add that is nice to see how things keep evolving in the React world (specially Redux, gorgeous work they did with typings) and that I enjoyed a lot (and suffered a bit) developing this tiny app, which seems to be the recipe for coding. I hope you like it, and I remain at your disposal.

Thanks, Santi.

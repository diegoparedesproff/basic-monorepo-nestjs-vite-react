- npm install -D turbo

- add to the package.json
      "workspaces": [ "apps/*" ]

- npm install -g @nestjs/cli@latest

- cd apps

- nest new api (project name)

- npm create vite@latest client (project name)
    (choose React and Typescript)

------

At this point you have 2 apps
client (Vite/React) and api ((webpack)/Nest.js)

How to handle the build for both?
using turbo...

create in the root turbo.json

{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "dev": {
      "cache": false
    }
  }
}

In the package.json in api

replace start:dev for just "dev" and add --preserveWatchOutput

In the package.json of the whole monorepo

"dev": "turbo run dev"

When you run that command, turbo is going to
go to all the apps inside the apps folder
and run the dev command, so it will run the
api and the client app

----

Defining a proxy for consuming the api.
Add that in the vite.config.ts

server:{
    proxy:{
        '/api':{
            target:'http://localhost:3000',
            changeOrigin: true
        }
    }
}

and in your apps/api/src/main.ts
add this line app.setGlobalPrefix('api');

Now any time that you make a request to /api
is going to do the fetch to http://localhost:3000

Then you can just do the fetch and see it working

----

For configuring the build of the monorepo:

- add this to your turbo.json in pipeline level
"build":{
    "dependsOn":[
        "^build"
    ],
    "outputs":[
        "dist/**"
    ]
}
- and add a build script in your package.json root
 "build" : "turbo run build"

 After this, when you do npm run build in the root
 you are going to get the build in the dist folder of
 the client app and the api

---------

Serving a static spa with Nest.js

https://docs.nestjs.com/recipes/serve-static#installation

npm install --save @nestjs/serve-static

as we are using turbo:
- npm install --workspace api @nestjs/serve-static

- go to your app.module.ts, add this in to your imports
    ServeStaticModule.forRoot({
      rootPath: join(__dirname, '../..', 'client', 'dist'),
    }),

(this is for production)

and after that just add to your root package.json
"start": "node apps/api/dist/main"
(this is for running the api in production)
(this will also serve the client app from nest)

in this case we use node instead of turbo in the script
because turbo is a dev dependency, in prod we need
to run the project with node

Now we don't need to have the client
running on http://localhost:5173/
and the api running on
http://localhost:3000/api
Now client is on http://localhost:3000
and api in http://localhost:3000/api


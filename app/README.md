# Marketplace Frontend

https://github.com/njain1985/Kuma-Service-Mesh-Observability/blob/main/app/dist/index.html

### Run your tests

- Before deploying the full stack app, let's bake-in New Relic Browser (RUM) Agent into your website. First, enter the app directory in your cloned git repo via terminal or command line.

- Next, cd dist directory and edit index.html

- Notice the HTML comment on line 8, replace it with the new relic browser script <!-- -->

- In order to generate the browser script for your marketplace app, simply

- You should now be able to view the app by going to [http://localhost:8080/](http://localhost:8080/).

- Once done, continue with the marketplace yaml deployment. click [here](https://github.com/njain1985/Kuma-Service-Mesh-Observability#continue)

### Build the app

If you have to build the app for hosted use:

```sh
npm run app:build
```

This will output to a `dist` folder.

### App notes

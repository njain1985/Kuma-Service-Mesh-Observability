# Marketplace Frontend Monitoring

Firstly, browser agent activation is generally extremely simple and doesn't require manual copy + paste script
with some exceptions (like in the case of NODEJS app services)

### Build the Frontend docker image with the New Relic Browser (RUM) Agent baked in

- First, enter the app directory in this cloned repo via terminal or command line.

- Next, "cd dist" and vim or vi  or nano index.html

- Notice the HTML comment on line 8, replace it with the new relic browser script <!-- -->

- In order to generate the browser script for your marketplace app, simply go to one.newrelic.com 

- ![STEP 1](https://user-images.githubusercontent.com/25683435/97065490-632f0d00-15f9-11eb-9e16-17c88668ec3f.png)
- ![STEP 2](https://user-images.githubusercontent.com/25683435/97065486-60ccb300-15f9-11eb-826b-53fe1494076b.png)
- ![STEP 3](https://user-images.githubusercontent.com/25683435/97065482-5ca09580-15f9-11eb-8676-3095fc95e4a7.png)


- You should now be able to view the app by going to [http://localhost:8080/](http://localhost:8080/).

- Once done, continue with the marketplace yaml deployment. click [here](https://github.com/njain1985/Kuma-Service-Mesh-Observability#continue)

### Build the app

If you have to build the app for hosted use:

```sh
npm run app:build
```

This will output to a `dist` folder.

### App notes
